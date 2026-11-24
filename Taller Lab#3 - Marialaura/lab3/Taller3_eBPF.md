# Taller #3: Observabilidad del kernel con eBPF en una plataforma WordPress de alta disponibilidad

**Universidad Latina de Costa Rica** · Sistemas Operativos II

- **Estudiante:** Marialaura Calderón Herrera
- **Docente:** Carlos Andrés Mendez Rodríguez
- **Sede:** San Pedro, San José
- **Fecha:** noviembre, 2026

---

## 1. Preguntas de análisis

### Práctica 1: contenedores y procesos reales

**A. ¿El contenedor tiene un kernel propio?**
No, comparte el kernel Linux del sistema anfitrión.

**B. ¿Por qué HAProxy aparece como un proceso normal del sistema?**
Porque los contenedores ejecutan procesos Linux reales sobre el kernel del host.

**C. ¿Qué namespaces se observan?**
Se pueden observar los de procesos, red, usuarios, montajes, IPC y UTS.

**D. ¿Qué relación hay entre el proceso del contenedor y los cgroups?**
Los cgroups permiten controlar y contabilizar los recursos que utiliza el proceso.

### Práctica 2: creación de procesos

**E. ¿Por qué `docker compose exec` genera eventos de creación y ejecución de procesos en el host?**
Porque crea nuevos procesos dentro del contenedor que son ejecutados por el kernel compartido.

### Práctica 3: llamadas al sistema

**F. ¿Qué procesos generan más llamadas al sistema?**
Generalmente Apache, HAProxy y los procesos que atienden las solicitudes.

**G. ¿Por qué aparece `curl` si la aplicación está en contenedores?**
Porque `curl` se ejecuta desde el host y también realiza llamadas al kernel.

**H. ¿Por qué puede aparecer `apache2`, `haproxy`, `redis-server` o `mariadbd`?**
Porque todos utilizan llamadas al sistema para realizar sus operaciones.

**I. ¿Qué indica que un proceso tenga muchas llamadas `futex`, `epoll_wait`, `read` o `write`?**
Indica que realiza operaciones de sincronización, espera de eventos, lectura o escritura.

### Práctica 5: `strace` y eBPF

**J. ¿Cuál proceso muestra más actividad: el maestro o el worker?**
Generalmente el worker, porque atiende las conexiones y solicitudes HTTP.

**K. ¿Por qué `epoll_wait()` aparece en servidores de red como HAProxy?**
Porque permite esperar y detectar eventos de múltiples conexiones de red.

**L. ¿Qué diferencia hay entre adjuntarse a un PID específico y observar eventos globales del kernel?**
`strace` observa un proceso específico, mientras que eBPF permite observar eventos de distintos procesos.

### Práctica 6: conexiones TCP

**M. ¿Qué procesos invocan `connect()`?**
Los procesos que necesitan establecer conexiones con otros servicios.

**N. ¿Por qué algunas conexiones parecen locales?**
Porque utilizan interfaces locales o la red interna de Docker.

**O. ¿Qué diferencia hay entre una conexión cliente-HAProxy y una conexión HAProxy-backend?**
La primera recibe solicitudes externas y la segunda comunica el balanceador con los servidores web.

**P. ¿Qué aporta `bpftrace` y qué aporta `ss`?**
`bpftrace` observa eventos de conexión y `ss` muestra información de los sockets activos.

### Práctica 7: sesiones PHP con Redis

**Q. ¿Por qué Redis permite que la sesión sobreviva aunque HAProxy envíe solicitudes a `web1` o `web2`?**
Porque ambos servidores pueden consultar las sesiones almacenadas en Redis.

**R. ¿Qué problema habría si las sesiones se guardaran solo en archivos locales dentro de cada contenedor web?**
Cada servidor tendría sesiones independientes y podría perderse la continuidad al cambiar de servidor.

**S. ¿Qué relación tiene esto con alta disponibilidad?**
Redis permite compartir las sesiones entre servidores y mantener la continuidad del servicio.

### Práctica 8: actividad de disco

**T. ¿Cuál operación produjo más actividad de bloque?**
La operación que generó más escrituras en el almacenamiento, según los conteos obtenidos.

**U. ¿La actividad de disco observada corresponde únicamente al contenedor?**
No, corresponde al kernel compartido y puede incluir actividad de otros procesos.

**V. En WSL2, ¿qué capas adicionales podrían influir en la latencia?**
Docker, el sistema de archivos virtualizado y la capa de almacenamiento de WSL2.

**W. ¿Por qué una escritura dentro del contenedor termina observándose como actividad del kernel compartido?**
Porque el contenedor utiliza los mismos mecanismos de entrada y salida del kernel del host.

### Práctica 9: planificación de CPU

**X. ¿El planificador agenda contenedores o procesos/hilos?**
Agenda procesos e hilos, no contenedores directamente.

**Y. ¿Qué procesos aparecen con mayor cantidad de cambios de contexto?**
Los procesos que realizan más actividad o necesitan alternar frecuentemente el uso de CPU.

**Z. ¿Cómo se relaciona esto con cargas CPU-bound, I/O-bound y tiempo de respuesta?**
Las cargas CPU-bound necesitan más tiempo de procesamiento, mientras que las I/O-bound suelen esperar operaciones de entrada y salida.

**AA. ¿Por qué `docker stats` y `bpftrace` muestran perspectivas distintas del mismo fenómeno?**
`docker stats` muestra consumo de recursos por contenedor y `bpftrace` observa eventos del kernel.

### Práctica 10: UID/GID y permisos

**BB. ¿Por qué conviene evitar que los archivos queden propiedad de `root`?**
Porque los servicios podrían no tener permisos para modificarlos.

**CC. ¿Qué problema se busca resolver al hacer coincidir UID/GID del contenedor con el host?**
Se busca mantener permisos correctos sobre los archivos compartidos.

**DD. ¿Por qué no conviene hacer `chown -R` automático sobre todo `/var/www/html` cada vez que inicia el contenedor?**
Porque puede modificar innecesariamente muchos permisos y aumentar el tiempo de inicio.

---

## Preguntas finales de reflexión

1. **¿Qué diferencia hay entre observar una aplicación desde sus logs y observarla desde el kernel?**
   Los logs muestran información de la aplicación y el kernel permite observar eventos internos del sistema operativo.

2. **¿Por qué eBPF es útil en ambientes con contenedores?**
   Porque permite observar procesos y recursos de los contenedores desde el kernel compartido.

3. **¿Qué eventos del laboratorio corresponden a procesos?**
   La creación, ejecución y planificación de procesos.

4. **¿Qué eventos corresponden a red TCP/IP?**
   Las conexiones, aceptaciones y transferencias de datos mediante llamadas como `connect()` y `accept4()`.

5. **¿Qué eventos corresponden a almacenamiento?**
   Las solicitudes de lectura y escritura observadas mediante tracepoints de bloque.

6. **¿Qué eventos corresponden al planificador de CPU?**
   Los cambios de contexto registrados mediante `sched_switch`.

7. **¿Qué información sensible podría exponerse mediante trazas eBPF?**
   Nombres de archivos, conexiones de red, rutas internas y parámetros de procesos.

8. **¿Cómo se relaciona este laboratorio con los temas de procesos, memoria, I/O, red y seguridad del sistema operativo?**
   Permite observar cómo el kernel administra los procesos, recursos, comunicaciones y permisos de las aplicaciones.

---

## 2. Evidencias

### 2.1 Preparación del ambiente

**Sistema operativo y kernel** (`hostnamectl`, `uname -a`):

![hostnamectl y uname -a](img/lab3-004-000.png)

**Versión y soporte de eBPF** (`bpftrace --version`, `sudo bpftrace --info`):

![bpftrace --version y --info](img/lab3-004-001.png)

**BTF y tracepoints de syscalls:**

![BTF y tracepoints syscalls](img/lab3-004-002.png)

**Tracepoints de `sched` y `block`:**

![Tracepoints sched y block](img/lab3-005-003.png)

### 2.2 Verificación del laboratorio WordPress HA

**Estado de los servicios** (`docker compose up -d`, `docker compose ps`):

![docker compose up -d y ps](img/lab3-005-004.png)

**Respuesta HTTP y balanceo por HAProxy** (el encabezado `x-backend` alterna entre `web1` y `web2`):

![curl -I y balanceo con X-Backend](img/lab3-006-005.png)

**Estadísticas de HAProxy:**

![curl a las estadísticas de HAProxy](img/lab3-006-006.png)

### 2.3 Práctica 1: relacionar contenedores con procesos reales

**PID real de cada contenedor:**

![PID_HOST de cada contenedor](img/lab3-006-007.png)

**Procesos en el host, namespaces y cgroups de HAProxy:**

![ps, lsns y cgroup de HAProxy](img/lab3-006-008.png)

### 2.4 Práctica 2: creación de procesos

**Traza de `FORK` y `EXEC` con `bpftrace`:**

![bpftrace FORK y EXEC](img/lab3-007-009.png)

**Comandos ejecutados en los contenedores para generar procesos:**

![docker compose exec en web1, redis y mariadb](img/lab3-007-010.png)

**Traza de `EXEC` mientras se genera tráfico HTTP:**

![bpftrace EXEC con tráfico curl](img/lab3-008-011.png)

### 2.5 Práctica 3: llamadas al sistema durante tráfico HTTP

**Conteo de llamadas al sistema por proceso:**

![Conteo de syscalls por comm](img/lab3-009-012.png)

**Filtrado solo para `apache2`:**

![Syscalls de apache2 por tracepoint](img/lab3-010-013.png)

### 2.6 Práctica 4: archivos abiertos por Apache

![openat de apache2](img/lab3-010-014.png)

### 2.7 Práctica 5: comparación entre `strace` y eBPF

**Generación de tráfico:**

![Bucles de curl](img/lab3-011-015.png)

**PID del maestro y workers de HAProxy:**

![PIDs de HAProxy](img/lab3-011-016.png)

**Salida de `strace` sobre HAProxy:**

![strace sobre HAProxy](img/lab3-011-017.png)

**Solicitud con `curl -v`:**

![curl -v con Connection: close](img/lab3-012-018.png)

**Eventos de red globales con `bpftrace`** (`accept4`, `connect`, `sendto`, `recvfrom`):

![Conteo de eventos de red por proceso](img/lab3-012-019.png)

### 2.8 Práctica 6: conexiones TCP

**Intentos de conexión con `connect()`:**

![bpftrace connect](img/lab3-013-020.png)

**Conteo de conexiones por proceso:**

![Conteo de connect por comm](img/lab3-014-021.png)

**Conexiones abiertas por HAProxy:**

![HAProxy abre conexión](img/lab3-015-022.png)

**Tráfico generado:**

![Bucles de curl con Connection: close](img/lab3-015-023.png)

**Sockets con `ss` y red Docker:**

![ss y docker network inspect](img/lab3-016-024.png)

### 2.9 Práctica 7: sesiones PHP con Redis

**Consumo de la página de prueba por HAProxy:**

![curl a ebpf-session-test.php](img/lab3-016-025.png)

**Claves en Redis (base 1):**

![redis-cli keys y DBSIZE](img/lab3-017-026.png)

**Actividad TCP mientras se usan las sesiones:**

![Conteo de connect durante la prueba](img/lab3-017-027.png)

![Detalle del conteo de connect](img/lab3-017-028.png)

### 2.10 Práctica 8: actividad de disco

**Tracepoints de bloque disponibles:**

![Tracepoints block_rq_issue y block_rq_complete](img/lab3-018-029.png)

**Conteo de actividad de bloque por proceso:**

![Conteo de block_rq_issue por comm](img/lab3-018-030.png)

**Escrituras provocadas en WordPress, Redis y MariaDB:**

![Escrituras en web1, Redis y MariaDB](img/lab3-019-031.png)

**Histograma de latencia de bloque:**

![Histograma de latencia en microsegundos](img/lab3-019-032.png)

### 2.11 Práctica 9: planificación de CPU

**Cambios de contexto con `sched_switch`:**

![Conteo de sched_switch por comm](img/lab3-020-033.png)

**Comparación con `docker stats` y `vmstat`:**

![docker stats y vmstat](img/lab3-020-034.png)

### 2.12 Comandos de cierre

![docker compose ps, curl -I y estadísticas de HAProxy](img/lab3-021-035.png)

### 2.13 Práctica 10: UID/GID y permisos

![id www-data, procesos Apache y stat de /var/www/html](img/lab3-021-036.png)
