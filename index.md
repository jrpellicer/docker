# Instalación de Docker
Instalación

	sudo su
	curl –fsSL https://get.docker.com/ | sh

Comprobar información de funcionamiento

	docker info

Añadir usuario al grupo *docker*

	usermod usuario -aG docker

# Imágenes
Visualizar imágenes descargadas

	docker images
  
Buscar imágenes en el hub

	docker search --limit 5 httpd

Descargar imagen

	docker pull httpd
  
Descargar imagen indicando tag

	docker pull mysql:5.7
  
Borrar imagen

	docker rmi mysql:5.7

# Contenedores
## Creación básica y puesta en marcha
Creación básica de un contenedor

	docker create httpd

Visualización de contenedores corriendo

	docker ps
  
Puesta en marcha de un contenedor creado y no arrancado

	docker start 7c5
  
## Inspección de contenedores
Inspección de contenedores creados

	docker inspect

## Parada y eliminación
Parada de un contenedor arrancado

	docker stop 7c5

Visualización de contendores corriendo y parados

	docker ps -a

Eliminación de un contenedor parado

	docker rm 7c5

## Ejecución sin creación ni descarga de imágenes
Podemos correr un contenedor directamente sin necesidad de hacer un *pull*, *create* y *start*

	docker run mysql

## Visualización logs
Visualización del log y depuración de posibles errores en un contendor

	docker logs c31

## Opciones de ejecucion y creación
Podemos correr un contenedor sin necesidad de indicar más parámetros que la imagen

	docker run httpd

Podemos indicar un tag de la imagen

	docker run httpd:2.4.51-alpine

Ejecución de un contenedor en segundo plano

	docker run -d httpd

Ejecución de un contenedor indicando el nombre. Si utilizamos esta opción, podemos después parar, reiniciar, eliminar, etc el contenedor referenciándolo por su nombre en lugar de su ID

	docker run -d --name web httpd

Ejecución de un contenedor con terminal interactiva

	docker run -dti --name web httpd

Ejecución de un contenedor con mapeo de puertos

	docker run -dti --name web -p 80:80 httpd

Ejecución de un contenedor con mapeo de directorios (bind mount)

	docker run -dti --name web -p 80:80 -v "$PWD":/usr/local/apache2/htdocs/ httpd
	docker run -dti --name web -p 80:80 -v /home/usuario/directorio:/usr/local/apache2/htdocs/ httpd

Ejecución de un contenedor pasando una variable de entorno

	docker run --name mibbdd -e MYSQL_ROOT_PASSWORD=123456 -d mysql

## Ejecución de comandos al lanzar el contenedor
Podemos ejecutar un comando dentro del contenedor al lanzarse. El contenedor se para una vez ejecutado el comando

	docker run debian cat /etc/debian_version

Ejecución de un comando con borrado del contenedor una vez finalizada la ejecución del comando

	docker run --rm debian cat /etc/debian_version

Ejecución de un comando con redireccionamiento de salida a un fichero de la máquina anfitrión

	docker run --rm debian cat /etc/debian_version > version.txt

## Ejecución de comandos en contenedores corriendo
Podemos ejecutar un comando dentro de un contenedor que esté corriendo en segundo plano con una terminal interactiva (-dti).
En primer lugar lanzamos el contenedor:
	
	docker run --name debian -dti debian

Una vez corriendo el contenedor podemos ejecutar comandos dentro mediante la orden docker exec:

	docker exec debian cat /etc/debian_version

Ejecución de un comando con redireccionamiento de salida a un fichero de la máquina anfitrión

	docker exec debian cat /etc/debian_version > version.txt

Ejecución de un terminal dentro de un contenedor ya ejecutándose

	docker exec -it debian /bin/bash

# Volúmenes
Los datos que se almacenan dentro de un contenedor se pierden cuando el contenedor deja de existir. Para conseguir la persistencia de los datos, Docker utiliza 2 técnicas:
- Los montajes de enlace (*bind mount*): Son los mapeos de directorio entre la máquina host y el contenedor que ya hemos visto antes.
- Los volúmenes. Este es el mecanismo recomendado por Docker para la persistencia de datos. Son generados y usados por los contenedores para almacenar la información de un modo aislado de la máquina host y pueden ser fácilmente migrados de un contenedor a otro.

Para asegurar la persistencia de los datos incluso cuando no exista el contenedor y poder tener copias de seguridad o poder migrarlos a otro contenedor, usaremos los volúmenes.

Los volúmenes se pueden crear con antelación mediante el comando  *docker volume create* o en el momento de la creación del contenedor. Podemos especificar un nombre o ser anónimos.

Al lanzar el contenedor hay que especificar el directorio del contenedor que será montado en el volumen. En algunas imágenes, ya se especifica que al crear el contenedor se creen también los volúmenes correspondientes de forma automática, sin necesidad de tener que montarlo el usuario. Es el caso de la imagen de mysql

Lancemos un contendor mysql:

	docker run --name mibbdd -e MYSQL_ROOT_PASSWORD=123456 -d mysql

Si inspeccionamos el contenedor:

	docker inspect mibbdd

Veremos que se ha creado un volumen montado en el directorio */var/lib/mysql*

	"Mounts": [
            {
                "Type": "volume",
                "Name": "8ea4defefe165c72028d8fe1da9953572021674d4fda916436623da4b90df1f8",
                "Source": "/var/lib/docker/volumes/8ea4defefe165c72028d8fe1da9953572021674d4fda916436623da4b90df1f8/_data",
                "Destination": "/var/lib/mysql",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ]

## Listado de volúmenes
Podemos ver los volúmenes creados en Docker con el siguiente comando:

	docker volume ls

## Inspección de volúmenes
Podemos inspeccionar volúmenes con el siguiente comando indicando el nombre del volumen (usamos tabulador para autocompletar):

	docker volume inspect 8ea4defefe165c72028d8fe1da9953572021674d4fda916436623da4b90df1f8

## Eliminación de un volumen
Si eliminamos un contenedor, no se borran los volúmenes asociados. Hay que eliminarlos manualmente. Del mismo modo, no se pueden eliminar volúmenes si están asociados a un contenedor.

Para eliminar un volumen utilizamos el comando *docker volume rm*: 

	docker stop mibbdd
	docker rm mibbdd
	docker volume rm 8ea4defefe165c72028d8fe1da9953572021674d4fda916436623da4b90df1f8

## Creación de un volumen
No es necesario crear previamente un volumen, puede hacerse directamente al crear el contenedor, pero si lo creamos manualmente podemos dar un nombre más amigable y representativo:

	docker volume create mis_datos_sql

## Uso de volúmenes al crear el contenedor
Lanzamos un contendor mysql especificando esta vez el nombre del volumen que hemos creado previamente y el punto de montaje dentro del contenedor:

	docker run --name mibbdd -v mis_datos_sql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql

Creamos una BBDD en él:

	docker exec -it mibbdd mysql -p

	CREATE DATABASE instituto;
	USE instituto;
	CREATE TABLE alumnos (nombre VARCHAR(30), edad INT);
	INSERT INTO alumnos VALUES ('Jose Sanchez', 22);
	INSERT INTO alumnos VALUES ('Alberto Jurado', 19);
	exit

Comprobamos que los datos se han guardado en la BBDD

	docker exec -it mibbdd mysql -p123456 -e "USE instituto; SELECT * FROM alumnos;"

Si paramos y eliminamos el contenedor, al no eliminarse el volumen,los datos no se pierden.

	docker stop mibbdd
	docker rm mibbdd

Podemos crear otro contenedor usando el mismo volumen y comprobamos que los datos no se han perdido.
	
	docker run --name otrabbdd -v mis_datos_sql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql
	docker exec -it otrabbdd mysql -p123456 -e "USE instituto; SELECT * FROM alumnos;"

# Portainer
Portainer es una aplicación empaquetada en un contenedor que nos permite administrar gráficamente todos los elementos de Docker (imágenes, contenedores, volúmenes, redes, ...), además de gestionar clusters, plantillas de imágenes y muchas cosas más.

Para instalar portainer creamos un volumen, lo montamos al crear el contenedor y exponemos el puerto 9000, que será por el que accedamos a la aplicación mediante un navegador.

	docker volume create portainer_data
	docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer

# Redes
Visualizar redes creadas

	docker network ls

Para crear un red con los valores por defecto (Modo Bridge y direcciones automáticas)
	
	docker network create redpelli

Para asignar una dirección de red en el momento de la creación de la red:

	docker network create --subnet=192.168.0.0/16 redpelli

Para inspeccionar una de las redes creadas

	docker network inspect redpelli

Si queremos asignar una red a un contenedor en el momento de crearlo:

	docker run -dit --name alpine1 --network redpelli alpine

Comprobamos la IP asignada que está dentro del rango de la red creada:

	docker exec alpine1 ip a

Podría pasar que hubiéramos creado un contenedor con la red por defecto y deseáramos conectarlo una vez está corriendo a una nueva red:

	docker run -dit --name alpine2 alpine
	docker exec alpine2 ip a
	docker network connect redpelli alpine2
	docker exec alpine2 ip a

Del mismo modo se puede desconectar un contenedor de una red:

	docker network disconnect redpelli alpine2

Al unir un contenedor a una red (tanto a la hora de crearlo como posteriormente una vez creado) le podemos asignar un nombre al contenedor (alias) que Docker resolverá como nombre DNS

	docker network connect redpelli --alias=alpine2 alpine2
	docker exec alpine1 ping alpine2 -c 4

Para eliminar una red (si no tiene ningún contenedor conectado):

	docker network rm redpelli


# Limpieza del sistema
Podemos ver la ocupación en disco de todos los elementos docker con el comando:

	docker system df

Eliminación de las imágenes que no contengan ningún contenedor asociado

	docker image prune -a

Eliminación de los contenedores parados

	docker container prune

Eliminación de los volúmenes no usados por ningún contenedor

	docker volume prune

Eliminación de todos los elementos que no estén en uso

	docker system prune
	
# Creación de imágenes
Docker permite la creación de imágenes de dos modos:
- A partir de un contenedor existente
- Mediante un archivo Dockerfile

## Creación de imágenes a partir de contenedores
Para convertir un contenedor en una imagen utilizamos el mandato commit:

	docker commit contenedor imagen


## Creación de imágenes mediante archivo Dockerfile

# Portabilidad y copia de seguridad de datos
# Docker Compose
