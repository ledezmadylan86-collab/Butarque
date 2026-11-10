# Incidencia Soporte - Capa de Equipamientos e infraestructuras

## 1. Descripción de la Capa
Este despliegue corresponde a la capa de **Equipamientos e infraestructuras** dentro del proyecto **Mapa vivo de Butarque**, gestionando los puntos de interés específicos asignados al departamento de Soporte.

## 2. Requisitos Previos
Para levantar y desplegar este entorno en el servidor es necesario contar con:
- Servidor con **Ubuntu Server** (o entorno Linux compatible).
- **Docker** y **Docker Compose** instalados (`docker-compose-v2`).
- El paquete comprimido del proyecto (`mapa-vivo-butarque.zip`).

## 3. Pasos de Despliegue (Implantación)
1. Descomprimir el paquete del proyecto en el servidor:
   ```bash
   unzip mapa-vivo-butarque.zip
   cd mapa-vivo-butarque
