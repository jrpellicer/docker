---
layout: page
title: Redes
nav_order: 6
---

# Redes

Al instalar Docker se establecen de forma predefinida 3 redes internas con las que podemos trabajar. Estas redes no se pueden eliminar y están siempre presentes:
- Red “bridge”: es la red por defecto de cualquier contenedor, dando una IP propia. Para funcionar utiliza una interfaz de red virtual en la máquina anfitrión llamada “docker0”.
- Red “host”: si un contenedor utiliza esta red, estar utilizando la misma configuración de red de la máquina anfitrión.
- Red “none”: red no permite acceso a otras redes. Solo permite el acceso a la interfaz de loopback.

Además de estas 3 redes predifinidas, Docker nos permite crear nuestras propias redes y conectar los contenedores a ellas.

Para visualizar las redes creadas ejecutamos:

	docker network ls

Para crear un red con los valores por defecto (Modo Bridge y direcciones automáticas):
	
	docker network create redpelli

Para asignar una dirección de red en el momento de la creación de la red:

	docker network create --subnet=192.168.0.0/16 redpelli

Para inspeccionar una de las redes creadas:

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

Al unir un contenedor a una red (tanto a la hora de crearlo como posteriormente una vez creado) le podemos asignar un nombre al contenedor (alias) que Docker resolverá como nombre DNS:

	docker network connect redpelli --alias=alpine2 alpine2
	docker exec alpine1 ping alpine2 -c 4

Para eliminar una red (si no tiene ningún contenedor conectado):

	docker network rm redpelli