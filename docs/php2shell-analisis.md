# PHP2Shell · Análisis

## Qué es

PHP2Shell es el nombre con el que se conocen dos vulnerabilidades críticas del lenguaje PHP: CVE-2026-109618 y CVE-2026-109619. Se hicieron públicas en octubre de 2026 y afectan al propio intérprete de PHP.

Estas vulnerabilidades pueden permitir que un atacante ejecute código de forma remota en un servidor sin necesidad de autenticarse. Esto se conoce como RCE (Remote Code Execution).

Están relacionadas con errores en la gestión de memoria de PHP:

- **CVE-2026-109618**: un error en la gestión de memoria cuando se agota la memoria disponible puede provocar corrupción de memoria.
- **CVE-2026-109619**: el intérprete puede conservar referencias a memoria que ya ha sido liberada y utilizarlas después.

Si un atacante consigue explotarlas, podría ejecutar comandos con los permisos del proceso que ejecuta PHP, acceder a información sensible o afectar al funcionamiento del servidor.

## Versiones afectadas y corregidas

Las versiones vulnerables abarcan varias ramas de PHP, incluidas versiones antiguas que ya no tienen soporte oficial.

| Rama de PHP | Versiones afectadas | Primera versión corregida |
|---|---|---|
| PHP 8.5 | Anteriores a 8.5.12 | 8.5.12 |
| PHP 8.4 | Anteriores a 8.4.27 | 8.4.27 |
| PHP 8.3 | Anteriores a 8.3.36 | 8.3.36 |
| PHP 8.2 | Anteriores a 8.2.35 | 8.2.35 |
| PHP 8.1, 8.0 y 7.4 | Anteriores a la corrección de seguridad | Parches específicos del proveedor |
| PHP 7.3 y anteriores | Todas | Sin parche oficial del proyecto PHP |

En Ubuntu 24.04 la corrección llega como parche de la distribución: el paquete `php8.3-cli` está corregido a partir de la versión `8.3.6-0ubuntu0.24.04.13`.

La solución recomendada es actualizar PHP a una versión corregida o instalar el parche de seguridad de la distribución. En versiones sin soporte es especialmente importante comprobar si existe un parche específico.

## Cómo afecta a nuestro servidor

Si nuestro servidor usa una versión vulnerable de PHP para atender peticiones web, podría estar expuesto a ataques remotos. El atacante podría aprovechar el fallo del intérprete para ejecutar código sin tener una cuenta de usuario.

Las principales consecuencias serían:

- **Pérdida de confidencialidad**: acceso a contraseñas, tokens u otros datos sensibles presentes en la memoria del proceso.
- **Pérdida de integridad**: modificación de archivos, aplicaciones o configuraciones a las que tenga acceso el proceso PHP.
- **Pérdida de disponibilidad**: caída de procesos o interrupción del servicio web.
- **Compromiso del servidor**: el atacante podría usar los permisos del proceso PHP para intentar acceder a otros recursos del sistema.

El impacto real depende de la versión instalada, de cómo se ejecute PHP y de los permisos del proceso. La vulnerabilidad no implica automáticamente que el atacante obtenga permisos de root.

Para proteger nuestro servidor hay que comprobar qué versión de PHP tiene instalada (`apt policy php8.3-cli`), instalar la actualización de seguridad y reiniciar los procesos afectados, como Apache o PHP-FPM. También conviene revisar los registros del servidor para detectar posibles intentos de explotación.

## Qué no está afectado

- Las instalaciones de PHP con los parches de seguridad aplicados.
- Los servidores que no ejecutan PHP para atender peticiones web.
- Los servidores web que usan solo otros lenguajes, como Java o Python, siempre que no tengan también una instalación vulnerable de PHP.
- Docker no es el origen de la vulnerabilidad, pero los contenedores que incluyen versiones vulnerables de PHP sí pueden verse afectados y deben actualizarse.
- Las aplicaciones no son vulnerables por ser WordPress, Drupal o Joomla: el problema está en PHP, así que también puede afectar a aplicaciones propias que usen una versión vulnerable.

**Conclusión:** PHP2Shell es una vulnerabilidad crítica que puede comprometer los servidores que ejecutan versiones vulnerables de PHP. La principal medida de protección es actualizar PHP a una versión corregida y comprobar que los procesos web usan esa versión.
