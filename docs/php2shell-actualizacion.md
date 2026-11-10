# PHP2Shell · Procedimiento de actualización

## Qué hace falta
- Un servidor Ubuntu (en mi caso, **Ubuntu 26.04.1 LTS "resolute"**) con acceso a Internet.
- Un usuario con permisos de `sudo`.
- Saber qué PHP tiene instalado el servidor:
```bash
  php -v
  dpkg -l | grep php
```
  En mi VM: **PHP 8.5.4** (paquetes `php8.5-cli`, `php8.5-common`, `libapache2-mod-php8.5`…), instalado desde los repositorios oficiales de Ubuntu.
- Que el servidor tenga activado el repositorio de seguridad (`resolute-security`), que es por donde llegan los parches.

## Cómo saber si un servidor está parcheado
```bash
apt policy php8.5-cli
```
Se comparan dos líneas:
- **Instalados**: la versión que tiene el servidor.
- **Candidato**: la última versión disponible en los repositorios.

Si las dos coinciden y la versión viene de `resolute-security` → **parcheado**.
Si *Instalados* es menor que *Candidato* → **falta el parche**.
Si sale `(ninguno)` o `php: command not found` → no tiene PHP, **no afectado**.

En mi VM salió:
```
php8.5-cli:
  Instalados: 8.5.4-0ubuntu1.5
  Candidato:  8.5.4-0ubuntu1.5
  *** 8.5.4-0ubuntu1.5 500
        500 http://security.ubuntu.com/ubuntu resolute-security/main amd64 Packages
      8.5.4-0ubuntu1 500
```
La versión original de la distribución es `8.5.4-0ubuntu1`; la instalada es `8.5.4-0ubuntu1.5`, la actualización de seguridad.

## Cómo se parchea
```bash
sudo apt update                        # refresca la lista de paquetes
apt list --upgradable | grep php       # ¿hay versiones nuevas de PHP?
sudo apt install --only-upgrade 'php8.5*' libapache2-mod-php8.5
```
Como el servidor usa PHP dentro de Apache (`libapache2-mod-php8.5`), después hay que reiniciarlo para que cargue la versión nueva:
```bash
sudo systemctl restart apache2
```
En mi VM, `apt list --upgradable | grep php` no devolvió nada: el servidor ya tenía la última versión de seguridad, así que no hizo falta instalar nada.

## Cómo se comprueba después
```bash
apt policy php8.5-cli     # Instalados = Candidato
php -v                    # PHP 8.5.4
apt list --upgradable | grep php   # no debe salir nada
```
Resultado en mi VM: `Instalados: 8.5.4-0ubuntu1.5`, igual que el candidato → **servidor parcheado**.

También comprobé que el Mapa vivo sigue funcionando:
```bash
curl http://localhost:8090/api/salud
# {"estado":"ok","puntos":21}
```
