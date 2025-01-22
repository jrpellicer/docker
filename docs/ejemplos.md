---
layout: page
title: Ejemplos de despliegue
nav_order: 12
---

# Ejemplos de despliegue
{: .no_toc }

<details open markdown="block">
  <summary>
    Tabla de contenidos
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>


## Servidor web

	mkdir www
	echo '<html><body><h1>Hola Mundo!</h1></body></html>' > www/index.html
	docker run -dit --name mi-apache -p 80:80 -v "$PWD"/www:/usr/local/apache2/htdocs/ httpd:latest

## Servidor web con archivo de configuración personalizado

En primer lugar obtenemos el archivo de configuración por defecto de un contenedor httpd:

	docker run --rm httpd:latest cat /usr/local/apache2/conf/httpd.conf > my-httpd.conf

A continuación lo modificamos, y una vez modificado creamos un contenedor con un DokcerFile copiando el fichero de configuración.

	FROM httpd:2.4
	COPY ./my-httpd.conf /usr/local/apache2/conf/httpd.conf

Creamos la imagen nueva con el nuevo archivo de configuración:

	docker build -t my-httpd .

Procedemos de manera similar al servidor web estándar pero con el nuevo nombre de imagen:

    mkdir www
	echo '<html><body><h1>Hola Mundo!</h1></html>' > www/index.html
	docker run -dit --name mi-apache -p 80:80 -v "$PWD"/www:/usr/local/apache2/htdocs/ my-httpd

## Servidor web con PHP

    mkdir www
	echo "<html><body><?php echo 'Versión actual de PHP: ' . phpversion(); ?></body></html>" > www/index.php
    docker run --name apache -d -p 80:80 -v "$PWD"/www:/var/www/html php:apache

## Servidor MySQL

    docker run --name mibbdd -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql

{: .warning }
Este contenedor de MySQL no acepta conexiones remotas, sólo del *localhost*. Por más que he investigado, no he logrado hacerlo funcionar (solamente por túnel SSH). Para conexiones remotas utilizo el contenedor de *MariaDB*.

## Servidor MySQL con Adminer (PHPmyAdmin)

    version: '3.1'

    services:

    db:
        image: mysql
        # NOTE: use of "mysql_native_password" is not recommended: https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html#upgrade-caching-sha2-password
        # (this is just an example, not intended to be a production configuration)
        command: --default-authentication-plugin=mysql_native_password
        restart: always
        environment:
        MYSQL_ROOT_PASSWORD: 123456

    adminer:
        image: adminer
        restart: always
        ports:
        - 8080:8080

Levantamos los contenedores:

    docker-compose up

 Una vez ejecutados dejamos unos minutos que se inicialicen los contenedores y accedemos a la dirección IP `https://localhost:8080`

## Servidor MariaDB

    docker run -d --name maria -p 3306:3306 -eMARIADB_ROOT_PASSWORD=123456 mariadb/server

## Servidor MariaDB con Adminer (PHPmyAdmin)

En lugar de hacerlo con *docker-compose* muestro cómo se haría con lanzamientos de contenedores independientes.

    docker run -d --name maria -p 3306:3306 -eMARIADB_ROOT_PASSWORD=123456 mariadb
    docker run -d --name adminer --link maria:db -p 8080:8080 adminer

## Servidor PostgresSQL

    docker run --name postgres -p 5432:5432 -e POSTGRES_PASSWORD=123456 -d postgres

Para administrar la BBDD desde la web podemos ejecutar *adminer* como en los ejemplos anteriores:

    docker run -d --name adminer --link postgres:db -p 8080:8080 adminer

## Servidor MongoDB

    docker run --name mongo -p 27017:27017 -d mongo:latest

## HTTPD, PHP y MySQL

Para integrar php y MySQL podemos levantar 2 contenedores con docker-compose a partir de los ficheros del repositorio de git hub:

    git clone https://github.com/kodetop/docker-php-mysql.git
    cd docker-php-mysql
    docker-compose up -d

Accedemos a `http://localhost/info.php`

En el fichero `.env` se pueden cambiar los parámteros relacionados con las versiones de PHP y MySQL, así como los usuarios y contraseñas de MySQL.

En el directorio `wwww` podemos crear un fichero `index.php` de comprobación de la conexión a la BBDD, con el siguiente contenido:

```php
<?php
//These are the defined authentication environment in the db service

// The MySQL service named in the docker-compose.yml.
$host = 'mysql';

// Database use name
$user = 'dbuser';

//database user password
$pass = 'dbpass';

// check the MySQL connection status
$conn = new mysqli($host, $user, $pass);
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
} else {
    echo "Connected to MySQL server successfully!";
}
?>
```

## Imagen de Ubuntu

    docker run -dti --name ubuntu ubuntu:latest

## Ubuntu 18.04 personalizado con SSH

Fichero **Dockerfile**:

    FROM ubuntu:18.04

    RUN apt-get update

    # Do not exclude man pages & other documentation
    RUN rm /etc/dpkg/dpkg.cfg.d/excludes

    # Reinstall all currently installed packages in order to get the man pages back
    RUN dpkg -l | grep ^ii | cut -d' ' -f3 | xargs apt-get install -y --reinstall && \
        rm -r /var/lib/apt/lists/*

    # Instalar y configurar manual en español y reconfigurar locales
    RUN apt-get update && apt-get install -y man manpages-es manpages-es-extra locales
    ENV LC_MESSAGES='es_ES.UTF-8'
    ENV LANGUAGE='es_ES.UTF-8'
    ENV LANG='es_ES.UTF-8'
    RUN export LC_MESSAGES=es_ES.UTF-8 && locale-gen es_ES.UTF-8

    # Instalar servidor SSH
    RUN apt-get install -y openssh-server && mkdir /var/run/sshd

    # Instalar aplicaciones sudo y nano
    RUN apt-get install -y sudo nano

    # Crear usuario alumno
    RUN useradd -m -G sudo -s /bin/bash alumno && echo "alumno:alumno" | chpasswd && echo "export LC_MESSAGES=es_ES.UTF-8" >> /home/alumno/.bashrc

    RUN apt-get clean && \
        rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

    EXPOSE 22
    CMD    ["/usr/sbin/sshd", "-D"]

Ejecutamos:

    docker build -t ubuntussh:18.04 .

## Samba

    mkdir samba
    docker run -dit --name samba -p 139:139 -p 445:445  \
      -e "USERID=1000" -e "GROUPID=1000" \
      -v "$PWD"/samba:/mount \
      dperson/samba -p -u "admin;qwe_123" -s "public;/mount;yes;no;no"

{: .note }
Las variables de entorno USERID y GROUPID hacen referencia al UID y GID del usuario de la máquina anfitrión que será propietario de la carpeta `./samba` (por ejemplo smbuser)

## NextCloud

    docker run --name nextcloud -d -v nextcloud:/var/www/html -p 8080:80 nextcloud

Accedemos a `hhtps://localhost/:8080` y acabamos de configurar.

Podemos optar pasar por variables de entorno las opciones de configuración, seleccionando por ejemplo SQLlite:

    docker run --name nextcloud -d -v nextcloud:/var/www/html -p 8080:80 \
      -eSQLITE_DATABASE=nextclouddb \
      -eNEXTCLOUD_ADMIN_USER=admin \
      -eNEXTCLOUD_ADMIN_PASSWORD=123456 \
      nextcloud

Si nos conectamos desde una IP remota, debemos pasar la variable de entorno `NEXTCLOUD_TRUSTED_DOMAINS` con la IP de la máquina servidor:

    docker run --name nextcloud -d -v nextcloud:/var/www/html -p 8080:80 \
      -eSQLITE_DATABASE=nextclouddb \
      -eNEXTCLOUD_ADMIN_USER=admin \
      -eNEXTCLOUD_ADMIN_PASSWORD=123456 \
      -eNEXTCLOUD_TRUSTED_DOMAINS=192.168.10.28 \
      nextcloud

## Portainer

    docker volume create portainer_data
    docker run -d -p 9000:9000 \
      -v /var/run/docker.sock:/var/run/docker.sock \
      -v portainer_data:/data \
      portainer/portainer

## NodeRED

    mkdir node-red
    docker run -d -p 1880:1880 --name nodered -v "$PWD"/node-red:/data nodered/node-red

## Moodle

    curl -sSL https://raw.githubusercontent.com/bitnami/containers/main/bitnami/moodle/docker-compose.yml > docker-compose.yml
    docker-compose up -d

Dejamos un tiempo para que arranquen los servicios y accedemos a `http://localhost/` y nos validamos con el usuario **user** y contraseña **bitnami**

Si deseamos cambiar el tamaño de los arcivos a subir (para restaurar algún curso, por ejemplo) hemos de modificar 2 parámetros del archivo de configuración **php.ini**

    docker exec -it moodle_moodle_1 /bin/bash
    apt update
    apt install nano -y
    nano /opt/bitnami/php/lib/php.ini	(modficar upload_max_filesize y post_max_size)
    exit
    docker restart moodle_moodle_1


## Active Directory en Samba

    docker run -d -t -i \
    --name samba \
    --hostname lovelace \
    --dns-search lovelace.naranjo.asir \
    --dns 192.168.0.10 \
    --dns 8.8.8.8 \
    --add-host lovelace.naranjo.asir:192.168.0.10 \
    --privileged \
    -p 192.168.0.10:53:53 \
    -p 192.168.0.10:53:53/udp \
    -p 192.168.0.10:88:88 \
    -p 192.168.0.10:88:88/udp \
    -p 192.168.0.10:135:135 \
    -p 192.168.0.10:137-138:137-138/udp \
    -p 192.168.0.10:139:139 \
    -p 192.168.0.10:389:389 \
    -p 192.168.0.10:389:389/udp \
    -p 192.168.0.10:445:445 \
    -p 192.168.0.10:464:464 \
    -p 192.168.0.10:464:464/udp \
    -p 192.168.0.10:636:636 \
    -p 192.168.0.10:1024-1044:1024-1044 \
    -p 192.168.0.10:3268-3269:3268-3269 \
    -v /etc/localtime:/etc/localtime:ro \
    -v /data/docker/containers/samba/data/:/var/lib/samba \
    -v /data/docker/containers/samba/config/samba:/etc/samba/external \
    -e "DOMAIN=NARANJO.ASIR" \
    -e "DOMAINPASS=qwe_123" \
    -e "DNSFORWARDER=8.8.8.8" \
    -e "HOSTIP=192.168.0.10" \
    -e "INSECURELDAP=true" \
    nowsci/samba-domain


## Python

Para ejecutar un script de python almacenado en el directorio actual de nombre *your-daemon-or-script.py*

    docker run -it --rm --name my-running-script \
      -v "$PWD":/usr/src/myapp \
      -w /usr/src/myapp \
      python:3 python your-daemon-or-script.py

## Jupyter Notebook

Podemos elegir entre distintas imágenes que incorporan una serie de paquetes preinstalados. Las imágenes a elegir son:

- jupyter/base-notebook
- jupyter/minimal-notebook
- jupyter/r-notebook
- jupyter/scipy-notebook
- jupyter/tensorflow-notebook
- jupyter/datascience-notebook
- jupyter/pyspark-notebook
- jupyter/all-spark-notebook

Para ejecutar tecleamos:

    mkdir jupyter
    docker run -it --name jupyter -p 8888:8888 -v "$PWD"/jupyter:/home/jovyan jupyter/base-notebook

Accedemeos a la dirección que nos muestra con el token corrspondiente:
    
    http://127.0.0.1:8888/lab?token=7bdd1b4497fc9b833c724004e837f15b823e9c713afc6ee9


{: .note }
Para instalar un paquete, podemos abrir una terminal dentro de la interfaz web y ejecutar el comando pip correspondiente: `pip install pymongo`

## Just-The-Docs Theme con Jekyll

Para crear un sitio local con el tema Just-the-Docs de Jekyll sin necesidad de instalar Jekyll.

    git clone https://github.com/just-the-docs/just-the-docs.git
    cd just-the-docs
    docker-compose up

{: .warning }
Durante la instalación de la gema puede dar un error de versiones. Se soluciona bajando la versión dentro del **Dockerfile** y modificando la penúltima línea (`RUN gem install bundler && bundle install` por `RUN gem install bundler:2.3.26 && bundle install`)

    FROM ruby:2.7

    ENV LC_ALL C.UTF-8
    ENV LANG en_US.UTF-8
    ENV LANGUAGE en_US.UTF-8

    WORKDIR /usr/src/app

## Just-The-Docs Theme con Jekyll (Nueva versión)

Para crear un sitio local con el tema Just-the-Docs de Jekyll sin necesidad de instalar Jekyll.

    git clone https://github.com/just-the-docs/just-the-docs.git
    cd just-the-docs
    docker run -p 4000:4000 -v $(pwd):/site bretfisher/jekyll-serve

## MkDocs con Jekyll

Para crear un sitio local con el tema MkDocs de Jekyll sin necesidad de instalar Jekyll.

EJEMPLO CON UN REPOSITORIO LLAMADO awsasir:

    git clone https://github.com/jrpellicer/awsasir.git
    cd awsasir
    docker run --rm -it -p 8000:8000 -v ${PWD}:/docs squidfunk/mkdocs-material
