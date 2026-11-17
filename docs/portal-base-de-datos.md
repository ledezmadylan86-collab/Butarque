# La base de datos

## Qué hace
Despliega el servicio `db` utilizando la imagen oficial de MariaDB (`mariadb:11.4`) para dar soporte al portal WordPress de Butarque. Este servicio se encarga de inicializar automáticamente la base de datos `butarque` junto con el usuario restringido `app_butarque`, persistir toda la información mediante un volumen de Docker y mantenerse aislado en la red interna sin publicar puertos hacia el exterior.

## Cómo lo he hecho
1. Creación de la carpeta del proyecto en la máquina virtual (`~/butarque-portal`) y definición del servicio `db` en el fichero `compose.yaml` usando interpolación de variables:

```yaml
services:
  db:
    image: mariadb:11.4
    container_name: butarque-portal-db-1
    restart: unless-stopped
    environment:
      MARIADB_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MARIADB_DATABASE: butarque
      MARIADB_USER: app_butarque
      MARIADB_PASSWORD: ${DB_APP_PASSWORD}
    volumes:
      - db-data:/var/lib/mysql

volumes:
  db-data:
```

2. Creación del fichero `.env` en la misma carpeta (`~/butarque-portal`) para almacenar las credenciales sin exponerlas en el código, y restricción de sus permisos de lectura con `chmod 600 .env`:

```env
DB_ROOT_PASSWORD=TU-CLAVE-DE-ADMINISTRACION
DB_APP_PASSWORD=TU-CLAVE-DE-LA-APLICACION
```

## Documentación oficial que he leído
* [Docker Compose Quickstart](https://docs.docker.com/compose/gettingstarted/) -> *Step 1: Set up the project* (punto 5, fichero `.env`), *Step 2: Define and start your services* y *Step 5: Persist data with named volumes*.
* [Installing and Using MariaDB via Docker](https://mariadb.com/docs/server/server-management/automated-mariadb-deployment-and-administration/docker-and-mariadb/installing-and-using-mariadb-via-docker) -> Arranque de MariaDB con variables de entorno.
* [Imagen oficial de MariaDB en Docker Hub](https://hub.docker.com/_/mariadb) -> *Environment Variables* y *Where to Store Data*.
* [Set, use, and manage variables in a Compose file with interpolation](https://docs.docker.com/compose/how-tos/environment-variables/variable-interpolation/) -> *Interpolation syntax* y *.env file*.

## Por qué lo he hecho así
* **Variable obligatoria:** Es imprescindible definir `MARIADB_ROOT_PASSWORD` para establecer la contraseña de administración de `root`; sin ella el contenedor no arranca por seguridad.
* **Creación de base de datos y usuario de aplicación:** Se emplean las tres variables `MARIADB_DATABASE` (crea la base `butarque`), `MARIADB_USER` (crea el usuario `app_butarque`) y `MARIADB_PASSWORD` (asigna su contraseña). De esta forma, WordPress nunca se conecta utilizando `root`. Estas variables solo se aplican en el primer arranque cuando el volumen está vacío.
* **Almacenamiento de datos:** MariaDB guarda sus datos internos en la ruta `/var/lib/mysql` del contenedor, la cual se monta sobre el volumen con nombre `db-data` para asegurar que la información no se pierda al detener o recrear los contenedores.
* **Aislamiento de red (sin `ports`):** Al omitir la sección `ports` en el servicio `db`, la base de datos no queda expuesta a la red externa ni a la VM; únicamente los contenedores conectados a la misma red interna de Docker Compose (como el servicio `wordpress`) pueden comunicarse con ella.
* **Interpolación y fichero `.env`:** Docker Compose sustituye las expresiones `${DB_ROOT_PASSWORD}` y `${DB_APP_PASSWORD}` por los valores definidos en el fichero `.env` (ubicado en el mismo directorio que `compose.yaml` y escrito en formato `CLAVE=VALOR`). Esto garantiza que ninguna contraseña real quede escrita en `compose.yaml` ni se suba al repositorio de GitHub.

## Cómo se comprueba
1. Validar la sintaxis del fichero de composición sin errores ni avisos de variables faltantes:
```bash
sudo docker compose config --quiet
```

2. Comprobar que las variables se interpolan correctamente en la configuración final pero permanecen ocultas en el fichero fuente:
```bash
sudo docker compose config | grep PASSWORD
grep PASSWORD compose.yaml
```

## Problemas, causa, solución y comprobación
* **Problema:** Al ejecutar `sudo docker compose config --quiet` tras redactar `compose.yaml`, la terminal mostró dos advertencias (`level=warning`) indicando que las variables `DB_ROOT_PASSWORD` y `DB_APP_PASSWORD` no estaban definidas y tomaban un valor vacío.
* **Causa:** El fichero `.env` aún no había sido creado en el directorio `~/butarque-portal` en el momento de realizar la primera validación de sintaxis.
* **Solución:** Crear el fichero `.env` en el mismo directorio con las dos contraseñas definidas y aplicar permisos restrictivos (`chmod 600 .env`) para que solo el propietario pueda leerlo.
* **Comprobación:** Ejecutar de nuevo `sudo docker compose config --quiet` (verificando que ya no muestra ningún aviso) y `sudo docker compose config | grep PASSWORD` (confirmando que las claves se cargan correctamente).
