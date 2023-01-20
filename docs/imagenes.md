---
layout: page
title: Imágenes
nav_order: 3
---

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
