# Práctica 3: Docker Swarm

Esta práctica se centra en Docker Swarm, la solución nativa de Docker para la orquestación de contenedores. Se explorará la creación de un clúster, el despliegue de servicios y la configuración de alta disponibilidad.

## Contenidos
- [Creación del clúster Docker Swarm](#creación-del-clúster-docker-swarm)
- [Despliegue de un servicio web en el clúster](#despliegue-de-un-servicio-web-en-el-clúster)
- [Gestión de la alta disponibilidad](#gestión-de-la-alta-disponibilidad)
- [Pruebas de tolerancia a fallos](#pruebas-de-tolerancia-a-fallos)

## Creación del clúster Docker Swarm

En este apartado se describe el proceso para crear un clúster de Docker Swarm con tres nodos:

1. Crear la primera máquina virtual (node1) que actuará como manager:

```bash
# En node1 (manager)
docker swarm init --advertise-addr <IP_MANAGER>
```

2. El comando anterior generará un token y un comando para unir nodos trabajadores al clúster. Este token debe usarse en los otros nodos:

```bash
# En node2 y node3 (workers)
docker swarm join --token <TOKEN> <IP_MANAGER>:<PORT>
```

3. Verificar que los nodos se han unido correctamente al clúster:

```bash
# En node1 (manager)
docker node ls
```

Este comando mostrará los tres nodos del clúster, indicando cuál es el manager y cuáles son workers.

## Despliegue de un servicio web en el clúster

Una vez configurado el clúster, se despliega un servicio web (nginx) con tres réplicas:

```bash
docker service create --name web --replicas 3 \
  --mount type=bind,src=/etc/hostname,dst=/usr/share/nginx/html/index.html,readonly \
  --publish published=8080,target=80 nginx
```

Este comando:
- Crea un servicio llamado "web"
- Configura 3 réplicas para alta disponibilidad
- Monta el archivo `/etc/hostname` del host como página principal del servidor web
- Publica el puerto 8080 del host y lo redirige al puerto 80 del contenedor

Para verificar el estado del servicio:

```bash
docker service ls
```

Para ver los contenedores individuales que componen el servicio:

```bash
docker ps
```

## Gestión de la alta disponibilidad

### Verificación del funcionamiento de las réplicas

Para comprobar que las tres réplicas están funcionando correctamente:

```bash
curl http://localhost:8080
```

Este comando debe devolver el contenido del archivo `/etc/hostname` de la máquina host.

### Cambio de escala del servicio

Se puede modificar dinámicamente el número de réplicas del servicio:

```bash
docker service scale web=2
```

Este comando reduce el número de réplicas a 2. Para verificar el cambio:

```bash
docker service ls
```

## Pruebas de tolerancia a fallos

Para comprobar la capacidad de recuperación ante fallos del clúster:

1. Detener el servicio Docker en uno de los nodos activos (por ejemplo, node2):

```bash
# En node2
systemctl stop docker
```

2. Verificar que el servicio sigue disponible a pesar de la caída de un nodo:

```bash
# En node1 o node3
curl http://localhost:8080
```

3. Comprobar cómo Docker Swarm ha redistribuido automáticamente las réplicas para mantener el número deseado:

```bash
docker service ls
```

Estas pruebas demuestran la capacidad de Docker Swarm para mantener la alta disponibilidad y la tolerancia a fallos dentro del clúster, asegurando que los servicios permanezcan accesibles incluso cuando un nodo deja de funcionar.

## Conclusiones

Docker Swarm proporciona una solución robusta para la orquestación de contenedores con características de alta disponibilidad y tolerancia a fallos. Su integración nativa con Docker lo hace especialmente adecuado para entornos donde ya se utiliza esta tecnología de contenedores.

Características destacadas:
- Configuración sencilla del clúster
- Balanceo de carga automático
- Escalado dinámico de servicios
- Recuperación automática ante fallos de nodos
