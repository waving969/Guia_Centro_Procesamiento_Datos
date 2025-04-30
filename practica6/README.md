# Práctica 6: Almacenamiento Sincronizado y Compartido

Esta práctica se centra en la implementación de un sistema de almacenamiento en la nube tipo Dropbox/Google Drive utilizando NextCloud, que permite sincronizar y compartir archivos entre diferentes dispositivos y usuarios.

## Contenidos
- [Despliegue del servidor NextCloud](#despliegue-del-servidor-nextcloud)
- [Configuración del cliente NextCloud en máquina local](#configuración-del-cliente-nextcloud-en-máquina-local)
- [Configuración del cliente NextCloud en máquina Vagrant](#configuración-del-cliente-nextcloud-en-máquina-vagrant)
- [Configuración avanzada con ZeroTier](#configuración-avanzada-con-zerotier)

## Despliegue del servidor NextCloud

Para desplegar el servidor NextCloud, utilizamos Docker Compose con una configuración que incluye todos los servicios necesarios:

### Archivo docker-compose.yml

```yaml
version: '3'
services:
  db:
    image: mariadb:10.11
    restart: always
    command: --transaction-isolation=READ-COMMITTED --log-bin=binlog --binlog-format=ROW
    volumes:
      - db:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD="practica6cpd"
      - MYSQL_PASSWORD="practica6cpd"
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
  redis:
    image: redis:alpine
    restart: always
  app:
    image: nextcloud
    restart: always
    ports:
      - 8080:80
    depends_on:
      - redis
      - db
    volumes:
      - nextcloud:/var/www/html
    environment:
      - MYSQL_PASSWORD="practica6cpd"
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_HOST=db
volumes:
  nextcloud:
  db:
```

### Pasos para el despliegue

1. Guardar el archivo `docker-compose.yml` en un directorio local
2. Desde ese directorio, ejecutar el siguiente comando para iniciar los servicios:

```bash
docker-compose up -d
```

3. Acceder a la interfaz web de NextCloud a través de un navegador:

```
http://localhost:8080
```

4. Configurar NextCloud en la primera ejecución:
   - Crear una cuenta de administrador
   - Configurar la conexión a la base de datos:
     - Host de la base de datos: `db`
     - Nombre de la base de datos: `nextcloud`
     - Usuario: `nextcloud`
     - Contraseña: `practica6cpd`

Tras completar estos pasos, el servidor NextCloud estará listo para su uso.

## Configuración del cliente NextCloud en máquina local

### Instalación del cliente

1. Descargar el cliente NextCloud desde la [página oficial](https://nextcloud.com/clients/)
2. Instalar el cliente siguiendo las instrucciones para tu sistema operativo

### Configuración del cliente

1. Ejecutar el cliente NextCloud
2. En la pantalla de configuración, introducir la dirección del servidor:

```
http://localhost:8080
```

3. Introducir las credenciales de usuario creadas durante la configuración del servidor
4. Seleccionar los directorios que se desean sincronizar

Tras la configuración, se creará una carpeta en el sistema local que estará sincronizada con el servidor NextCloud. Cualquier archivo que se añada, modifique o elimine en esta carpeta se sincronizará automáticamente con el servidor.

## Configuración del cliente NextCloud en máquina Vagrant

Para permitir que una máquina Vagrant acceda al servidor NextCloud, primero debemos modificar la configuración del servidor para que acepte conexiones desde la IP de la máquina Vagrant.

### Modificación del archivo config.php en el servidor

1. Acceder al contenedor de NextCloud:

```bash
docker exec -it <id_contenedor_nextcloud> /bin/bash
```

2. Editar el archivo de configuración para añadir la IP de la máquina Vagrant:

```bash
nano config/config.php
```

3. Añadir la IP de la máquina Vagrant al array `trusted_domains`:

```php
'trusted_domains' => 
  array (
    0 => 'localhost',
    1 => '192.168.56.1',  // IP de la máquina Vagrant
  ),
```

4. Guardar los cambios y reiniciar el contenedor:

```bash
docker restart <id_contenedor_nextcloud>
```

### Instalación y configuración del cliente en la máquina Vagrant

1. Descargar e instalar el cliente NextCloud en la máquina Vagrant
2. Configurar el cliente utilizando la IP del servidor NextCloud
3. Introducir las credenciales de usuario
4. Seleccionar los directorios a sincronizar

Una vez configurado, la máquina Vagrant tendrá acceso a los mismos archivos que la máquina local, permitiendo la colaboración y sincronización entre ambos sistemas.

## Configuración avanzada con ZeroTier

Para permitir el acceso al servidor NextCloud desde cualquier dispositivo conectado a una red ZeroTier, debemos modificar nuevamente la configuración del servidor.

### Añadir la IP de ZeroTier al archivo config.php

1. Acceder al contenedor de NextCloud:

```bash
docker exec -it <id_contenedor_nextcloud> /bin/bash
```

2. Editar el archivo de configuración:

```bash
nano config/config.php
```

3. Añadir la IP asignada por ZeroTier al array `trusted_domains`:

```php
'trusted_domains' => 
  array (
    0 => 'localhost',
    1 => '192.168.56.1',  // IP de la máquina Vagrant
    2 => '172.23.165.43', // IP asignada por ZeroTier
  ),
```

4. Guardar los cambios y reiniciar el contenedor:

```bash
docker restart <id_contenedor_nextcloud>
```

### Acceso desde dispositivos en la red ZeroTier

Una vez configurado, cualquier dispositivo que forme parte de la red ZeroTier podrá acceder al servidor NextCloud utilizando la IP asignada por ZeroTier:

```
http://172.23.165.43:8080
```

Esto permite ampliar el acceso al sistema de almacenamiento a dispositivos remotos que estén conectados a la misma red ZeroTier, incluso si no están en la misma red local física.

## Conclusiones

La implementación de NextCloud como sistema de almacenamiento sincronizado y compartido ofrece una solución versátil y segura para la gestión de archivos en diversos entornos:

- **Control total**: Al alojar el servidor en infraestructura propia, se mantiene el control completo sobre los datos
- **Flexibilidad de acceso**: La combinación con ZeroTier permite el acceso desde cualquier ubicación sin necesidad de exponer el servidor a Internet
- **Sincronización automática**: Cualquier cambio realizado en un dispositivo se propaga automáticamente a todos los demás
- **Colaboración**: Múltiples usuarios pueden acceder y modificar los mismos archivos

Este enfoque proporciona una alternativa de código abierto a servicios comerciales como Dropbox o Google Drive, con ventajas adicionales en términos de privacidad y control.
