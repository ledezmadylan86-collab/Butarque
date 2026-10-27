PHP2Shell · Análisis
1. Qué es
PHP2Shell es el nombre con el que se conocen dos vulnerabilidades críticas de seguridad del lenguaje PHP: CVE-2026-109618 y CVE-2026-109619. Se hicieron públicas en octubre de 2026 y afectan al propio intérprete de PHP. 
Plesk+1


Estas vulnerabilidades pueden permitir que un atacante ejecute código de forma remota en un servidor sin necesidad de autenticarse. Esto se conoce como RCE (Remote Code Execution).

Las vulnerabilidades están relacionadas con errores en la gestión de memoria de PHP:
- CVE-2026-109618: un error durante la gestión de memoria cuando se agota la memoria disponible puede provocar corrupción de memoria.
- CVE-2026-109619: el intérprete puede conservar referencias a memoria que ya ha sido liberada y utilizarlas posteriormente.
Si un atacante consigue explotarlas, podría ejecutar comandos con los permisos del proceso que ejecuta PHP, acceder a información sensible o afectar al funcionamiento del servidor. 
Quantum+1

2. Versiones afectadas y corregidas
Las versiones vulnerables abarcan distintas ramas de PHP, incluidas versiones antiguas que ya no reciben soporte oficial.
Versión de PHP	Versiones afectadas	Primera versión corregida
PHP 8.5	Anteriores a 8.5.12	8.5.12
PHP 8.4	Anteriores a 8.4.27	8.4.27
PHP 8.3	Anteriores a 8.3.36	8.3.36
PHP 8.2	Anteriores a 8.2.35	8.2.35
PHP 8.1, 8.0 y 7.4	Versiones anteriores a la corrección de seguridad correspondiente	Requieren parches específicos del proveedor
PHP 7.3 y anteriores	Versiones vulnerables sin corrección oficial general	Sin parche oficial del proyecto PHP


Plesk+2
La solución recomendada es actualizar PHP a una versión corregida o instalar el parche de seguridad oficial de la distribución o proveedor de alojamiento. En versiones sin soporte, es especialmente importante comprobar si existe un parche específico.
3. Cómo afecta a nuestro servidor
Si nuestro servidor utiliza una versión vulnerable de PHP para atender peticiones web, podría estar expuesto a ataques remotos. El atacante podría aprovechar un fallo del intérprete para ejecutar código sin disponer de una cuenta de usuario.

Las principales consecuencias serían:
- Pérdida de confidencialidad: acceso a contraseñas, tokens u otros datos sensibles que estén presentes en la memoria del proceso.
- Pérdida de integridad: modificación de archivos, aplicaciones o configuraciones a las que tenga acceso el proceso PHP.
- Pérdida de disponibilidad: caída de procesos o interrupción del servicio web.
- Compromiso del servidor: el atacante podría utilizar los permisos del proceso PHP para intentar acceder a otros recursos del sistema.
El impacto real depende de la versión instalada, de cómo se ejecute PHP y de los permisos asignados al proceso. La vulnerabilidad no implica automáticamente que el atacante obtenga permisos de administrador o de root. 
Quantum+1


Para proteger nuestro servidor debemos comprobar qué versión de PHP utiliza realmente el servicio web, instalar la actualización de seguridad correspondiente y reiniciar los procesos afectados, como Apache o PHP-FPM. También conviene revisar los registros del servidor para detectar posibles intentos de explotación.
4. Qué no está afectado
- Las instalaciones de PHP con los parches de seguridad aplicados están protegidas frente a estas vulnerabilidades concretas.
- Los servidores que no ejecutan PHP para atender peticiones web no están expuestos directamente a este fallo del intérprete.
- Los servidores web que utilizan exclusivamente otros lenguajes, como Java o Python, no se ven afectados directamente, siempre que no ejecuten también una instalación vulnerable de PHP.
- Docker no es el origen de la vulnerabilidad. Sin embargo, los contenedores que incluyen versiones vulnerables de PHP sí pueden verse afectados y deben actualizarse.
- Las aplicaciones no son vulnerables por ser WordPress, Drupal o Joomla específicamente. El problema se encuentra en PHP, por lo que también puede afectar a aplicaciones propias y a otros sistemas que utilicen una versión vulnerable. 
  Docker+1
Conclusión: PHP2Shell es una vulnerabilidad crítica que puede comprometer la seguridad de los servidores que ejecutan versiones vulnerables de PHP. La principal medida de protección consiste en actualizar PHP a una versión corregida y verificar que los procesos web estén utilizando esa actualización.
