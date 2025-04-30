# Práctica 5: Acceso Remoto

Esta práctica se centra en la implementación y configuración de herramientas para el acceso remoto entre máquinas, utilizando tecnologías como SSH, VNC y ZeroTier para establecer conexiones seguras y eficientes.

## Contenidos
- [Configuración de acceso remoto mediante SSH y VNC](#configuración-de-acceso-remoto-mediante-ssh-y-vnc)
- [Acceso SSH al escritorio remoto](#acceso-ssh-al-escritorio-remoto)
- [Acceso directo mediante VNC](#acceso-directo-mediante-vnc)
- [Acceso SSH a la máquina de un compañero](#acceso-ssh-a-la-máquina-de-un-compañero)
- [Acceso VNC a la máquina virtual del compañero](#acceso-vnc-a-la-máquina-virtual-del-compañero)

## Configuración de acceso remoto mediante SSH y VNC

Para establecer conexiones remotas entre máquinas, necesitamos configurar:

1. **ZeroTier**: Red virtual que permite conectar dispositivos independientemente de su ubicación física
2. **SSH**: Para acceso seguro a la terminal
3. **VNC**: Para acceso al entorno gráfico del escritorio remoto

### Requisitos previos

- Tener instalado ZeroTier en ambas máquinas (local y remota)
- Configurar el servidor VNC en la máquina remota
- Conocer las direcciones IP asignadas por ZeroTier

## Acceso SSH al escritorio remoto

### Iniciar el servidor VNC en la máquina remota

Primero debemos iniciar el servidor VNC en la máquina virtual para permitir conexiones remotas al escritorio:

```bash
vncserver :1
```

Este comando inicia un servidor VNC en el puerto 5901 (correspondiente al display :1).

### Establecer conexión SSH con redirección de puertos

Desde la máquina local, utilizamos SSH para conectarnos a la máquina remota y redirigir el puerto VNC:

```bash
ssh -L 5901:localhost:5901 vagrant@172.23.165.43
```

Este comando:
- Establece una conexión SSH con la máquina virtual
- Redirige el puerto 5901 de la máquina remota al puerto 5901 de la máquina local
- Permite acceder al servidor VNC remoto como si estuviera ejecutándose localmente

> **Nota**: La dirección IP (172.23.165.43) debe reemplazarse por la IP de tu máquina virtual en la red ZeroTier.

## Acceso directo mediante VNC

Una vez establecida la redirección de puertos mediante SSH, podemos conectarnos al escritorio remoto utilizando un cliente VNC:

### Usando un cliente VNC en macOS

En macOS, podemos utilizar el cliente VNC integrado. Para ello:

1. Abrir Finder
2. En la barra de direcciones, introducir:

```
vnc://172.23.165.43:5901
```

O, si estamos utilizando la redirección de puertos mediante SSH:

```
vnc://localhost:5901
```

### Verificación de la conexión

Para asegurarnos de que la conexión entre las máquinas está funcionando correctamente, podemos realizar un ping desde la máquina local a la dirección IP de la máquina remota:

```bash
ping 172.23.165.43
```

Una respuesta exitosa confirma que ambas máquinas están correctamente conectadas a través de ZeroTier.

## Acceso SSH a la máquina de un compañero

Para acceder a la máquina de un compañero mediante SSH, es necesario que ambas máquinas estén conectadas a la misma red ZeroTier.

### Pasos para conectarse a la máquina del compañero

1. Verificar que ambas máquinas estén conectadas a la misma red ZeroTier
2. Identificar la dirección IP de la máquina del compañero en la red ZeroTier
3. Establecer la conexión SSH:

```bash
ssh -L 5901:localhost:5901 vagrant@10.147.18.2
```

Este comando establece una conexión SSH con la máquina del compañero y redirige el puerto VNC para acceder posteriormente al escritorio remoto.

## Acceso VNC a la máquina virtual del compañero

Una vez establecida la conexión SSH con la máquina del compañero, podemos acceder a su escritorio remoto utilizando un cliente VNC:

### Pasos para conectarse al escritorio remoto del compañero

1. Asegurarse de que ambas máquinas estén conectadas a la red ZeroTier
2. Verificar que el servidor VNC esté en ejecución en la máquina del compañero
3. Utilizar un cliente VNC para conectarse:

En macOS:
```
vnc://10.147.18.2:5901
```

> **Nota**: Las direcciones IP pueden cambiar si se crean nuevas redes virtuales. En caso de problemas de conexión, verificar las IPs asignadas en la interfaz de ZeroTier.

### Verificación de la conexión

Para confirmar que la máquina del compañero está accesible, podemos realizar un ping desde nuestra máquina local:

```bash
ping 10.147.18.2
```

Una respuesta exitosa confirma que ambas máquinas están correctamente conectadas a través de ZeroTier y que la configuración de acceso remoto funciona adecuadamente.

## Conclusiones

El uso combinado de ZeroTier, SSH y VNC proporciona una solución completa para el acceso remoto tanto a la línea de comandos como al entorno gráfico de máquinas remotas. Esta configuración es especialmente útil para administración remota, trabajo colaborativo y acceso a recursos compartidos.

Puntos clave:
- ZeroTier facilita la conexión entre dispositivos independientemente de su ubicación física
- SSH proporciona acceso seguro y cifrado a la terminal remota
- La redirección de puertos mediante SSH permite acceder a servicios remotos de forma segura
- VNC permite acceder y controlar el entorno gráfico de una máquina remota
