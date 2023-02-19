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

## Acciones básicas

Las opciones más comunes que se pueden utilizar con el comando docker-compose son:

- **up**: Esta opción se utiliza para crear y/o iniciar los contenedores definidos en el archivo *docker-compose.yml*. Si los contenedores ya existen, esta opción los iniciará, a menos que se especifique lo contrario con la opción `--no-recreate`.

- **stop**: Esta opción se utiliza para detener los contenedores definidos en el archivo *docker-compose.yml*, sin eliminarlos. Los contenedores se pueden reiniciar posteriormente utilizando el comando `docker-compose start`.

- **start**: Se utiliza para iniciar los contenedores definidos en el archivo *docker-compose.yml*, si ya han sido creados previamente.

- **down**: La utilizamos para detener y eliminar los contenedores, redes y volúmenes definidos en el archivo *docker-compose.yml*. Esta opción también elimina cualquier recurso creado con la opción `up`.

```bash
docker-compose up --no-recreate
```

{: .note }
También es posible utilizar el parámetro `-d` para levantar los servicios en segundo plano: `docker-compose up -d`

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

## Instrucciones de *docker-compose.yml*

Las instrucciones más comunes que se definen en la plantilla son las siguientes:

### version:

Indica la versión de formato de docker-compose.yaml que se está utilizando.

```yaml
version: '3'
```

### services

La etiqueta services es una sección fundamental del archivo docker-compose.yml. En esta sección se definen los servicios que componen la aplicación, donde cada servicio representa una parte o componente de la aplicación.

Cada servicio en la sección services se define mediante un bloque con un nombre y un conjunto de opciones de configuración, tales como la imagen de Docker a utilizar, las variables de entorno, los volúmenes montados, las dependencias, etc.

```yaml
version: '3'
services:
  db:
    image: postgres
    volumes:
      - db-data:/var/lib/postgresql/data
```

### image:

Se utiliza para indicar la imagen de Docker que se utilizará para construir un contenedor de Docker.

```yaml
services:
  web:
    image: nginx:latest
```

{: .note }
También es posible utilizar una imagen personalizada en lugar de una imagen preconstruida. Para hacer esto, en lugar de especificar una imagen en la etiqueta image, se puede usar la etiqueta `build` para indicar la ubicación del Dockerfile que se utilizará para construir la imagen personalizada.

### build:

Define la ruta de acceso a un directorio que contiene un archivo Dockerfile para crear una imagen de Docker personalizada, en caso de no utilizar la equiqueta `image`

```yaml
services:
  web:
    build: .
```

### dockerfile:

Especifica un nombre de fichero alternativo al por defecto *Dokerfile*

### command:

Define un comando personalizado, distinto al definido en el *Dockerfile*, que se debe ejecutar cuando se inicia un contenedor.

```yaml
command: ["/usr/sbin/apache2", "-D", "FOREGROUND"]
```

### entrypoint:

Define un script o comando, distinto al definido en el *Dockerfile*, que se debe ejecutar automáticamente cuando se inicia un contenedor.

```yaml
entrypoint: /usr/bin/comando.sh
```

### depends_on:

Se utiliza para definir las dependencias entre servicios en el entorno Docker Compose. Con `depends_on`, se puede especificar que un servicio depende de otro servicio y que este último debe iniciarse primero antes de que el otro pueda iniciarse.

{: .warning }
Es importante tener en cuenta que solo indica el orden de inicio y no garantiza que un servicio dependiente esté listo antes de que se inicie un servicio que depende de él.

```yaml
version: '3'
services:
  database:
    image: postgres
  web:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - database
```

### environment:

Define las variables de entorno que se deben establecer en los contenedores.

```yaml
environment:
   WORDPRESS_DB_HOST: db:3306
   WORDPRESS_DB_USER: wordpress
   WORDPRESS_DB_PASSWORD: wordpress
   WORDPRESS_DB_NAME: wordpress
```

### expose:

Se utiliza para indicar qué puertos expone un contenedor.

{: .warning }
Cuando se utiliza esta etiqueta, se indica a Docker que el contenedor expone ciertos puertos, pero no se indica cómo se deben mapear estos puertos en el host. Esto significa que los puertos expuestos a través de expose solo son accesibles desde otros contenedores en la misma red de Docker. Para acceder a este puerto desde el host, es necesario utilizar la etiqueta `ports` en el archivo *docker-compose.yaml*.

### ports:

Define cómo se deben mapear los puertos de los contenedores a los puertos del host.

```yaml
ports:
   - "8000:8000"
```

### networks:

Sirve para definir las redes personalizadas que se utilizarán en la aplicación que se ejecutará en Docker Compose.

Al definir una red personalizada, se puede controlar cómo se comunican los contenedores que se ejecutan en la misma red. Por defecto, los contenedores que se ejecutan en Docker Compose están en la misma red, lo que significa que pueden comunicarse entre sí a través de la dirección IP de contenedor o el nombre de contenedor.

La etiqueta `driver` especifica el controlador de red que se utilizará para la red personalizada.

Cada servicio puede conectarse a una o más redes personalizadas utilizando la etiqueta `networks`. Por ejemplo, el siguiente archivo *docker-compose.yml* define dos servicios, *service1* y *service2*, y los conecta a la red personalizada *my-network*:

```yaml
version: '3'
services:
  service1:
    ...
    networks:
      - my-network
  service2:
    ...
    networks:
      - my-network
networks:
  my-network:
    driver: bridge
```

### volumes:

Define los volúmenes de datos que se deben crear para almacenar información que persiste fuera de los contenedores.

```yaml
version: '3'
services:
  service1:
    ...
    volumes:
      - /ruta/local:/ruta/del/contenedor
      - myvolume:/ruta2/del/contenedor
```

{: .note }
Dos servicios distintos pueden compartir un mismo volumen.
