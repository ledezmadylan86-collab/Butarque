# 3 · Puesta en marcha

## Qué hace

Esta parte explica cómo se levanta el portal WordPress de Butarque con Docker Compose, cómo se completa el asistente de instalación y cómo se comprueba que WordPress usa la base de datos con su propio usuario (app_butarque) y no con root. Corresponde a los pasos 5, 6 y 7.

## Cómo lo he hecho

### Paso 5 · Levantar el portal

Desde la carpeta del portal, con el `compose.yaml` y el `.env` ya preparados:

```
cd ~/butarque-portal
sudo docker compose up -d
sudo docker compose ps
```

Al levantar, Compose creó la red `butarque-portal_default`, los volúmenes `butarque-portal_datos-db` y `butarque-portal_datos-wordpress` y los dos contenedores.

### Paso 6 · El asistente de WordPress

Abrí en el navegador `http://IP-DE-TU-VM:8080` y completé el asistente:

1. Idioma: Español.
2. Datos del sitio:
   - Título del sitio: Portal de Butarque
   - Nombre de usuario: admin_butarque
   - Contraseña: TU-CLAVE-DE-ADMIN
   - Correo electrónico: webmaster@butarque.example
3. Pulsé «Instalar WordPress» y después «Acceder».
4. Entré con admin_butarque y se abrió el Escritorio en `http://IP-DE-TU-VM:8080/wp-admin`.

### Paso 7 · Comprobar la base con el usuario de la aplicación

Dentro del servicio `db`, entrando como app_butarque (pide la contraseña `DB_APP_PASSWORD` del `.env`):

```
sudo docker compose exec db mariadb -u app_butarque -p -e "SHOW DATABASES;"
sudo docker compose exec db mariadb -u app_butarque -p butarque -e "SHOW TABLES;"
```

## Documentación oficial que he leído

- Docker Compose Quickstart → Step 2: Define and start your services y Step 7: Inspect and debug your running stack (incluido «Run commands inside a running container»): https://docs.docker.com/compose/gettingstarted/
- docker compose up → la opción `-d`: https://docs.docker.com/reference/cli/docker/compose/up/
- docker compose ps → qué significa cada columna: https://docs.docker.com/reference/cli/docker/compose/ps/
- docker compose logs → ver los registros de un servicio: https://docs.docker.com/reference/cli/docker/compose/logs/
- How to install WordPress → el script de instalación: https://developer.wordpress.org/advanced-administration/before-install/howto-install/
- docker compose exec → su sintaxis: https://docs.docker.com/reference/cli/docker/compose/exec/
- Installing and Using MariaDB via Docker → cómo se entra con el cliente mariadb: https://mariadb.com/docs/server/server-management/automated-mariadb-deployment-and-administration/docker-and-mariadb/installing-and-using-mariadb-via-docker
- Imagen de MariaDB (Docker Hub) → Connect to MariaDB from the MariaDB command line client: https://hub.docker.com/_/mariadb

## Por qué lo he hecho así

- Uso `up -d` para arrancar los contenedores en segundo plano y que la terminal quede libre.
- En `docker compose ps`, la columna PORTS muestra qué está publicado: `wordpress` sale con `0.0.0.0:8080->80/tcp` porque el puerto 8080 de la VM lleva al 80 del contenedor. `db` sale solo con `3306/tcp`, sin flecha, porque no publica puertos: solo se llega a ella desde la red interna de Compose.
- El asistente no me pidió los datos de la base (servidor, usuario, contraseña y nombre), a diferencia del de la documentación oficial. Ya se los había dado a WordPress en el `compose.yaml`, con las variables `WORDPRESS_DB_HOST` (db), `WORDPRESS_DB_USER` (app_butarque), `WORDPRESS_DB_PASSWORD` y `WORDPRESS_DB_NAME` (butarque). Por eso solo pide los datos del sitio.
- Uso `docker compose exec db` para ejecutar el cliente `mariadb` dentro del contenedor de la base, porque la base no publica puertos y no se puede llegar a ella desde fuera.
- Entro con app_butarque y no con root para comprobar que la aplicación solo ve su propia base.

## Cómo se comprueba

`sudo docker compose ps`: los dos servicios en Up, `wordpress` publicado en el 8080 y `db` sin publicar:

```
NAME                          IMAGE                     SERVICE     STATUS          PORTS
butarque-portal-db-1          mariadb:11.4              db          Up 12 seconds   3306/tcp
butarque-portal-wordpress-1   wordpress:php8.3-apache   wordpress   Up 12 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp
```

`http://IP-DE-TU-VM:8080/wp-admin` abre el Escritorio de WordPress al entrar con admin_butarque.

`SHOW DATABASES` como app_butarque: solo ve su base y la del sistema:

```
+--------------------+
| Database           |
+--------------------+
| butarque           |
| information_schema |
+--------------------+
```

`SHOW TABLES` en butarque: las doce tablas que creó el asistente:

```
+-----------------------+
| Tables_in_butarque    |
+-----------------------+
| wp_commentmeta        |
| wp_comments           |
| wp_links              |
| wp_options            |
| wp_postmeta           |
| wp_posts              |
| wp_term_relationships |
| wp_term_taxonomy      |
| wp_termmeta           |
| wp_terms              |
| wp_usermeta           |
| wp_users              |
+-----------------------+
```

## Problemas: causa, solución y comprobación

No tuve problemas: los dos servicios arrancaron a la primera, el asistente apareció sin errores y las comprobaciones de la base dieron el resultado esperado.

Si WordPress muestra «Error al establecer una conexión con la base de datos» justo después de levantar, la causa es que `db` todavía está arrancando. La solución es esperar unos segundos y recargar. Se comprueba con `sudo docker compose logs db`: cuando la base aparece como lista para conexiones, el portal carga.





