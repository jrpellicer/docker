---
layout: page
title: Creación de imágenes
nav_order: 9
---

# Creación de imágenes
{: .no_toc }

Docker permite la creación de imágenes de dos modos distintos:
- A partir de un contenedor existente.
- Mediante un archivo Dockerfile.

<details open markdown="block">
  <summary>
    Tabla de contenidos
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

## Creación de imágenes a partir de contenedores
Si queremos convertir un contenedor creado (y modificado) en una imagen utilizamos el mandato commit:

	docker commit contenedor imagen

El nombre de la imagen puede ser un nombre cualquiera, pero si pretendemos subirlo a un repositorio (por ejemplo Docker Hub), es conveniente que pongamos el nombre de imagen con el usuario y la versión (tag) del siguiente modo:

	docker commit contenedor usuario/imagen:tag

### Ejemplo de creación de una imagen a partir de un contenedor
Vamos a crear una imagen nueva de Ubuntu con el editor nano instalado.

Ejecutamos un contenedor a partir de la imagen original de Ubuntu:

	docker run --name mi_ubuntu -dti ubuntu

Comprobamos que no está el editor nano:

	docker exec -ti mi_ubuntu nano

Nos falla el comando. Instalamos el paquete:

	docker exec mi_ubuntu apt update
	docker exec mi_ubuntu apt install nano -y

Comprobamos que ya está instalado el editor nano:

	docker exec -ti mi_ubuntu nano

Creamos una imagen nueva a partir del contenedor con el nano instalado (indicando nuestro usuario y la versión):

	docker commit mi_ubuntu jrpellicer/ubuntu_nano:1.0

Visualizamos las imágenes:

	docker images

## Creación de imágenes mediante archivo Dockerfile

Docker nos permite también crear una nueva imagen a partir de una imagen original e ir indicando mediante instrucciones los cambios hasta llegar a una nueva imagen. Estas instrucciones hay que incluirlas denrtro de un fichero llamado **Dockerfile**.

Los pasos a seguir para crear una imagen a partir de un fichero Dockerfile son los siguientes:
- Creamos un directorio en el que pondremos el fichero **Dockerfile** y otros ficheros que fueran necesarios incluir en nuestro contenedor (archivos de configuración, por ejemplo).
- Editamos el fichero **Dockerfile** e introducimos en él las instrucciones necesarias para la creación de la imagen.
- Ejecutamos el comando *docker build* para la creación de la imagen.

El comando para crear la imagen es:

	docker build -t nombre_nueva_imagen

### Instrucciones en Dockerfile

{: .note-title }
> Un ejemplo de fichero Dockerfile
>
> FROM ubuntu:14.04 
>
> RUN apt-get update && apt-get -y upgrade<br>
> RUN apt-get install -y mysql-server
>
> ADD my.cnf /etc/mysql/conf.d/my.cnf<br>
> ADD script.sh /usr/local/bin/script.sh
>
> RUN chmod +x /usr/local/bin/script.sh
>
> EXPOSE 3306
>
> CMD ["/usr/local/bin/script.sh"]

El fichero Dockerfile admite las siguientes expresiones:
- FROM
- RUN
- CMD
- EXPOSE
- ADD
- COPY
- ENTRYPOINT
- VOLUME
- ENV
- WORKDIR
- Otros comandos

#### FROM

Indica la imagen base sobre la que se va a crear la nueva. Debe ser la primera línea del fichero.

	FROM debian:latest

#### RUN

Mediante la expresión **RUN** se especifican los comandos que se deben ejecutar sobre la imagen base para generar la imagen final. Se pueden utilizar todos los comandos RUN que necesitemos dentro del fichero, pero es preferible agrupar los comandos mediante la expresión && debido a que durante la generación de la imagen, por cada comando RUN se creará un contenedor intermedio, haciendo más pesado el proceso de creación.

RUN admite 2 sintaxis posibles:
- RUN comando parámetro1 parámetro2
- RUN ["comando", "parámetro 1", "parámetro 2"]

Algunos ejemplos:

	RUN apt update && apt install -y apache2
	RUN ["apt", "install", "-y", "vim"]

#### CMD
Se utiliza para proporcionar un comando por defecto al crear un contenedor basado en esa imagen. Este comando es el punto de entrada del contenedor (si no se especifica otro al crearlo con run o create), y no se ejecutará durante la creación de la imagen.

Esta línea la ponemos al final del fichero Dockerfile.

	CMD /bin/bash

#### EXPOSE
La expresión **EXPOSE** se utiliza para especificar los puertos que estarán a la escucha dentro del contenedor. En ningún caso los expondrá automáticamente (habrá que hacerlo al crear el contenedor), peró sí que los habilitará para poder exponerlos después.

	EXPOSE 80 443 8080

#### ADD
Esta expresión la utilizamos para copiar nuestros ficheros dentro de la imagen durante el proceso de creación de la misma.

Esmuy útil cuando creamos nuestras imágenes con aplicaciones el poder copiar los ficheros de configuración u otros ficheros necesarios, como podría ser el contenido de una página web o la copia de seguridad de una BBDD a importar dentro del contenedor.

Es posible copiar también directorios completos.

Todos los ficheros o directorios a copiar deben estar en el mismo directorio que el fichero Dockerfile.

Si copiamos un fichero comprimido (tar, gzip, xz o bzip2) será extraído automáticamente y el fichero no se conservará dentro del contenedor, sólo su contenido. En caso de no querer eso, debemos utilizar el comando **COPY**.

	ADD miweb.conf /etc/apache2/sites-availables/
	ADD miweb/ /var/www/html/
	ADD motd.tar /etc/motd

#### COPY
El funcionamiento es similar a **ADD** para copiar ficheros dentro de la imagen a crear. La diferencia es que los archivos comprimidos (tar, gzip, xz o bzip2) no son extraídos y los copia tal cual.

Es la expresión recomendada a utilizar en lugar de ADD.

	COPY miweb.conf /etc/apache2/sites-availables/
	COPY miweb/ /var/www/html/
	COPY *.conf /etc/apache2

#### ENTRYPOINT
Por defecto, las imágenes son creadas para que todos los comandos indicados a ejecutar en el contenedor lo hagan utilizando */bin/sh*. A través de la instrucción **ENTRYPOINT** podemos cambiar el comportamiento por defecto.

También admite 2 sintaxis posibles:
- ENTRYPOINT comando parámetro1 parámetro2
- ENTRYPOINT ["comando", "parámetro 1", "parámetro 2"]

En el siguiente ejemplo, cuando ejecutemos un contendor, el argumento que le indiquemos será un parámetro de la instrucción cat.

	ENTRYPOINT ["cat"]

Se puede combinar con la expresión **CMD**

	ENTRYPOINT ["cat"]
	CMD ["/etc/debian_version"]

De este modo, si se ejecuta sin argumentos el contenedor, el comando que ejecutaría sería el cat seguido del parámetro por defecto que sería la ruta del fichero `/etc/debian_version`

#### VOLUME
Con la expresión **VOLUME** se crea un punto de montaje que podrá ser accesible por otros contenedores oo enlazarlo a un directorio de la máquina anfitrión.

Al crear un contenedor basado en esa imagen se creará automáticamente un volumen en el directorio del contenedor que hayamos indicado.

	RUN mkdir /datos && date > /datos/fecha.txt
	VOLUME /datos

#### ENV
Esta expresión nos permite establecer variables de entorno dentro el contenedor que use la imagen que estamos creando. Admite 2 sintaxis (en una podemos especificar más de una variable de entorno):

- ENV clave valor
- ENV clave=valor clave2=valor2 clave3=valor3

Ejemplo:

	RUN apt update && apt install -y locales locales-all
	ENV LC_ALL="es_ES.UTF-8" LANG="es_ES.UTF-8"

#### WORKDIR
Con esta expresión podemos modificar (las veces que queramos) el directorio de trabajo de los comandos RUN, CMD, ENTRYPOINT, ADD y COPY durante el proceso de creación de la imagen. Por defecto, el directorio de trabajoes la raíz (/).

	WORKDIR /tmp
	RUN echo "test" > test.txt
	WORKDIR /var/tmp
	RUN echo "test2" > test2.txt

### Ejemplo de creación de una imagen a partir de un Dockerfile

Dockerfile para un contenedor Ubuntu 18.04 con el servicio SSH instalado y los manuales en español.

	FROM ubuntu:18.04

	RUN apt-get update

	# No excluir man pages y otra documentación
	RUN rm /etc/dpkg/dpkg.cfg.d/excludes

	# Reinstalar todos los paquetes instalados para recuperar los manuales
	RUN dpkg -l | grep ^ii | cut -d' ' -f3 | xargs apt-get install -y --reinstall && \
		rm -r /var/lib/apt/lists/*

	# Instalar y configurar manual en español y reconfigurar locales
	RUN apt-get update && apt-get install -y man manpages-es manpages-es-extra locales
	ENV LC_MESSAGES='es_ES.UTF-8'
	ENV LANGUAGE='es_ES.UTF-8'
	ENV LANG='es_ES.UTF-8'
	RUN export LC_MESSAGES=es_ES.UTF-8
	RUN locale-gen es_ES.UTF-8

	# Instalar servidor SSH
	RUN apt-get install -y openssh-server
	RUN mkdir /var/run/sshd

	# Instalar aplicaciones sudo y nano
	RUN apt-get install -y sudo nano

	# Crear usuario alumno con la misma contraseña
	RUN useradd -m -G sudo -s /bin/bash alumno
	RUN echo "alumno:alumno" | chpasswd
	RUN echo "export LC_MESSAGES=es_ES.UTF-8" >> /home/alumno/.bashrc

	RUN apt-get clean && \
		rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

	EXPOSE 22
	CMD    ["/usr/sbin/sshd", "-D"]

Lanzamos el comando de creación de la imagen:

	docker build -t buntu_ssh