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

Lo he utilizado en la Raspberry Pi 3/2, y actualmente la utilizo con la Odroid C2, ya que está última tiene soporte para acelerar por hardware el decode de x265 (HEVC).

Este post es para recordarme como armar el media center con contenedores docker en LibreELEC y poder correr un MySQL como servidor central para la DB, habilitando la base de datos compartida entre todos los dispositivos que corran Kodi en la red (Kodi como aplicación además de Linux corre en Windows, Max OS X, Android e iOS).

Algunos pasos hay que realizarlos en Kodi/LibreELEC primero.

1. Habilitar el la espera de conexión de red antes de iniciar Kodi; en el menú "LibreELEC" -> "Settings" -> "Network", habilitar "Wait for network before starting kodi", y definir el tiempo maximo (10 sec, por defecto).

2. Luego en "Addons", buscamos "Docker", de Team LibreELEC. Con esto tendremos el community engine para poder correr contendores. 

3. Buscamos una Docker image para MySQL/MariaDB, en el caso de la Odroid C2, es un CPU ARMv8, es decir arquitectura aarch64. Por lo tanto, las imagenes oficiales de mysql no funcionan (solo tienen para x86). Actualmente hay distintas opciones, pero en su momento la única que encontre fue la de haotseng/ebspace: 
* https://github.com/haotseng/aarch64_dockerfiles 
* https://hub.docker.com/r/ebspace/aarch64-mysql/

Actualmente MariaDB publica imagenes arm64v8, por lo que yo recomendaria utilizar esas, que tendrán mejor soporte:
* https://hub.docker.com/r/arm64v8/mariadb/

Aquí la lista de imagenes de Docker Hub con soporte para arm64v8:
* https://hub.docker.com/u/arm64v8/


Creamos el directorio en el host de LibreELEC:

```mkdir /storage/mysql```

Instalamos y corremos la imagen de docker:
```
docker run -d -p 3306:3306 \
-v /storage/mysql:/var/lib/mysql \
--restart unless-stopped \
-e MYSQL_ROOT_PASSWORD=passrootmysql \
--name kodi-mysql \
ebspace/aarch64-mysql
```

Logueamos y creamos la DB y usuario para Kodi:
```
docker exec -it kodi-mysql mysql -u root -p
```

Si necesitamos revisar algo del contenedor, podemos ingresar por bash:
```
docker exec -it kodi-mysql bash
```

Luego hay que configurar la conexión en Kodi dentro de LibreELEC, en el host nuevamente:

```nano ~/.kodi/userdata/advancedsettings.xml```

Agregamos lo siguiente (incluyo un par de parametros relacionados a buffer y cache):

```
<advancedsettings>
<cache>
#        <memorysize>0</memorysize>
# number of bytes used for buffering streams in memory
#   When set to 0 the cache will be written to disk instead of RAM 
        <buffermode>1</buffermode>  
## Choose what to buffer:
#    0) Buffer all internet filesystems (like "2" but additionally also ftp, webdav, etc.) (default)
#    1) Buffer all filesystems (including local)
#    2) Only buffer true internet filesystems (streams) (http, etc.)
#    3) No buffer -->
        <readfactor>4.0</readfactor> 
# this factor determines the max readrate in terms of readbufferfactor * avg bitrate of a video file.
##This can help on bad connections to keep the cache filled. It will also greatly speed up buffering. Default value 4.0.
</cache>
# db
<videodatabase>
        <type>mysql</type>
	<host>127.0.0.1</host>
        <port>3306</port>
        <user>kodi</user>
        <pass>kodi</pass>
</videodatabase> 
<musicdatabase>
        <type>mysql</type>
	<host>127.0.0.1</host>
        <port>3306</port>
        <user>kodi</user>
        <pass>kodi</pass>
</musicdatabase>
<videolibrary>
        <importwatchedstate>true</importwatchedstate>
        <importresumepoint>true</importresumepoint>
</videolibrary>
</advancedsettings>
</musicdatabase>
<videolibrary>
        <importwatchedstate>true</importwatchedstate>
        <importresumepoint>true</importresumepoint>
</videolibrary>
</advancedsettings>
```




Última modificación: 2018-08-28
Versiones: 
* LibreELEC 8.2.5
* Arch: Odroid_C2.aarch64
* Docker: 8.2.114 (Team LibreELEC)

