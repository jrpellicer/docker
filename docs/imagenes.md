---
layout: page
title: Imágenes
nav_order: 3
---
# Imágenes

Cuando creamos un contenedor, éste se basará en el contenido de una imagen para funcionar. Un contenedor depende de una imagen para su creación, pero una vez creado y en funcionamiento cuando se realizan las diferentes tareas y modificaciones sobre el contenedor, esos cambios solo se verán reflejados en el contenedor y no en la imagen original.

En el Docker Hub ([https://hub.docker.com/](https://hub.docker.com/)) existen multitud de imágénes oficiales listas para descargar y ser usadas para crear nuestros contenedores. También podemos crear de manera sencilla nuestras propias imágenes.

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
