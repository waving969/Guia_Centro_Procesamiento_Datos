# Práctica 4: Creación de máquinas virtuales con Vagrant y despliegue de almacenamiento con GlusterFS

En esta práctica implementaremos un sistema de almacenamiento distribuido utilizando GlusterFS con 3 nodos virtuales:
- 2 nodos como servidores de almacenamiento (centos1 y centos2) 
- 1 nodo como cliente (centos3)

Crearemos un volumen redundante utilizando los discos de los dos servidores, lo que nos permitirá mantener la disponibilidad del sistema incluso si uno de los nodos falla.

## Índice
- [Introducción a GlusterFS](#introducción-a-glusterfs)
- [Creación de máquinas virtuales con Vagrant](#creación-de-máquinas-virtuales-con-vagrant)
- [Instalación y configuración de GlusterFS](#instalación-y-configuración-de-glusterfs)
- [Creación de bricks (componentes de almacenamiento)](#creación-de-bricks)
- [Configuración del sistema de archivos distribuido](#configuración-del-sistema-de-archivos-distribuido)
- [Instalación del cliente GlusterFS](#instalación-del-cliente-glusterfs)
- [Comprobación de la redundancia](#comprobación-de-la-redundancia)

## Introducción a GlusterFS

GlusterFS es un sistema de almacenamiento distribuido que permite crear volúmenes redundantes y escalables con varios nodos. Sus principales características son:

- **Escalabilidad**: Permite agregar nodos de forma dinámica
- **Redundancia**: Ofrece replicación para garantizar alta disponibilidad
- **Rendimiento**: Permite distribuir la carga entre varios servidores
- **Acceso transparente**: Los clientes acceden a los datos como si fuera un sistema de archivos local

## Creación de máquinas virtuales con Vagrant

Para esta práctica utilizaremos 3 máquinas virtuales con CentOS. Vagrant nos permite automatizar la creación de estas máquinas.

1. Crear una carpeta para el proyecto y descargar el archivo Vagrantfile desde el archivo comprimido proporcionado:

```bash
mkdir cpd4
cd cpd4
# Copiar el archivo Vagrantfile del archivo comprimido vagrantfile_pr4.zip
```

2. Iniciar las máquinas virtuales:

```bash
vagrant up
```

3. Acceder a las máquinas (repite el comando para cada máquina):

```bash
vagrant ssh centos1
vagrant ssh centos2
vagrant ssh centos3
```

## Instalación y configuración de GlusterFS

### Instalación en los servidores (centos1 y centos2)

1. Cambiar a usuario root:

```bash
sudo su -
```

2. Activar el repositorio CRB (Code Ready Builder) y actualizar el sistema:

```bash
dnf config-manager --set-enabled crb
dnf update
```

3. Buscar la versión disponible de GlusterFS:

```bash
dnf search centos-release-gluster
```

4. Instalar GlusterFS:

```bash
dnf -y install centos-release-gluster11
dnf -y update
dnf -y install glusterfs glusterfs-cli glusterfs-libs glusterfs-server
```

### Iniciando el servicio

1. Habilitar e iniciar el servicio GlusterFS:

```bash
systemctl enable glusterd.service
systemctl start glusterd.service
```

2. Configurar el firewall para permitir el tráfico de GlusterFS:

```bash
firewall-cmd --add-service=glusterfs --permanent
firewall-cmd --reload
```

3. Configurar el cluster conectando los nodos:

En centos1:
```bash
gluster peer probe centos2
gluster peer status
```

En centos2:
```bash
gluster peer probe centos1
gluster peer status
```

## Creación de bricks

Un "brick" en GlusterFS es una unidad básica de almacenamiento. Vamos a crear uno en cada servidor.

### En ambos servidores (centos1 y centos2):

1. Crear partición en el disco adicional:
```bash
fdisk /dev/sdb
# Presionar 'n' para nueva partición
# Presionar 'p' para primaria
# Número de partición: 1
# Aceptar valores por defecto para primer y último sector
# Presionar 't' para cambiar tipo
# Seleccionar tipo '8e' (Linux LVM)
# Presionar 'w' para guardar cambios
```

2. Actualizar la tabla de particiones:
```bash
partprobe
```

3. Crear volumen físico, grupo de volúmenes y volumen lógico:
```bash
pvcreate /dev/sdb1
vgcreate vg01 /dev/sdb1
lvcreate -l 100%FREE -n lv01 vg01
```

4. Crear sistema de archivos XFS:
```bash
mkfs.xfs /dev/mapper/vg01-lv01
```

5. Crear el directorio para el brick:
```bash
mkdir -p /gluster/bricks/brick1
```

6. Configurar el montaje automático editando /etc/fstab:
```bash
echo "/dev/mapper/vg01-lv01 /gluster/bricks/brick1 xfs defaults 0 0" >> /etc/fstab
mount -a
```

## Configuración del sistema de archivos distribuido

1. Verificar que los nodos estén conectados:
```bash
gluster peer status
gluster pool list
```

2. Crear directorios para el volumen GlusterFS en ambos servidores:
```bash
mkdir /gluster/bricks/brick1/vol1
```

3. Crear el volumen replicado:
```bash
gluster volume create glustervol1 replica 2 transport tcp centos1:/gluster/bricks/brick1/vol1 centos2:/gluster/bricks/brick1/vol1
```

4. Iniciar el volumen:
```bash
gluster volume start glustervol1
```

5. Verificar la información del volumen:
```bash
gluster volume info glustervol1
```

## Instalación del cliente GlusterFS

En el nodo centos3:

1. Instalar los paquetes necesarios:
```bash
sudo dnf config-manager --set-enabled crb
sudo dnf -y update
sudo dnf -y install centos-release-gluster11
sudo dnf -y update
sudo dnf -y install glusterfs-cli glusterfs-fuse
```

2. Crear el punto de montaje:
```bash
sudo mkdir /gdatos1
```

3. Montar el volumen GlusterFS:
```bash
sudo mount -t glusterfs centos1:/glustervol1 /gdatos1
```

## Comprobación de la redundancia

1. Crear archivos de prueba en el volumen montado:
```bash
sudo touch /gdatos1/test_file.txt
echo "Test content" | sudo tee /gdatos1/test_file.txt
```

2. Apagar uno de los nodos (centos1) para simular un fallo:
```bash
# En centos1
sudo shutdown -h now
```

3. Verificar que el volumen sigue disponible en el cliente (centos3):
```bash
cat /gdatos1/test_file.txt
```

4. Crear nuevos archivos mientras centos1 está apagado:
```bash
echo "New content during failure" | sudo tee /gdatos1/during_failure.txt
```

5. Levantar centos1 nuevamente y verificar la sincronización:
```bash
# Una vez que centos1 está disponible de nuevo
ls -la /gluster/bricks/brick1/vol1
cat /gluster/bricks/brick1/vol1/during_failure.txt
```

## Recursos adicionales

- [Archivo Vagrantfile](recursos/Vagrantfile)
- [Documentación oficial de GlusterFS](https://docs.gluster.org/en/latest/)
