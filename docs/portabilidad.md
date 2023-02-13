---
layout: page
title: Portabilidad y Backup
nav_order: 10
---

# Portabilidad y copia de seguridad de datos
{: .no_toc }

<details open markdown="block">
  <summary>
    Tabla de contenidos
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

La facilidad de creación de contenedores y de imágenes permite que se puedan portar contenedores a otras máquinas de un modo muy sencillo y rápido, simplemente creando el contenedor a partir de la imagen del hub o a partir de una copia previamente hecha por nosotros. El inconveniente de estos métodos es que se despliega el contenedor pero NO se salvan los datos.

Por este motivo hay que diferenciar entre hacer una copia de seguridad del contenedor y una copia de seguridad de los volúmenes que contienen los datos del contenedor.

## Copia y restauración del contenedor (sin datos)

Es posible hacer una copia de un contenedor mediante el comando **export** y almacenarlo en un fichero *tar*. La sintaxis es la su¡iguiente:

    docker export contenedor > fichero.tar

El contenedor puede estar parado o en ejecución.

    docker export mi_ubuntu > mi_ubuntu.tar

La restauración del contenedor consiste en la creación de una imagen basada en el contenido de la copia de seguridad del contenedor. Una vez restaurada la imagen, hay que crear el contenedor. Se utiliza el comando **import**

    docker import mi_ubuntu.tar mi_ubuntu:1.0
    docker run -ti mi_ubuntu:1.0

El nuevo contenedor creado a partir de la imagen restaurada **no contendrá los datos** que pudiera tener el contenedor original sobre el que se hizo la copia.

## Copia y restauración de los datos de un contenedor

En el apartado de [volúmenes](../vol%C3%BAmenes) hemos visto que hay 2 posibilidades para tener persistencia de datos:
- Los montajes de enlace (*bind mount*): Son los mapeos de directorio entre la máquina host y el contenedor.
- Los volúmenes. Son generados y usados por los contenedores para almacenar la información de un modo aislado de la máquina host y pueden ser fácilmente migrados de un contenedor a otro.

En el primer caso, hacer una copia de los datos es tan sencillo como hacer una copia del directorio de la máquina host que estamos compartiendo con el contenedor.

En el segundo caso, los volúmenes también se encuentran montados en una carpeta de trabajo de docker, pero es más sencillo hacer una copia de seguridad mediante una técnica consistente en crear un contenedor montando los volúmenes que queremos hacer copia, ejecutar internamente el mandato de copia y almacenar la copia en un fichero de la máquina host. Esto que parece muy complicado se consigue fácilmente ejecutando la siguiente orden:

    docker run --rm --volumes-from CONTENEDOR -v $(pwd):/backup busybox tar cvf /backup/backup.tar carpeta1 carpeta2 carpeta3

El comando ejecuta un contenedor con la iamgen *busybox*, una imagen de Linux muy ligera con algunas utilidades y pequeños ejecutables. Una vez ejecutado el contenedor se elimina (opción --rm). En la creación se montan los volúmenes que hay asociados al contenedor del que queremos hacer la copia (`--volumes-from CONTENEDOR`). También se monta una carpeta que enlaza el directorio actual de la máquina host (`pwd`) con el directorio `/backup` del contenedor. Por último, la orden que se ejecuta en el contenedor es `tar cvf /backup/backup.tar carpeta1 carpeta2`, que realiza la copia de seguridad de las carpetas asociadas a los volúmenes montados (carpeta1, carpeta2, carpeta3, ...).

Para restaurar los datos, hacemos lo mismo pero con el comando tar inverso:

    docker run --rm --volumes-from CONTENEDOR -v $(pwd):/backup busybox tar xvf /backup/backup.tar

### Ejemplo de Copia y restauración de datos de un volumen

Lancemos un contendor mysql con nombre mibbdd:

	docker run --name mibbdd -e MYSQL_ROOT_PASSWORD=123456 -d mysql

Si inspeccionamos el contenedor:

	docker inspect mibbdd

Veremos que se ha creado automáticamente un volumen montado en el directorio */var/lib/mysql* del contenedor.

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

Vamos a realizar una copia de seguridad indicando el directorio del contenedor donde está montado el volumen con los datos (`/var/lib/mysql`).

    docker run --rm --volumes-from mibbdd -v $(pwd):/backup busybox tar cvf /backup/backup.tar /var/lib/mysql

Ya está hecha la copia de seguridad en el fichero `backup.tar` del directorio actual de la máquina host.

Ahora vamos a proceder a restaurar la copia, para ello eliminamos antes el contendor y los volúmenes que estén sin contenedores asociados (recordemos que al eliminar un contenedor no se eliminan los volúmenes).

    docker stop mibbdd
    docker rm mibbdd
    docker volume prune

Y ahora lanzamos un nuevo contenedor con el mismo u otro nombre.

    docker run --name mibbdd -e MYSQL_ROOT_PASSWORD=123456 -d mysql

Antes de restaurar la copia, paramos elcontenedor para que se detengan los servicios:

    docker stop mibbdd

Restauramos la copia de seguridad:

    docker run --rm --volumes-from mibbdd -v $(pwd):/backup busybox tar xvf /backup/backup.tar

Ya tenemos los datos restaurados. Arrancamos de nuevo el contenedor:

    docker start mibbdd

Comprobamos que tenemos los datos restaurados lanzando una consulta:

    docker exec -it mibbdd mysql -p123456 -e "USE instituto; SELECT * FROM alumnos;"