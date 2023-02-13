---
layout: page
title: Docker Compose
nav_order: 11
---

# Docker Compose
{: .no_toc }

Docker Compose es una herramienta no incluida dentro de la instalación de Docker Engine que nos va a permitir definir y ejecutar contenedores basados en una plantilla definida en un fichero YAML. Es una aplicación para simplificar la tarea de lanzar múltiples contenedores con una configuración específica y enlazarlos entre sí.

<details open markdown="block">
  <summary>
    Tabla de contenidos
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

## Instalación

Si no tenemos instalado Docker Compose podemos optar por varios métodos de instalación:
- Mediante un paquete deb si nuestra distribución los soporta: 

    `sudo apt install docker-compose`

- Descargando el binario desde github [https://github.com/docker/compose/releases](https://github.com/docker/compose/releases):

    `sudo curl -L "https://github.com/docker/compose/releases/download/1.29.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

    `sudo chmod +x /usr/local/bin/docker-compose`

- Mediante un paquete PIP de Python: 

    `sudo pip3 install docker-compose`

- Instalando Docker Desktop, que ya incluye Docker Compose y Docker Engine.

Una vez instalado Docker Compose, podemos comprobar si la instalación es correcta con:

    docker-compose --version

## Formato del fichero *docker-compose.yml*

La plantilla del contenedor o contenedores a ejecutar se define en un fichero YAML de nombre **docker-compose.yml** que contiene varias palabras clave el cual se creará en un directorio con los posibles archivos necesarios para la creación del contenedor o de la imagen si fuera necesaria mediante un *Dockerfile*.

El fichero **docker-compose.yml** debe respetar la sintaxis del formato [YAML](https://es.wikipedia.org/wiki/YAML), y la extensión del archivo puede ser *yml* o *yaml*.

En YAML, se usa la sangría al estilo Python para indicar la incorporación de un elemento de código dentro de otro. No se admiten los caracteres de tabulación, así que se usan los espacios en blanco.

Las listas incluyen valores en un orden específico y pueden contener cualquier cantidad de elementos. Las secuencias de las listas empiezan con un guion (-) y un espacio, y se utiliza la sangría para separarlas del elemento principal.

```yaml
version: "3.9"

services:
  db:
     image: mysql:5.7
     volumes:
        - db_data:/var/lib/mysql
     restart: always
     environment:
        MYSQL_ROOT_PASSWORD: somewordpress
        MYSQL_DATABASE: wordpress
        MYSQL_USER: wordpress
        MYSQL_PASSWORD: wordpress
     
  wordpress:
     depends_on:
        - db
     image: wordpress:latest
     ports:
        - "8000:80"
     restart: always
     environment:
        WORDPRESS_DB_HOST: db:3306
        WORDPRESS_DB_USER: wordpress
        WORDPRESS_DB_PASSWORD: wordpress
        WORDPRESS_DB_NAME: wordpress

volumes:
   db_data:
```

## Instrucciones

Las instrucciones más comunes que se definen en la plantilla son las siguientes:

### version:


### build:

### dockerfile:

### args:

### command:

### entrypoint:

### container_name

### depends_on:

### environment:

### expose:

### image:

### networks:

### ports:

### volumes:
   
   docker-compose up
   docker-compose up -d

   docker-compose stop
   docker-compose down