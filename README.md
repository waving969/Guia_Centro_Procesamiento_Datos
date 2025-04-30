# Guía de Prácticas de Centro de Procesamiento de Datos

Este repositorio contiene documentación detallada sobre diversas prácticas realizadas en el ámbito de Centro de Procesamiento de Datos. Cada práctica incluye instrucciones paso a paso, comandos utilizados y capturas de pantalla para facilitar la reproducción de los ejercicios.

## Índice de Prácticas

1. [Contenedores Docker e inicialización](practicas/practica1/README.md)
2. [Contenedores Docker 2: Creación de imágenes personalizadas](practicas/practica2/README.md)
3. [Docker Swarm: Ejecución de un servicio web](practicas/practica3/README.md)
4. [Creación de máquinas virtuales con Vagrant y despliegue de almacenamiento con GlusterFS](practica4/README.md)
5. [Acceso remoto mediante SSH a un escritorio de una máquina virtual](practicas/practica5/README.md)
6. [Almacenamiento sincronizado y compartido: Despliegue de un servidor NextCloud y ZeroTier](practicas/practica6/README.md)
7. [Copias de seguridad con Kopia](practicas/practica7/README.md)
8. [Seguridad: Fail2Ban, Google Authenticator, servidor Nginx con protección y red TOR](practicas/practica8/README.md)
9. [Virtualización con LXD y gestión de alertas con Telegram](practicas/practica9/README.md)
10. [Monitorización de recursos con Grafana](practicas/practica10/README.md)

## Estructura del Repositorio

```
├── README.md
├── LICENSE
├── .gitignore
├── practicas/
│   ├── practica1/
│   │   ├── README.md
│   │   ├── imagenes/
│   │   └── recursos/
│   ├── practica2/
│   │   ├── README.md
│   │   ├── imagenes/
│   │   └── recursos/
│   ├── ... (estructura similar para cada práctica)
│   └── practica10/
│       ├── README.md
│       ├── imagenes/
│       └── recursos/
```

## Tecnologías cubiertas

- **Virtualización**: Vagrant, VirtualBox, LXD
- **Contenedores**: Docker, Docker Swarm
- **Almacenamiento distribuido**: GlusterFS, NextCloud
- **Seguridad**: SSH, Fail2Ban, Google Authenticator, red TOR
- **Copias de seguridad**: Kopia
- **Redes**: ZeroTier
- **Monitorización**: Grafana
- **Alertas**: Telegram

## Requisitos previos

Para seguir estas guías necesitarás:

- Conocimientos básicos de Linux
- Un sistema operativo Linux (recomendado) o Windows con WSL2
- Conexión a Internet para descargar los paquetes necesarios
- Hardware suficiente para ejecutar máquinas virtuales:
  - CPU: 4 núcleos o más
  - RAM: 8 GB mínimo recomendado
  - Almacenamiento: 50 GB de espacio libre

## Cómo usar esta guía

Cada práctica está contenida en su propia carpeta con:

1. Un archivo README.md con instrucciones paso a paso
2. Capturas de pantalla en la carpeta "imagenes"
3. Archivos de configuración, scripts y otros recursos en la carpeta "recursos"

Se recomienda seguir las prácticas en orden, ya que algunas construyen sobre conocimientos adquiridos en prácticas anteriores.

## Contribuciones

Si deseas contribuir a este repositorio, puedes hacer un fork y enviar un pull request con tus mejoras o correcciones. Agradezco cualquier contribución que ayude a mejorar la calidad y claridad de estas guías.

## Autor

Juan Navarro Maldonado

## Licencia

Este proyecto está bajo la Licencia [MIT](LICENSE)