# Práctica 1: Contenedores Docker e inicialización

En esta práctica aprenderemos los conceptos básicos de contenedores Docker a través de varios ejercicios prácticos que nos permitirán entender cómo funcionan los contenedores, volúmenes y redes en Docker.

## Índice
- [Compartiendo directorios con el host](#compartiendo-directorios-con-el-host)
- [Creación interactiva de un contenedor Docker](#creación-interactiva-de-un-contenedor-docker)
- [Acceso por SSHFS](#acceso-por-sshfs)
- [Crear un volumen utilizando un controlador de volumen](#crear-un-volumen-utilizando-un-controlador-de-volumen)
- [Recursos adicionales](#recursos-adicionales)

## Introducción a Docker

Docker es una plataforma de software que permite crear, probar e implementar aplicaciones rápidamente mediante el uso de contenedores. Los contenedores empaquetan el código, las configuraciones y las dependencias de una aplicación en unidades estandarizadas, lo que permite que la aplicación se ejecute de manera consistente en cualquier entorno.

## Compartiendo directorios con el host

### Objetivo

Aprender a compartir un directorio local del host con un contenedor Docker, permitiendo que los archivos almacenados en nuestra máquina local sean accesibles dentro del contenedor.

### Procedimiento

1. Crear un directorio local con los archivos necesarios:

```bash
mkdir -p ~/nginx-ejemplo
cd ~/nginx-ejemplo
```

2. Crear un archivo `index.html` personalizado:

```bash
echo "<html>
<head>
    <title>Mi página personalizada en Docker</title>
</head>
<body>
    <h1>¡Hola desde mi contenedor NGINX!</h1>
    <p>Esta página está siendo servida desde un contenedor Docker.</p>
</body>
</html>" > index.html
```

3. Ejecutar un contenedor NGINX montando nuestra carpeta local:

```bash
docker run --name nginx2 -v $(pwd):/usr/share/nginx/html -p 8081:80 -p 8444:443 -d nginx
```

4. Acceder a nuestra página web personalizada desde el navegador visitando `http://localhost:8081`

### Explicación

El comando utilizado realiza las siguientes acciones:
- `--name nginx2`: Asigna un nombre al contenedor
- `-v $(pwd):/usr/share/nginx/html`: Monta el directorio actual en la ruta donde NGINX busca archivos HTML
- `-p 8081:80 -p 8444:443`: Mapea los puertos del contenedor (80 y 443) a puertos de nuestro host (8081 y 8444)
- `-d`: Ejecuta el contenedor en modo "detached" (en segundo plano)
- `nginx`: Utiliza la imagen oficial de NGINX

Los cambios realizados en los archivos locales se reflejarán automáticamente en el contenedor sin necesidad de reiniciarlo.

## Creación interactiva de un contenedor Docker

### Objetivo

Aprender a crear y utilizar contenedores en modo interactivo, lo que nos permite ejecutar comandos en tiempo real dentro del contenedor.

### Procedimiento

1. Crear un contenedor Ubuntu interactivo:

```bash
docker run -it ubuntu bash
```

2. Una vez dentro del contenedor, podemos verificar que estamos en él con:

```bash
uname -a
cat /etc/os-release
```

3. Explorar el contenedor ejecutando varios comandos:

```bash
ls -la
apt update
apt install -y curl
curl --version
```

4. Salir del contenedor:

```bash
exit
```

### Explicación

El comando utilizado realiza las siguientes acciones:
- `-i`: Mantiene abierta la entrada estándar (STDIN) aunque no esté conectada
- `-t`: Asigna un pseudo-TTY, permitiendo una experiencia de terminal interactiva
- `ubuntu`: Utiliza la imagen oficial de Ubuntu
- `bash`: Inicia una shell Bash dentro del contenedor

Esto nos permite trabajar dentro del contenedor como si estuviéramos en una máquina virtual completa, pero con la ventaja de la ligereza y eficiencia de los contenedores.

## Acceso por SSHFS

### Objetivo

Aprender a montar sistemas de archivos remotos dentro de un contenedor Docker usando SSHFS.

### Requisitos previos

- Acceso SSH a un servidor remoto
- Credenciales válidas

### Procedimiento

1. Crear un contenedor Alpine con privilegios:

```bash
docker run -it --privileged=true alpine
```

2. Instalar SSHFS dentro del contenedor:

```bash
apk add sshfs
```

3. Crear un directorio para el punto de montaje:

```bash
mkdir mi_remoto
```

4. Montar el directorio remoto:

```bash
sshfs usuario@servidor.ejemplo.com:. mi_remoto
```

5. Navegar y acceder a los archivos remotos:

```bash
cd mi_remoto
ls -la
```

### Explicación

SSHFS (SSH Filesystem) es una herramienta que permite montar un sistema de archivos remoto accesible a través de SSH. Al ejecutar el contenedor con `--privileged=true`, le otorgamos los permisos necesarios para realizar operaciones de montaje. Esto es particularmente útil para acceder a datos remotos sin tener que copiarlos localmente.

## Crear un volumen utilizando un controlador de volumen

### Objetivo

Aprender a crear y usar volúmenes Docker con el controlador SSHFS para acceder a datos remotos.

### Procedimiento

1. Instalar el complemento SSHFS para Docker:

```bash
docker plugin install vieux/sshfs
```

2. Verificar la instalación del complemento:

```bash
docker plugin ls
```

3. Crear un volumen utilizando el controlador SSHFS:

```bash
docker volume create \
  --driver vieux/sshfs \
  -o sshcmd=usuario@servidor.ejemplo.com \
  -o password=contraseña \
  ssh_volume
```

Nota: Por motivos de seguridad, es preferible usar autenticación por clave SSH en lugar de contraseñas.

4. Crear un contenedor que utilice el volumen:

```bash
docker run -it --name ubuntu_sshfs \
  -v ssh_volume:/mnt/remote_data \
  ubuntu /bin/bash
```

5. Dentro del contenedor, explorar los datos montados:

```bash
cd /mnt/remote_data
ls -la
```

### Explicación

El controlador de volumen `vieux/sshfs` permite crear volúmenes Docker que se conectan a servidores remotos mediante SSH. Esto ofrece varias ventajas:

- Persistencia de datos más allá del ciclo de vida del contenedor
- Posibilidad de compartir el mismo volumen entre varios contenedores
- Acceso a datos remotos sin necesidad de sincronizaciones manuales

## Recursos adicionales

- [Documentación oficial de Docker](https://docs.docker.com/)
- [Referencia de comandos Docker](https://docs.docker.com/engine/reference/commandline/cli/)
- [Guía de volúmenes en Docker](https://docs.docker.com/storage/volumes/)
- [Docker Hub](https://hub.docker.com/) - Repositorio de imágenes Docker

## Ejercicios propuestos

1. Crear un contenedor de MySQL y montar un volumen local para persistir los datos
2. Crear una red Docker y conectar múltiples contenedores entre sí
3. Implementar un servidor web Nginx con PHP-FPM usando dos contenedores conectados
