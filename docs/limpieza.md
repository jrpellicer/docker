---
layout: page
title: Limpieza del sistema
nav_order: 7
---

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