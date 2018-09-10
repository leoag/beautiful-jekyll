---
layout: post
title: Usando Docker en LibreELEC para correr MySQL
image: /img/hello_world.jpeg
---

LibreELEC (un fork de OpenELEC) es una distribución especializada para correr Kodi, una aplicación de software libre para media centers o home theaters.

Actualmente corre sobre el siguiente hardware:
* Raspberry Pi 0/1/2/3
* x86_64 devices with Intel, AMD and nVidia GPU's
* WeTek Play 1 and 2, WeTek Hub
* Odroid C2
* Cubox iMX6 devices

Yo la he utilizado en la Raspberry Pi 3, y actualmente la utilizo con la Odroid C2, ya que está última tiene soporte para acelerar por hardware el decode de x265 (HEVC).

Este post es para recordarme como montar contenedores docker en LibreELEC y poder correr un MySQL como servidor, habilitando la base de datos compartida entre todos los dispositivos que corran Kodi (Kodi como aplicación, también corre en Windows, Max OS X, Android, iOS y, por supuesto, Linux).

Primero hay que instalar Docker CE en LibreELEC/Kodi, mediante el gestor de Addons de la interface.

Habilitar la opción de "Wait for network before starting Kodi", ya que necesitamos que docker este funcionando y las interfaces de red en linea antes de que Kodi intente levantar la DB.

En el caso de la Odroid C2, es un CPU ARMv8, es decir arquitectura aarch64. Por lo tanto, las imagenes oficiales de mysql no funcionan (solo tienen para x86). Actualmente hay distintas opciones, pero en su momento la única que encontre fue la de haotseng: 
* https://github.com/haotseng/aarch64_dockerfiles 
* https://hub.docker.com/r/ebspace/aarch64-mysql/

Aquí la lista oficial de imagenes de docker para arm64v8:
* https://hub.docker.com/u/arm64v8/


Creamos el directorio:

```mkdir /storage/mysql```

Montamos la imagen de docker:
```
docker run -d -p 3306:3306 -v /storage/mysql:/var/lib/mysql --restart unless-stopped -e MYSQL_ROOT_PASSWORD=passrootmysql --name kodi-mysql ebspace/aarch64-mysql
```

Logueamos y creamos la DB y usuario para Kodi:
```
docker exec -it kodi-mysql mysql -u root -p
```

Si necesitamos revisar algo del contenedor, podemos ingresar por bash:
```
docker exec -it kodi-mysql bash
```


Última modificación: 2018-08-28
Versiones: 
* LibreELEC 8.2.5
* Arch: Odroid_C2.aarch64
* Docker: 8.2.114 (Team LibreELEC)

