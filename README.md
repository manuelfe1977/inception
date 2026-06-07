*This project has been created as part of the 42 curriculum by manuelfe.*

# Inception

Inception es un proyecto de administración de sistemas que consiste en diseñar una infraestructura web segura y multi-contenedor utilizando Docker Compose. El objetivo es levantar de forma local un servidor NGINX con TLS, WordPress (mediante PHP-FPM) y MariaDB, totalmente aislados en redes y volúmenes protegidos bajo las directrices estrictas del sujeto.

### Description

Inception es el proyecto cumbre de la rama de Sistemas (SysAdmin) en la metodología de 42. Su propósito fundamental es enseñarte a diseñar, configurar y desplegar una arquitectura de microservicios profesional, segura y aislada utilizando Docker.

El proyecto te prohíbe taxativamente usar imágenes preconfiguradas de Docker Hub (como la imagen oficial de WordPress o MariaDB). En su lugar, te obliga a construir cada contenedor desde cero (usando Dockerfiles basados en Debian), instalando y configurando los servicios de manera nativa mediante scripts de Bash. Todo ello debe ser orquestado por un único archivo Docker Compose y ejecutarse dentro de una Máquina Virtual aislada.

La infraestructura obligatoria consta de tres contenedores independientes que se comunican entre sí en una red privada virtual:

NGINX (El Guardián Dedicado): Es el único contenedor que tiene los puertos abiertos al mundo exterior (puerto 443). Actúa como servidor web proxy inverso y solo acepta conexiones seguras cifradas mediante TLS v1.2 o TLS v1.3. El tráfico HTTP normal (puerto 80) está estrictamente prohibido.

WordPress + PHP-FPM (El Motor de Contenido): Aloja los archivos de la web. Como NGINX no sabe interpretar el código PHP, cuando llega una petición, este contenedor la procesa usando el servicio PHP-FPM a través del puerto interno 9000.

MariaDB (La Base de Datos): Almacena de forma persistente los usuarios, posts y configuraciones de la web. Está completamente aislado en la red interna de Docker (puerto 3306) y nunca puede ser accesible desde fuera de la máquina virtual.

**Conceptos clave**

Persistencia de Datos.
Politicas de reinicio.
Variables de entorno y seguridad.

**Virutal Machines vs Docker**

Ambas tecnologías ofrecen soluciones a problemas similares:

1. Entornos aislados. Permiten ejecutar aplicaciones en un entorno aislado.
2. Portabilidad. Puedes empaquetar tu VM o tu contenedor de Docker y funcionara igual en cualquier ordenador.
3. Gestión de recursos. En ambos sistemas puedes limitar el uso de recursos para no saturar al host anfitrion.

Se diferencian en:

1. Arquitectura interna:
	VM: Cada VM incluye un SO completo con su propio kernel, gestion de memoria, drivers, etc, ademas de la aplicación a ejecutar.
	Docker: Se ejecuta sobre Docker Engine y no incluye ningun sistema operativo completo, todos los contenedores comparten el kernel del host anfitrion. solo empaqueta la aplicación y las librerias binarias estrictamente necesarias para que corra.
2. Rendimiento y velocidad:
	VM: Tardara minutos en arrancar porque simula el encendido de un ordenador virtual y carga el sistema operativo completo. Consume mucha RAM solo para mantenerse encendida.
	Docker: Arranca en muy pocos segundos, al no tener que arrancar un SO completo consume una fraccion muy pequeña de RAM y CPU.
3. Tamaño en disco:
	VM: Necesitara al menos entre 10 y 20 GB de disco duro.
	Docker: Una imagen de Docker optimizada esta entre 50Mb y 200Mb.

**Secrets vs Environment Variables**

La diferencia fundamental se reduce a una cuestión de seguridad y arquitectura: mientras que las variables de entorno (environment) exponen tus datos en texto plano por todo el sistema, los Docker Secrets los protegen cifrándolos y montándolos como archivos temporales directamente en la memoria RAM del contenedor.

Las variables de entorno son excelentes para para configuraciones generales pero peligrosas para datos sensibles(contraseñas o claves privadas).
Cuando Docker levanta el contenedor toma las variables de .env y las inyecta como variables globales del sistema operativo del contenedor. estan flotando como texto plano, si alguien consigue acceso de lectura a tu terminal host, puede ejecutar un simple `docker inspect <nombre_contenedor>` o entrar al contenedor y escribir `env`, y verá todas tus contraseñas en texto plano en la pantalla. Además, se quedan guardadas en los metadatos del contenedor.

Los Docker secrets estan diseñados para proteger informacion critica. En lugar de inyectar texto en el entorno, Docker coge un archivo físico de tu máquina real (donde guardas la contraseña) y lo monta únicamente dentro de la memoria RAM del contenedor.
El archivo que contiene la contraseña se monta mediante un sistema de archivos volátil (tmpfs). Nunca se escribe en el disco duro del contenedor ni aparece en los metadatos. Si ejecutas `docker inspect`, la contraseña no aparece por ningún lado. Cuando el contenedor se apaga, el secreto se evapora de la memoria RAM al instante sin dejar rastro.


**Docker Network vs Host Network**

La diferencia principal radica en el aislamiento: por defecto, Docker aísla las conexiones del contenedor dentro de una red virtual, mientras que el modo `host` elimina por completo esa barrera y mete al contenedor directamente en la red de tu máquina real.
· Docker Network (`bridge`)
Docker crea una tarjeta de red virtual dentro de tu host (un puente llamado habitualmente `docker0`) y le asigna una IP privada interna a cada contenedor (por ejemplo, `172.18.0.x`).
Aislamiento total. Los contenedores no pueden ver los puertos la maquina fisica ni de los otros contenedores a menos que los conectes explicitamente a la misma red virtual
Mapeo de puertos obligatorio.
Resolución por nombre (DNS).
· Host Network (`host`)
En este modo, se desactiva por completo el aislamiento de red del contenedor. El contenedor no recibe una IP propia de Docker; en su lugar, utiliza directamente la IP y la tarjeta de red de tu máquina real.
Sin barreras.
No hay mapeo de puertos.
Conflicto de puertos.
Mayor rendimiento, al no hacer traducciones de direcciones de red (NAT).

**Docker Volumes vs Bind Mounts**

· Bind mounts
Consiste en mapear una ruta exacta y especifica de tu host dentro del contendor. El usuario tiene control total de la carpeta, tiene acceso desde el host y el contenedor vera los cambios al instante. Depende por completo de la estructura de carpetas del host, por lo que si mueves el contenedor de ordenador el contenedor fallara si no existe la ruta.

. Docker volumes
No le damos una ruta exacta, simplemente le damos un nombre al volumen, Docker crea una carpeta en el disco duro solo gestionada por el. No importa la ruta fisica por tanto es 100% portable. Los usuarios normales no pueden tener acceso a esas carpetas.

### Instructions


### Resources


