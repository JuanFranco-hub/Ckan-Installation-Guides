# Cómo instalar CKAN 2.9 desde Source en Ubuntu 18.04 y 20.04

### 1. Actualice los paquetes de Ubuntu:
```sh
sudo apt-get update
```

### 2. Instalar y configurar PostgreSQL:
```sh
# Instalamos PostgreSQL con el siguiente comando:
sudo apt-get install -y postgresql

# Luego, creamos un usuario de PostgreSQL para CKAN y Datastore con el siguiente comando
# Introduzca ***{contraseña1}***
sudo -u postgres createuser -S -D -R -P ckan_default

# Creamos las bases de datos de CKAN y Datastore con los siguientes comandos:
sudo -u postgres createdb -O ckan_default ckan_default -E utf-8
sudo -u postgres createdb -O ckan_default datastore_default -E utf-8

# Creamos otro usuario de PostgreSQL para leer la base de datos de Datastore
# Introduce ***{contraseña2}***
sudo -u postgres createuser -S -D -R -P -l datastore_default

#Por último, verificamos que las bases de datos ckan_default y datastore_default estén en la lista de bases de datos con el siguiente comando:
sudo -u postgres psql -l
```

### 3. Instalación y configuración de Apache Solr 6.5.1 en Ubuntu:
```sh
# Instalar openjdk-8-jdk
sudo apt-get install openjdk-8-jdk

# Cambiar a usar openjdk-8-jdk
sudo update-alternatives --set java /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java

# Descargar solr-6.5.1
wget http://archive.apache.org/dist/lucene/solr/6.5.1/solr-6.5.1.tgz

# Descomprimir solr-6.5.1 y ejecutar script de instalación
tar xzf solr-6.5.1.tgz solr-6.5.1/bin/install_solr_service.sh --strip-components=2
sudo bash ./install_solr_service.sh solr-6.5.1.tgz

# Acceder a solr como usuario solr
sudo su solr

# Crear colección ckan en solr
cd /opt/solr/bin
./solr create -c ckan

# Configurar solrconfig.xml
cd /var/solr/data/ckan/conf
mv solrconfig.xml solrconfig.xml.bak
wget https://raw.githubusercontent.com/ckan/ckan/master/contrib/docker/solr/solrconfig.xml
rm managed-schema
ln -s /usr/lib/ckan/default/src/ckan/ckan/config/solr/schema.xml schema.xml
exit

# Reiniciar solr
sudo service solr restart

## Para solucionar la vulnerabilidad de solr 
### https://issues.apache.org/jira/browse/SOLR-13669

# Habilitar firewall ufw
sudo ufw enable

# Configurar firewall para permitir sólo ssh, http y https
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https

```

### 4. Instalación de dependencias de Python y paquetes de Ubuntu requeridos por CKAN:
Verifique la versión de Ubuntu usando el comando 
```sh
cat /etc/os-release
```
- Para Ubuntu 20.04:
```sh
sudo apt-get install python-dev libpq-dev redis-server git build-essential

sudo add-apt-repository universe

sudo apt install python2

sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 1

sudo update-alternatives --config python

curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py

sudo python2 get-pip.py

sudo apt install virtualenv

```
- **Para Ubuntu 18.04**:
```sh
sudo apt-get install python-dev libpq-dev redis-server python-pip python-virtualenv git-core
```

### 5. Configurar python2 y pip2:
```sh
#Compruebe la versión de python y establecala en 2.7
python -V
# Python 2.7.x

#Verifique la versión de pip y configúrelo para que se ejecute desde ... (python 2.7)
pip -V
# pip x.x.x from /usr/local/lib/python2.7/dist-packages/pip (python 2.7)
```

### 6. Establezca la ruta de CKAN:
```sh

# Preparar ruta de CKAN
sudo mkdir -p /usr/lib/ckan/predeterminado
sudo chown -R $(whoami) /usr/lib/ckan/predeterminado 

# Preparar ubicación de almacenamiento
sudo mkdir -p /var/lib/ckan/predeterminado
sudo chown -R $(whoami) /var/lib/ckan
sudo chmod -R 775 /var/lib/ckan

```

### 7. Instalar CKAN:
```sh
# Crea un entorno virtual con python2 en la ruta "/usr/lib/ckan/default".
virtualenv --python=python2 /usr/lib/ckan/default

# Activa el entorno virtual.
. /usr/lib/ckan/default/bin/activate

# Nos desplazamos al directorio "/usr/lib/ckan/default".
cd /usr/lib/ckan/default

# Actualiza la versión de pip.
pip install --upgrade pip

# Instala una versión específica de setuptools (44.1.0).
pip install setuptools==44.1.0

# Instala una versión específica de CKAN a partir de su repositorio en GitHub.
pip install -e 'git+https://github.com/ckan/ckan.git@ckan-2.9.5#egg=ckan[requirements-py2]'

# Desactiva el entorno virtual.
deactivate
```

### 8. Configure y cree una base de datos para CKAN.
#### 8.1 Establecer who.ini:
```sh
sudo mkdir -p /etc/ckan/default

sudo ln -s /usr/lib/ckan/default/src/ckan/who.ini /etc/ckan/default/who.ini

sudo chown -R $(whoami) /etc/ckan/
```
#### 8.2 Edite el archivo de configuración y cree la base de datos CKAN de la siguiente manera:
```sh
# Activa el entorno virtual para CKAN
. /usr/lib/ckan/default/bin/activate

# Genera un archivo de configuración 
ckan generate config /etc/ckan/default/ckan.ini

# Abre el archivo en un editor de texto
sudo vi /etc/ckan/default/ckan.ini
    - Editar {contraseña1} (del paso 2 de configuración) de sqlalchemy.url
        > sqlalchemy.url = postgresql://ckan_default:{contraseña1}@localhost/ckan_default
    - Habilitar y editar {contraseña1} (del paso 2 de configuración) de ckan.datastore.write_url
        > ckan.datastore.write_url = postgresql://ckan_default:{contraseña1}@localhost/datastore_default
    - Habilitar y editar {contraseña2} (del paso 2 de configuración) de ckan.datastore.read_url
        > ckan.datastore.read_url = postgresql://datastore_default:{contraseña2}@localhost/datastore_default
    - Definir ckan.site_url
        > ckan.site_url = http://localhost:5000
    - Habilitar y editar solr_url
        > solr_url = http://127.0.0.1:8983/solr/ckan
    - Habilitar ckan.redis.url
        > ckan.redis.url = redis://localhost:6379/0
    - Editar ckan.plugins 
        > ckan.plugins = stats text_view image_view recline_view resource_proxy datastore webpage_view
    - Editar ckan.views.default_views 
        > ckan.views.default_views = image_view text_view recline_view webpage_view
    - Habilitar y editar ckan.storage_path
        > ckan.storage_path = /var/lib/ckan/default

# Reiniciar solr
sudo service solr restart

# Inicializar Base de Datos
ckan -c /etc/ckan/default/ckan.ini db init

# Desactiva el entorno virtual.
deactivate

```

### 9. Cree CKAN SysAdmin y asigne permisos de DataStore:

```sh
# Activa el entorno virtual para CKAN
. /usr/lib/ckan/default/bin/activate

# Crear {nombre_usuario}
ckan -c /etc/ckan/default/ckan.ini sysadmin add {nombre_usuario}

# Establecer permisos a DataStore
ckan -c /etc/ckan/default/ckan.ini datastore set-permissions | sudo -u postgres psql --set ON_ERROR_STOP=1

```

###  10. Cómo configurar la producción de CKAN 

#### 10.1 Instale y configure uwsgi.
```sh
# Activa el entorno virtual para CKAN
. /usr/lib/ckan/default/bin/activate

# Instalar la dependencia UWSGI para CKAN
pip install uwsgi

# Desactiva el entorno virtual.
deactivate

# Copiar el archivo de configuración de UWSGI para CKAN al directorio de configuración de CKAN
sudo cp /usr/lib/ckan/default/src/ckan/ckan-uwsgi.ini /etc/ckan/default/

# Copiar el archivo WSGI principal para CKAN al directorio de configuración de CKAN
sudo cp /usr/lib/ckan/default/src/ckan/wsgi.py /etc/ckan/default/
```
#### 10.2 Instale y configure supervisor para ejecutar uwsgi.
```sh

# Instala supervisor
sudo apt-get install supervisor

# Crea una carpeta de registro para CKAN
sudo mkdir -p /var/log/ckan

# Crea una configuración de supervisor para ckan-uwsgi
sudo vi /etc/supervisor/conf.d/ckan-uwsgi.conf
```
Agrega el siguiente comando
```sh
[program:ckan-uwsgi]

command=/usr/lib/ckan/default/bin/uwsgi -i /etc/ckan/default/ckan-uwsgi.ini

; Start just a single worker. Increase this number if you have many or
; particularly long running background jobs.
numprocs=1
process_name=%(program_name)s-%(process_num)02d

; Log files - change this to point to the existing CKAN log files
stdout_logfile=/var/log/ckan/ckan-uwsgi.stdout.log
stderr_logfile=/var/log/ckan/ckan-uwsgi.stderr.log

; Make sure that the worker is started on system start and automatically
; restarted if it crashes unexpectedly.
autostart=true
autorestart=true

; Number of seconds the process has to run before it is considered to have
; started successfully.
startsecs=10

; Need to wait for currently executing tasks to finish at shutdown.
; Increase this if you have very long running tasks.
stopwaitsecs = 600

; Required for uWSGI as it does not obey SIGTERM.
stopsignal=QUIT
```
#### 10.3 Instalar y configurar nginx.
```sh
# Instala nginx
sudo apt-get install nginx

# Abre el archivo de configuración en un editor de texto.
sudo vi /etc/nginx/sites-available/ckan
```
Agrega el siguiente comando
```sh
proxy_cache_path /var/cache/nginx/proxycache levels=1:2 keys_zone=cache:30m max_size=250m;
proxy_temp_path /tmp/nginx_proxy 1 2;

server {
    client_max_body_size 100M;
    server_tokens off;
    location / {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host $host;
        proxy_cache cache;
        proxy_cache_bypass $cookie_auth_tkt;
        proxy_no_cache $cookie_auth_tkt;
        proxy_cache_valid 30m;
        proxy_cache_key $host$scheme$proxy_host$request_uri;
        # In emergency comment out line to force caching
        # proxy_ignore_headers X-Accel-Expires Expires Cache-Control;
    }

}
```
#### 10.4 Empieza a usar CKAN
```sh
# Eliminar el archivo default de nginx
sudo rm -r /etc/nginx/sites-enabled/default

# Habilitar CKAN para nginx
sudo ln -s /etc/nginx/sites-available/ckan /etc/nginx/sites-enabled/ckan

# Preparar proxycache
sudo mkdir -p /var/cache/nginx/proxycache && sudo chown www-data /var/cache/nginx/proxycache

# Cambiar permisos necesarios
sudo chown -R www-data:www-data /var/lib/ckan
sudo chown -R www-data:www-data /usr/lib/ckan/default/src/ckan/ckan/public
sudo chown -R www-data /tmp/default/

# Modificar configuración de CKAN
sudo vi /etc/ckan/default/ckan.ini
    - Especificar la dirección IP en ckan.site_url
        > ckan.site_url = http://{dirección IP}


# Reiniciar servicios
sudo supervisorctl reload
sudo service nginx restart

```

### 11. Pruebe el sitio web a través de http://{dirección IP}

### 12. Cronjob para el seguimiento de vistas de página:

Crear configuración de trabajos en segundo plano
```sh
# Copiar archivo de configuración de worker de CKAN a supervisor
sudo cp /usr/lib/ckan/default/src/ckan/ckan/config/supervisor-ckan-worker.conf /etc/supervisor/conf.d/ckan-worker.conf
```

Reinicia supervisor
```sh
# Reiniciar servicio
sudo supervisorctl reload
```

```sh

# Editar tareas programadas en cron
crontab -e
```
Agrega el siguiente comando
    
    # Agregar tarea para actualizar seguimiento y reconstruir índice de búsqueda de CKAN cada hora
    @hourly /usr/lib/ckan/default/bin/ckan -c /etc/ckan/default/ckan.ini tracking update && /usr/lib/ckan/default/bin/ckan -c /etc/ckan/default/ckan.ini search-index rebuild -r

### 13. Instalar ckanext-xloader:
```sh
# Activa el entorno virtual de CKAN
source /usr/lib/ckan/default/bin/activate

# Cambia al directorio por defecto de CKAN
cd /usr/lib/ckan/default

# Instala la extensión de CKAN desde el repositorio de GitLab
pip install -e 'git+https://gitlab.nectec.or.th/opend/ckanext-xloader.git#egg=ckanext-xloader'

# Instala los paquetes necesarios para la extensión
pip install -r src/ckanext-xloader/requirements.txt

# Actualiza la biblioteca `requests` a su última versión con correcciones de seguridad
pip install -U requests[security]
```
Edite el archivo de configuración de CKAN de la siguiente manera:
```sh
sudo vi /etc/ckan/default/ckan.ini
```
```sh
    - Agregue la configuración después de la línea [app:main]
        > ckanext.xloader.just_load_with_messytables = false
        > ckanext.xloader.ssl_verify = false
    - ckan.plugins (cambie de datapusher a xloader)
        > ckan.plugins = ... xloader ...
    - ckanext.xloader.jobs_db.uri (agregue esta configuración después de sqlalchemy.url y que tenga el mismo valor)
        > ckanext.xloader.jobs_db.uri = postgresql://ckan_default:{contraseña1}@localhost/ckan_default
```
```sh
# Reiniciar supervisor
sudo supervisorctl reload
```

Para que el xloader envíe todo automáticamente al DataStore todos los días, configure el cronjob de la siguiente manera:
```sh
crontab -e
```
Agrega el siguiente comando

    @daily /usr/lib/ckan/default/bin/paster --plugin=ckanext-xloader xloader submit all -c /etc/ckan/default/ckan.ini