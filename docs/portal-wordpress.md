# Parte 2 · WordPress

## Qué hace
El servicio `wordpress` ejecuta el portal web de la Mesa de Butarque con la imagen
oficial `wordpress:php8.3-apache`. Se publica en el puerto 8080 de la VM
(http://IP-DE-TU-VM:8080), se conecta a la base de datos `butarque` del servicio `db`
con el usuario de la aplicación `app_butarque` (nunca con root) y guarda sus ficheros
en el volumen `wp_data` para no perderlos al parar los contenedores.

## Cómo lo he hecho
En `~/butarque-portal/compose.yaml` he añadido el servicio `wordpress` y, al final,
los volúmenes:

```yaml
  wordpress:
    image: wordpress:php8.3-apache
    restart: unless-stopped
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: app_butarque
      WORDPRESS_DB_PASSWORD: ${DB_APP_PASSWORD}
      WORDPRESS_DB_NAME: butarque
    volumes:
      - wp_data:/var/www/html
    depends_on:
      - db

volumes:
  db_data:
  wp_data:
```

La contraseña no está escrita en el fichero: Compose la lee de la variable
`DB_APP_PASSWORD` del fichero `.env`, que está en la misma carpeta y no se sube al
repositorio (está en `.gitignore`).

El `compose.yaml` completo, sin contraseñas, está en `portal/compose.yaml`.

## Documentación oficial que he leído
- Docker Compose Quickstart → Step 2: Define and start your services y Step 7
  (un servicio llega a otro por su nombre) — https://docs.docker.com/compose/gettingstarted/
- Imagen de WordPress (Docker Hub) → How to use this image (variables `WORDPRESS_DB_…`)
  — https://hub.docker.com/_/wordpress
- Networking in Compose → Default network and service discovery
  — https://docs.docker.com/compose/how-tos/networking/

## Por qué lo he hecho así
**a) ¿Qué cuatro variables necesita WordPress para conectarse?**
`WORDPRESS_DB_HOST`, `WORDPRESS_DB_USER`, `WORDPRESS_DB_PASSWORD` y `WORDPRESS_DB_NAME`.

**b) ¿Qué pones en `WORDPRESS_DB_HOST` y por qué funciona?**
Pongo `db`, el nombre del servicio de la base de datos. Funciona porque Compose crea
una red por defecto para todos los servicios del proyecto y cada contenedor puede
encontrar a los demás por el nombre de su servicio (DNS interno de Docker). Así no
hace falta saber la IP del contenedor ni publicar el puerto de la base.

**c) ¿Cómo se escribe «el 8080 de la VM al 80 del contenedor»?**
`"8080:80"` en `ports:`. El primer número es el puerto de la VM (host) y el segundo
el del contenedor, donde escucha Apache.

**d) ¿Qué carpeta de WordPress va en un volumen?**
`/var/www/html`, donde están los ficheros de WordPress (`wp-config.php`, temas,
plugins e imágenes subidas). Va en el volumen `wp_data` para que no se pierdan al
parar o borrar el contenedor.

Además:
- WordPress usa `app_butarque` y no root, para que la aplicación solo tenga acceso
  a su propia base.
- `depends_on: db` hace que la base arranque antes que WordPress.

## Cómo se comprueba
1. `sudo docker compose config --quiet` → no muestra errores ni avisos.
2. `sudo docker compose config --services` → muestra `db` y `wordpress`.
3. `sudo docker compose ps` → `wordpress` aparece con `0.0.0.0:8080->80/tcp`.
4. En el navegador, http://IP-DE-TU-VM:8080/wp-admin abre el Escritorio con
   `admin_butarque`.
5. En el repositorio, `portal/compose.yaml` solo tiene `${…}` en las contraseñas y
   no hay ningún `.env`.

## Problemas: causa, solución y comprobación
**Problema:** al comprobar con `sudo docker compose config --quiet` salió
`service "db" refers to undefined volume db_data: invalid compose project`.
**Causa:** el servicio `db` usaba el volumen `db_data`, pero no estaba declarado en
el bloque `volumes:` de nivel superior.
**Solución:** añadí al final del fichero el bloque `volumes:` con `db_data:` y
`wp_data:`.
**Comprobación:** al repetir `sudo docker compose config --quiet` ya no sale el error.
