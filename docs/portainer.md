---
layout: page
title: Portainer
nav_order: 8
---

# Portainer
Portainer es una aplicación empaquetada en un contenedor que nos permite administrar gráficamente todos los elementos de Docker (imágenes, contenedores, volúmenes, redes, ...), además de gestionar clusters, plantillas de imágenes y muchas cosas más.

Para instalar portainer creamos un volumen, lo montamos al crear el contenedor y exponemos el puerto 9000, que será por el que accedamos a la aplicación mediante un navegador.

	docker volume create portainer_data
	docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer