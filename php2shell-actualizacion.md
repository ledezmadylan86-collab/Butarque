Para comprobar y parchear PHP2Shell necesitamos:

- Una máquina virtual con Ubuntu Server 24.04 y conexión a Internet.
- Acceso a una terminal con permisos `sudo`.
- Apache instalado y, si el servidor utiliza PHP, el conector correspondiente: `mod_php`, PHP-FPM o `php-cgi`.
- Los repositorios de paquetes de Ubuntu disponibles para instalar las actualizaciones de seguridad.

PHP2Shell afecta al intérprete PHP y permite ejecutar código en el servidor sin autenticación. Para solucionarlo, es necesario actualizar PHP a una versión que incluya el parche. Si no es posible actualizarlo, la medida indicada es desconectar la web afectada.

## Cómo saber si un servidor está parcheado

**1. Identificar el conector entre Apache y PHP.**

Ejecutamos los siguientes comandos:

```bash
sudo apache2ctl -M | grep -i php
systemctl is-active php8.3-fpm
```

El primer comando muestra los módulos de Apache relacionados con PHP. Si aparece `php_module (shared)`, el servidor utiliza `mod_php`, que es el conector más expuesto según la documentación del problema.

El segundo comando comprueba si el servicio PHP-FPM está activo. Si devuelve `active`, estamos utilizando PHP-FPM. Si no aparece ningún módulo o servicio activo, debemos comprobar qué configuración utiliza el servidor.

**2. Consultar la versión de PHP instalada.**

```bash
php -v
```

Este comando muestra la versión de PHP, pero no permite confirmar por sí solo si el parche está aplicado, porque Ubuntu puede incorporar la corrección sin cambiar la versión principal de PHP.

**3. Actualizar la lista de paquetes disponibles.**

```bash
sudo apt update
```

Este comando descarga la información más reciente de los repositorios configurados para conocer las versiones de los paquetes disponibles.

**4. Comprobar la versión instalada del paquete.**

```bash
apt policy php8.3-cli
```

El comando muestra la versión instalada y las versiones disponibles en los repositorios. En Ubuntu Server 24.04, según el procedimiento del sprint, la versión corregida de referencia es `8.3.6-0ubuntu0.24.04.13` o superior.

Si la versión instalada es igual o superior a esa referencia, el paquete ya está parcheado. Si es inferior, debemos actualizarlo.

Si aparece `(none)` en la versión instalada, significa que ese paquete no está instalado. Si no tenemos PHP instalado, la VM no está afectada por esta vulnerabilidad de PHP; debemos dejar constancia de ello en la documentación.

## Cómo se parchea

Si el paquete instalado tiene una versión anterior a la corregida, ejecutamos:

```bash
sudo apt install --only-upgrade php8.3-cli php8.3-common libapache2-mod-php8.3 php8.3-fpm php8.3-cgi
```

Este comando actualiza los paquetes indicados que ya están instalados, sin instalar los que no lo están. La opción `--only-upgrade` permite actualizar los paquetes existentes sin añadir automáticamente los que falten.

Los paquetes incluidos corresponden al intérprete PHP de línea de comandos, los componentes comunes de PHP y los conectores o modos de ejecución para Apache, PHP-FPM y CGI.

Después de actualizar PHP, reiniciamos Apache:

```bash
sudo systemctl restart apache2
```

El reinicio permite que Apache vuelva a cargar PHP con los componentes actualizados.

Si el servidor utiliza PHP-FPM, también reiniciamos su servicio:

```bash
sudo systemctl restart php8.3-fpm
```

La actualización del paquete es la medida necesaria para corregir la vulnerabilidad. Si no podemos actualizar PHP, debemos desconectar la web afectada.

## Cómo se comprueba después

**1. Volver a consultar la versión instalada.**

```bash
apt policy php8.3-cli
```

Comprobamos que la versión instalada sea `8.3.6-0ubuntu0.24.04.13` o superior. Esta consulta permite verificar que el paquete de Ubuntu contiene la actualización de seguridad indicada en el procedimiento.

**2. Comprobar el estado de los servicios.**

```bash
sudo systemctl status apache2
systemctl is-active php8.3-fpm
```

El primer comando permite comprobar el estado de Apache. El segundo permite verificar si PHP-FPM está activo, cuando se utiliza ese conector.

**3. Registrar los resultados.**

Debemos anotar el conector utilizado por Apache y la versión del paquete `php8.3-cli` antes y después de la actualización. Si PHP no está instalado, también debemos documentarlo.

**Resultados de nuestra VM:**

- Conector Apache-PHP: [indicar el resultado obtenido].
- Versión instalada antes de actualizar: [indicar la versión o `none`].
- Versión instalada después de actualizar: [indicar la versión o `none`].
- Estado final de Apache: [indicar el resultado].
- Estado final de PHP-FPM, si corresponde: [indicar el resultado].
- Resultado: [parche aplicado y comprobado / PHP no instalado].

Con estas comprobaciones queda documentado el procedimiento seguido y el estado de PHP en nuestra máquina virtual.
