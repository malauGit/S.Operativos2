# Taller #2: WordPress con balanceo de carga, Redis y tolerancia a fallos

**Universidad Latina de Costa Rica** · Sistemas Operativos II

- **Estudiante:** Marialaura Calderón Herrera
- **Docente:** Carlos Andrés Mendez Rodríguez
- **Sede:** San Pedro, San José
- **Fecha:** noviembre, 2026

---

## 1. Preguntas de análisis

**a. ¿Por qué es relevante que `www-data` tenga UID 1003 dentro de ambos contenedores?**
Porque permite que ambos contenedores tengan acceso a los archivos de WordPress con los permisos correctos.

**b. ¿Qué ventaja ofrece ajustar el UID/GID en el `entrypoint` en lugar de incorporarlo como `ARG` de construcción?**
Permite cambiar los permisos al iniciar el contenedor sin tener que reconstruir la imagen.

**c. ¿Qué ocurriría si `web1` utilizara UID 33 y `web2` UID 1003?**
`web1` podría tener problemas de permisos, mientras que `web2` podría acceder correctamente a los archivos.

**d. ¿Por qué Redis Object Cache no es, por sí solo, el responsable de mantener el inicio de sesión nativo de WordPress?**
Porque Redis almacena datos en caché, mientras que WordPress utiliza cookies y claves de autenticación para mantener el inicio de sesión.

**e. ¿Qué datos deben ser idénticos o compartidos para que cualquier réplica valide la cookie de autenticación?**
Las claves y sales de autenticación de WordPress, además de la misma base de datos de usuarios.

**f. ¿Qué diferencia existe entre balanceo de carga y alta disponibilidad?**
El balanceo distribuye las solicitudes entre servidores; la alta disponibilidad busca mantener el servicio funcionando ante fallos.

**g. ¿Por qué esta solución no es completamente tolerante a fallos?**
Porque todavía existen componentes únicos, como MariaDB, Redis y HAProxy, cuya falla puede afectar todo el sistema.

**h. ¿Qué ocurre si falla MariaDB?**
WordPress pierde el acceso a sus datos y las dos réplicas pueden dejar de funcionar correctamente.

**i. ¿Qué ocurre si falla Redis? Distinga entre caché de objetos y sesiones PHP de plugins.**
Se pierde la caché de objetos y puede disminuir el rendimiento. Los plugins que guardan sesiones PHP en Redis también pueden presentar errores.

**j. ¿Qué limitación presenta `/home/orh` si las réplicas se trasladan a hosts distintos?**
Es un directorio local y no se comparte automáticamente entre diferentes equipos.

**k. ¿Qué función cumplen `fall 1` y `rise 2` en HAProxy?**
`fall 1` marca un servidor como caído después de un fallo. `rise 2` lo considera disponible después de dos comprobaciones exitosas.

**l. ¿Por qué no se expone MariaDB ni Redis mediante `ports`?**
Porque solo necesitan comunicarse dentro de la red interna de Docker, sin quedar expuestos al exterior.

**m. ¿Qué riesgo representa que un usuario pertenezca al grupo `docker`?**
Puede administrar contenedores y obtener privilegios equivalentes a los de administrador (root).

**n. ¿Por qué una solicitud que estaba en ejecución podría fallar aunque exista otra réplica?**
Porque HAProxy no siempre puede trasladar una solicitud que ya estaba siendo procesada por el servidor que falló.

**o. ¿Qué componentes deberían duplicarse para eliminar los puntos únicos de fallo?**
MariaDB, Redis y HAProxy, además de contar con almacenamiento compartido y redundante.

**p. ¿Cómo cambiaría este laboratorio al implementarlo en Docker Swarm o Kubernetes?**
Se facilitaría la administración de réplicas, la detección de fallos, el reinicio automático y la distribución de solicitudes.

---

## 2. Evidencias

### 2.1 Usuario del sitio y propietario del directorio

`id orh` y `stat` sobre `/home/orh`.

![id orh y stat de /home/orh](img/lab2-004-000.png)

### 2.2 Archivos del proyecto y permisos

![chmod, .env y find de los archivos del proyecto](img/lab2-004-001.png)

![stat de wp-login.php](img/lab2-004-002.png)

### 2.3 Construcción y despliegue

```bash
docker compose up -d
```

![docker compose up -d](img/lab2-004-003.png)

![Contenedores, red y volúmenes creados](img/lab2-004-004.png)

### 2.4 Estado de los servicios y validación de HAProxy

![docker compose ps y validación de haproxy.cfg](img/lab2-004-005.png)

### 2.5 Identidad de `www-data` en ambos contenedores

![id www-data en web1 y web2 (UID/GID 1003)](img/lab2-004-006.png)

### 2.6 Escritura en el bind mount

![Archivos de prueba creados desde web1 y web2](img/lab2-004-007.png)

### 2.7 Instalación de WordPress

![Configuración de la conexión a la base de datos](img/lab2-005-008.png)

![Inicio de sesión en WordPress](img/lab2-006-009.png)

![Escritorio de WordPress](img/lab2-006-010.png)

### 2.8 Redis

Extensiones de PHP en ambos backends:

![php -m en web1 y web2](img/lab2-007-011.png)

Redis Object Cache conectado:

![Redis Object Cache: Status Connected](img/lab2-007-012.png)

Claves en la base lógica 0:

![redis-cli DBSIZE](img/lab2-007-013.png)

### 2.9 Indicador de backend (must-use plugin)

![backend-indicator.php en mu-plugins](img/lab2-007-014.png)

### 2.10 Prueba de balanceo de carga

Diez solicitudes alternando entre `web1` y `web2`:

![Diez solicitudes con curl](img/lab2-008-015.png)

### 2.11 HAProxy con ambos backends UP

![Estadísticas de HAProxy con web1 y web2 en UP](img/lab2-008-016.png)

### 2.12 Prueba de fallo de `web1`

![Barra de administración: Backend web1](img/lab2-008-017.png)

![docker compose stop web1 y respuesta desde web2](img/lab2-008-018.png)

La sesión continúa autenticada, ahora atendida por `web2`:

![Barra de administración: Backend web2](img/lab2-008-019.png)

### 2.13 Prueba de fallo de `web2`

![docker compose stop web2 y respuesta desde web1](img/lab2-008-020.png)

![Estadísticas de HAProxy con web2 en DOWN](img/lab2-008-021.png)

### 2.14 Propiedad de los archivos

![find de propiedad en /home/orh](img/lab2-008-022.png)

![Archivos que no tienen UID/GID 1003](img/lab2-008-023.png)
