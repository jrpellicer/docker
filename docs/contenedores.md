---
layout: page
title: Contenedores
nav_order: 4
---

# Contenedores
{: .no_toc }

<details open markdown="block">
  <summary>
    Tabla de contenidos
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>


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

## Ejecución rápida sin creación ni descarga de imágenes
Podemos correr un contenedor directamente sin necesidad de hacer un *pull*, *create* y *start*

	docker run mysql

## Visualización de logs
Visualización del log y depuración de posibles errores en un contendor

	docker logs c31

## Opciones de ejecución y creación
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
