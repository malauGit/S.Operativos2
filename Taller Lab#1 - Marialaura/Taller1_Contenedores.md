# Taller #1: Contenedores

**Universidad Latina de Costa Rica** · Sistemas Operativos II

- **Estudiante:** Marialaura Calderón Herrera
- **Docente:** Carlos Andrés Mendez Rodríguez
- **Sede:** San Pedro, San José
- **Fecha:** noviembre, 2026

---

## 1. ¿Qué es un contenedor?

Un contenedor es una unidad de software que permite ejecutar una aplicación junto con sus dependencias, bibliotecas y configuraciones necesarias. Esto facilita que funcione de la misma manera en diferentes equipos y sistemas operativos, sin necesidad de instalar todos sus componentes directamente en el sistema principal.

Los contenedores utilizan los recursos del sistema operativo del equipo anfitrión, por lo que son más ligeros que las máquinas virtuales tradicionales.

## 2. Diferencias entre LXD y Docker

| Característica | LXD | Docker |
| :--- | :--- | :--- |
| Enfoque principal | Contenedores del sistema | Contenedores de aplicaciones |
| Funcionamiento | Ejecuta un sistema Linux dentro del contenedor | Ejecuta apps y sus dependencias |
| Administración | Permite gestionar contenedores como si fueran máquinas virtuales | Usa imágenes y contenedores para desplegar aplicaciones |
| Uso común | Crear entornos Linux aislados | Desarrollar, probar y desplegar apps |
| Recursos | Comparte el kernel del sistema anfitrión | Comparte el kernel del sistema anfitrión |

## 3. Flujo de la petición

1. El usuario ingresa a la aplicación desde el navegador utilizando `localhost`.
2. HAProxy recibe la petición por el puerto 80 y la distribuye entre los servidores web disponibles.
3. `web1` o `web2` recibe la petición y ejecuta el código PHP correspondiente.
4. PHP establece una conexión con MariaDB para consultar la información de la base de datos.
5. MariaDB devuelve los resultados al servidor web.
6. El servidor procesa la información y envía una respuesta al navegador del usuario.

## 4. Conclusión

La práctica permitió comprender cómo funcionan los contenedores y cómo Docker facilita la ejecución de diferentes servicios de manera independiente. Esta arquitectura permite organizar las aplicaciones, mejorar la distribución de solicitudes y facilitar su administración.

---

## 5. Evidencias

### 5.1 Instalación de Docker y verificación

Post-instalación (`usermod -aG docker`), versión de Docker y Compose, y prueba con `hello-world`.

![Instalación de Docker y hello-world](img/lab1-003-000.png)

### 5.2 Descompresión del proyecto

![Descompresión de grafana.zip](img/lab1-003-001.png)

![Estructura del proyecto con ls -R](img/lab1-003-002.png)

### 5.3 Archivo `docker-compose.yml`

![Contenido de docker-compose.yml](img/lab1-003-003.png)

### 5.4 Construcción y despliegue

```bash
docker compose up --build -d
```

![docker compose up --build -d](img/lab1-003-004.png)

### 5.5 Estado de los servicios y pruebas con `curl`

![docker compose ps, curl y curl -v](img/lab1-004-005.png)

### 5.6 Inspección del contenedor HAProxy

![docker inspect haproxy](img/lab1-004-006.png)

### 5.7 Puertos, procesos y uso de recursos

`docker port haproxy`, `docker top web1`, `docker top mariadb` y `docker stats --no-stream`.

![docker port, top y stats](img/lab1-005-007.png)

### 5.8 Logs de los contenedores

![docker logs de haproxy, web1 y mariadb](img/lab1-005-008.png)

### 5.9 Ejecución de comandos dentro de los contenedores

![docker exec: php -v, ls, getent y consulta a MariaDB](img/lab1-005-009.png)

### 5.10 Prueba de balanceo de carga

![Solicitudes alternando entre las IP de web1 y web2](img/lab1-006-010.png)

### 5.11 Administración de red

![docker network ls e inspect](img/lab1-006-011.png)

### 5.12 Volúmenes y configuración de HAProxy

![docker volume ls y haproxy.cfg](img/lab1-006-012.png)

### 5.13 Ciclo de vida de los contenedores

`docker stop web2`, `docker start web2` y `docker restart haproxy`.

![stop, start y restart](img/lab1-007-013.png)

### 5.14 Cierre del entorno

```bash
docker compose down
```

![docker compose down](img/lab1-007-014.png)
