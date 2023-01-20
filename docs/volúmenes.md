---
layout: page
title: Volúmenes
nav_order: 5
---

# Volúmenes
{: .no_toc }

<details open markdown="block">
  <summary>
    Tabla de contenidos
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>


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