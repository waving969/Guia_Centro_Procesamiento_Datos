# Práctica 2: Contenedores Docker (II)

En esta práctica se exploran conceptos avanzados de Docker, centrándonos en la creación de imágenes personalizadas, su publicación en Docker Hub y la configuración de servicios complejos como WordPress mediante Docker Compose.

## Contenidos
- [Creación de imágenes personalizadas con Dockerfile](#creación-de-una-imagen-personalizada-de-docker-con-dockerfile)
- [Publicación de imágenes en Docker Hub](#subir-imagen-a-hubdockercom)
- [Despliegue de WordPress con Docker Compose](#iniciar-servidor-wordpress-y-editar-la-página-principal)
- [Pruebas de rendimiento con Sysbench](#pruebas-de-rendimiento-con-sysbench)

## Creación de una imagen personalizada de Docker con Dockerfile

En este apartado creamos una imagen personalizada basada en Debian que incluye Apache2 y un archivo `index.html` personalizado.

### Dockerfile utilizado

```dockerfile
FROM debian
MAINTAINER Usuario CPD "juannavarrom@correo.ugr.es"
RUN apt-get update && apt-get install -y apache2 && apt-get clean && rm -rf /var/lib/apt/lists/*
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2
EXPOSE 80
ADD ["index.html","/var/www/html/"]
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

### Archivo index.html personalizado

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Prueba CPD</title>
</head>
<body>
    <h1>Prueba inicial CPD</h1>
</body>
</html>
```

### Comandos utilizados

Para construir la imagen:
```bash
sudo docker build -t ejercicio1 .
```

Para ejecutar un contenedor basado en esta imagen:
```bash
docker run --name x1 -p8080:80 -d ejercicio1
```

Para verificar que el contenedor está funcionando:
```bash
docker ps
```

Finalmente, podemos acceder a la página web en `localhost:8080`.

## Subir imagen a hub.docker.com

Proceso para publicar nuestra imagen personalizada en Docker Hub:

1. Iniciar sesión en Docker Hub:
   ```bash
   docker login
   ```

2. Reconstruir la imagen con el formato apropiado para Docker Hub:
   ```bash
   sudo docker build -t waving969/cpd:1.0 .
   ```

3. Publicar la imagen:
   ```bash
   docker push waving969/cpd:1.0
   ```

## Iniciar servidor WordPress y editar la página principal

Configuración de un entorno WordPress utilizando Docker Compose.

### docker-compose.yml utilizado

```yaml
version: '3.3'
services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: cpdwordpress
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: cpdwordpress
volumes:
  db_data: {}
```

Para iniciar el entorno WordPress:
```bash
docker-compose up -d
```

Luego se puede acceder al panel de administración en `localhost:8000` para completar la configuración.

## Pruebas de rendimiento con Sysbench

Creación de una imagen Docker para realizar pruebas de rendimiento con Sysbench.

### Dockerfile para la imagen de pruebas

```dockerfile
# Usar la imagen base de Ubuntu
FROM ubuntu:20.04
# Establecer el directorio de trabajo
WORKDIR /app
# Actualizar e instalar sysbench
RUN apt-get update && \
    apt-get install -y sysbench && \
    apt-get clean
# Comando por defecto para ejecutar sysbench
CMD ["sysbench", "--help"]
```

### Creación y publicación de la imagen

```bash
docker build -t waving969/sysbench-ubuntu:1.0 .
docker push waving969/sysbench-ubuntu:1.0
```

### Ejecución de pruebas de rendimiento

Para ejecutar Sysbench sin restricciones:
```bash
docker run waving969/sysbench-ubuntu:1.0 sysbench cpu run
```

Para limitar las CPUs disponibles (ejemplo con 5 CPUs):
```bash
docker run --cpus="5" waving969/sysbench-ubuntu:1.0 sysbench cpu run
```

Las pruebas muestran cómo los recursos limitados afectan al rendimiento, observando reducciones en los eventos por segundo y aumentos en la latencia cuando se restringen las CPUs disponibles para el contenedor.
