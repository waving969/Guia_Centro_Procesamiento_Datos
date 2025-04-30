# Práctica 7: Copias de Seguridad

Este documento contiene una guía detallada para implementar diferentes soluciones de copias de seguridad y sincronización entre servidores Linux.

## Índice
1. [Almacenamiento Rsync con SSH](#1-almacenamiento-rsync-con-ssh)
2. [Servidor GIT](#2-servidor-git)
3. [Copias de seguridad con Kopia mediante SSH](#3-copias-de-seguridad-con-kopia-mediante-ssh)
4. [Instalación del servidor Minio para Objetos S3](#4-instalación-del-servidor-minio-para-objetos-s3)
5. [Utilizar el servidor Minio S3 como servidor para Kopia](#5-utilizar-el-servidor-minio-s3-como-servidor-para-kopia)

## 1. Almacenamiento Rsync con SSH

### Creación y sincronización inicial
1. Crear la carpeta `test1` en el servidor centos2:
```bash
mkdir -p test1
```

2. Sincronizar esta carpeta desde centos1 utilizando rsync sobre SSH:
```bash
rsync -avz -e ssh /ruta/carpeta_local/ usuario@centos2:/ruta/test1/
```

### Actualización de archivos
1. Añadir un archivo de prueba:
```bash
echo "contenido de prueba" > apartado1.txt
```

2. Resincronizar con el servidor:
```bash
rsync -avz -e ssh /ruta/carpeta_con_nuevo_archivo/ usuario@centos2:/ruta/test1/
```

## 2. Servidor GIT

### Configuración del repositorio local
1. Crear y configurar un repositorio Git local en centos2:
```bash
mkdir proyecto-git
cd proyecto-git
git init
git config --global user.name "Tu Nombre"
git config --global user.email "tu.email@ejemplo.com"
```

### Configuración del repositorio remoto
1. Crear un repositorio Git en centos1:
```bash
mkdir -p /ruta/repo-remoto.git
cd /ruta/repo-remoto.git
git init --bare
```

2. Conectar el repositorio local con el remoto:
```bash
# En el repositorio local (centos2)
git remote add origin usuario@centos1:/ruta/repo-remoto.git
```

3. Realizar el primer push:
```bash
# Crear un archivo de ejemplo
echo "# Mi proyecto" > README.md
git add README.md
git commit -m "Primer commit"
git push -u origin master
```

### Actualización del repositorio
1. Realizar cambios en el repositorio local:
```bash
echo "Nueva línea de texto" >> README.md
git add README.md
git commit -m "Actualización del README"
```

2. Subir los cambios al repositorio remoto:
```bash
git push origin master
```

## 3. Copias de seguridad con Kopia mediante SSH

### Instalación de Kopia
1. Instalar Kopia CLI en ambos servidores:
```bash
# Descargar la versión más reciente
curl -L -o kopia.tar.gz https://github.com/kopia/kopia/releases/latest/download/kopia-linux-x64.tar.gz
tar -xzf kopia.tar.gz
sudo mv kopia /usr/local/bin/
```

### Configuración del servidor Kopia
1. Crear un directorio de prueba en centos2:
```bash
mkdir -p test3
```

2. Crear el repositorio Kopia en centos2:
```bash
kopia repository create filesystem --path /ruta/repositorio-kopia
```

3. Iniciar el servidor Kopia en centos2:
```bash
kopia server start --address 0.0.0.0:51515 --server-username admin --server-password contraseña-segura
```

### Configuración del cliente Kopia
1. Conectar desde centos1 al servidor Kopia usando el token generado:
```bash
kopia repository connect server --url http://centos2:51515 --server-cert-fingerprint FINGERPRINT --username admin --password contraseña-segura
```

2. Verificar el estado del repositorio:
```bash
kopia repository status
```

### Creación y gestión de backups
1. Crear un directorio para la copia de seguridad en centos1:
```bash
mkdir -p datos_importantes
echo "datos de prueba" > datos_importantes/archivo1.txt
```

2. Realizar la copia de seguridad:
```bash
kopia snapshot create /ruta/datos_importantes
```

3. Verificar las snapshots creadas:
```bash
kopia snapshot list
```

4. Restaurar archivos desde el repositorio:
```bash
kopia restore SNAPSHOT_ID /ruta/destino_restauracion
```

## 4. Instalación del servidor Minio para Objetos S3

### Instalación de Docker y Minio
1. Instalar Docker en centos1:
```bash
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker
```

2. Configurar y ejecutar Minio en un contenedor Docker:
```bash
docker run -d \
  --name minio \
  -p 9000:9000 \
  -p 9001:9001 \
  -e "MINIO_ROOT_USER=minioadmin" \
  -e "MINIO_ROOT_PASSWORD=minioadmin" \
  -v /ruta/datos_minio:/data \
  minio/minio server /data --console-address ":9001"
```

### Configuración del cliente Minio
1. Instalar el cliente Minio en centos2:
```bash
wget https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
sudo mv mc /usr/local/bin/
```

2. Configurar el acceso al servidor Minio:
```bash
mc alias set myminio http://centos1:9000 minioadmin minioadmin
```

### Operaciones básicas con Minio
1. Crear un bucket:
```bash
mc mb myminio/mi-bucket
```

2. Crear subdirectorios:
```bash
mc mb myminio/mi-bucket/subdirectorio
```

3. Copiar archivos al bucket:
```bash
mc cp archivo.txt myminio/mi-bucket/
```

### Acceso desde el cliente local
1. Instalar el cliente Minio en la máquina local siguiendo los mismos pasos que para centos2.

2. Verificar la configuración listando los archivos:
```bash
mc ls myminio/mi-bucket
```

## 5. Utilizar el servidor Minio S3 como servidor para Kopia

1. Configurar Kopia para usar Minio como repositorio:
```bash
kopia repository create s3 \
  --bucket=mi-bucket \
  --endpoint=http://centos1:9000 \
  --access-key=minioadmin \
  --secret-access-key=minioadmin \
  --prefix=kopia-backups
```

2. Realizar copias de seguridad en el repositorio S3:
```bash
kopia snapshot create /ruta/datos
```

3. Listar las copias de seguridad almacenadas en S3:
```bash
kopia snapshot list
```

---

## Autor
Juan Navarro Maldonado
