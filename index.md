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

Ejecución de un contenedor indicando el nombre

	docker run -d --name web httpd

Ejecución de un contenedor con terminal interactiva

	docker run -dti --name web httpd

Ejecución de un contenedor con mapeo de puertos

	docker run -dti --name web -p 80:80 httpd

Ejecución de un contenedor con mapeo de directorios

	docker run -dti --name web -p 80:80 -v "$PWD":/usr/local/apache2/htdocs/ httpd
	docker run -dti --name web -p 80:80 -v /home/usuario/directorio:/usr/local/apache2/htdocs/ httpd
