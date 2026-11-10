# Incidencia PHP2Shell

## 1. Descripción de la incidencia

PHP2Shell es el nombre utilizado para identificar dos vulnerabilidades críticas de seguridad que afectan al intérprete PHP: CVE-2026-109618 y CVE-2026-109619.

Estas vulnerabilidades permiten que un atacante remoto, sin necesidad de autenticarse, pueda ejecutar código en el servidor que utiliza una aplicación PHP vulnerable.

El problema supone un riesgo importante para los servidores web, ya que un atacante podría ejecutar comandos con los permisos del usuario que ejecuta PHP.

## 2. Causa del problema

La incidencia se debe a dos fallos de seguridad en PHP que permiten la ejecución remota de código (RCE).

El problema afecta a las aplicaciones que utilizan versiones vulnerables del intérprete PHP. Por este motivo, era necesario actualizar los paquetes afectados para evitar posibles accesos y ejecuciones de código no autorizadas.

Las vulnerabilidades están identificadas como CVE-2026-109618 y CVE-2026-109619.

## 3. Solución aplicada

Para solucionar la incidencia, Ubuntu publicó actualizaciones de seguridad que corrigen las vulnerabilidades PHP2Shell.

En nuestro entorno utilizamos Ubuntu 24.04 con PHP 8.3. La versión corregida del paquete `php8.3-cli` es:

`8.3.6-0ubuntu0.24.04.13`

Para actualizar el paquete, los comandos utilizados como procedimiento de corrección son:

```bash
sudo apt update
sudo apt install --only-upgrade php8.3-cli
```

De esta manera se instala la actualización de seguridad disponible sin necesidad de cambiar de versión principal de PHP.

## 4. Comprobación después de la actualización

Para verificar que nuestra máquina virtual cumple el requisito de seguridad, ejecutamos:

```bash
apt policy php8.3-cli
```

El resultado obtenido en nuestra máquina virtual fue:

```text
Installed: 8.3.6-0ubuntu0.24.04.13
Candidate: 8.3.6-0ubuntu0.24.04.13
```

La versión instalada coincide con la versión corregida publicada para Ubuntu 24.04.

Por tanto, podemos confirmar que el paquete `php8.3-cli` de nuestra máquina virtual cumple el requisito de actualización contra PHP2Shell.

**Resultado final: VM parcheada correctamente.**

## 5. Referencias

- Ubuntu: [PHP2Shell – Actualizaciones de seguridad disponibles](https://ubuntu.com/blog/php2shell-vulnerability-fixes-available)
- Ubuntu Security Notices: USN-8948-1.
- Identificadores de las vulnerabilidades: CVE-2026-109618 y CVE-2026-109619.
