---
layout: page
title: Instalación
nav_order: 2
---

# Instalación de Docker
La forma más sencilla de instalación es mediante el script de instalación rápida descargado de la página oficial. Hace de forma automática todos los pasos que se encuentran en la documentación de instalación, sin importarnos la plataforma ni la distribución de Linux sobre la que ejecutamos el script.

	sudo su
	curl –fsSL https://get.docker.com/ | sh

Para comprobar la información de funcionamiento:

	docker info

{: .warning }
En algunas ocasiones es necesario reiniciar el sistema con reboot.

Añadimos nuestro usuario al grupo *docker* para tener privilegios de ejecución sobre docker:

	usermod $USER -aG docker

Para que los cambios en el grupo surtan efecto en la sesión iniciada podemos ejecutar el siguiente comando:

	newgrp docker

{: .note }
Si nos sigue pidiendo privilegios de administrador (sudo), será necesario cerrar la sesión del usuario y volver a iniciarla.

