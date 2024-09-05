# Cómo instalar CKAN usando Docker Compose

Para instalar CKAN con Docker Compose, se recomienda instalar docker y docker-compose primero con las siguientes versiones:
- docker >= 19
- docker-compose >= versión 1.13

Cómo verificar las versiones de docker y docker-compose:
```sh
docker -v
#Versión de Docker 19.03.13, construcción 4484c46d9d

docker-compose -v
#versión de docker-compose 1.26.2, construcción desconocida
```
## [Preparación] Instalación de docker
```sh
sudo apt-get update

curl https://get.docker.com | sh
#Esperar hasta que la instalación esté completa
```
Dar permiso de uso al usuario:
```sh
sudo usermod -aG docker `whoami`

newgrp docker
```
Verificar la versión de docker:
```sh
docker -v
```

## [Preparación] Instalación de docker-compose
```sh
sudo apt install docker-compose
```
Verificar la versión de docker-compose:
```sh
docker-compose -v
```

## Instalación de CKAN Docker y Extensiones
### 1. Descargar ckan-docker-thai-gdc
```sh
git clone https://gitlab.nectec.or.th/opend/ckan-docker-thai-gdc.git ~/ckan-docker
```

### 2. Crear un archivo .env a partir del archivo .env.template proporcionado
```sh
cd ~/ckan-docker

cp .env.template .env
```

### 3. Editar el archivo .env
```sh
vi .env
    - Establecer la contraseña para la base de datos de CKAN
        > POSTGRES_PASSWORD={contraseña_ckan}
    - Establecer la contraseña para Datastore
        > DATASTORE_READONLY_PASSWORD={contraseña_datastore}
    - Establecer el nombre del host para la base de datos Postgres
        > POSTGRES_HOST=db
    - Establecer la versión de CKAN (cambiar a 2.9)
        > CKAN_VERSION=2.9
    - Número de proyecto para el contenedor (por defecto)
        > PROJECT_NUMBER=1
    - Establecer el puerto para Nginx (cambiar a 80)
        > NGINX_PORT=80
    - Establecer el puerto para Datapusher
        > DATAPUSHER_PORT=8800
    - Establecer la URL para el sitio web (cambiar a IP o Dominio)
        > DEFAULT_URL=http://{IP o Dominio}
    - Establecer el ID del sitio CKAN (por defecto)
        > CKAN_SITE_ID=default
    - Establecer el puerto de CKAN
        > CKAN_PORT=5000
    - Establecer detalles del SysAdmin del sistema
        > CKAN_SYSADMIN_NAME={nombre_administrador}
        > CKAN_SYSADMIN_PASSWORD={contraseña_administrador}
        > CKAN_SYSADMIN_EMAIL={email_administrador}
    - URL para conectar con solr
        > CKAN_SOLR_URL=http://solr:8983/solr/ckan
    - URL para conectar con redis
        > CKAN_REDIS_URL=redis://redis:6379/0
    - Ruta para el almacenamiento de CKAN
        > CKAN__STORAGE_PATH=/var/lib/ckan
    - Todos los plugins habilitados
        > CKAN__PLUGINS=envvars discovery search_suggestions thai_gdc stats opendstats image_view text_view recline_view resource_proxy webpage_view datastore xloader scheming_datasets pdf_view hierarchy_display hierarchy_form dcat dcat_json_interface structured_data
    - Vistas predeterminadas
        > CKAN__VIEWS__DEFAULT_VIEWS=image_view text_view recline_view webpage_view pdf_view
```

### 4. Iniciar CKAN con docker-compose
```sh
docker-compose up -d --build

# Verificar el funcionamiento de docker-compose en ejecución, después espera aproximadamente 15 segundos
docker ps
```

### 5. Probar acceder al sitio web a través de http://{Dominio/IP}

## [Adicional] Cómo detener y limpiar los datos de CKAN docker
```sh
# Comando para detener y eliminar contenedores docker en docker compose
docker-compose down
# Comando para eliminar todos los volúmenes no utilizados
docker volume prune
# Comando para eliminar imágenes docker no utilizadas
docker system prune
```
### Cómo actualizar las imágenes docker cuando se actualiza la extensión thai gdc
```
# Actualizar la

 imagen docker
docker pull thepaeth/ckan-thai_gdc:ckan-2.9-xloader

# Ir al directorio donde clonamos el docker de thai gdc
cd ~/ckan-docker

# Asegurarse de que el archivo docker-compose.yml esté presente
ls -la | grep docker-composer

# Editar CKAN__PLUGINS en el archivo .env con la siguiente línea
CKAN__PLUGINS=envvars discovery search_suggestions thai_gdc stats opendstats image_view text_view recline_view resource_proxy webpage_view datastore xloader scheming_datasets pdf_view hierarchy_display hierarchy_form dcat dcat_json_interface structured_data

# Ejecutar docker compose para actualizar la imagen del contenedor
docker-compose up -d

```
