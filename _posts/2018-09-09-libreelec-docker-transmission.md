---
layout: post
title: Usando Docker en LibreELEC para correr Transmission
image: /img/hello_world.jpeg
---

LibreELEC (un fork de OpenELEC) es una distribución especializada para correr Kodi, una aplicación de software libre para media centers o home theaters.

Este post es una continuación de:
* Instalando LibreELEC en la Odroid C2
* Usando Docker en LibreElec para correr MySQL

La configuración es insegura. Esta orientada a la conveniencia y disponibilidad (el rpc de Transmission no tendra usuario/clave).

Utilizamos la Odroid con un disco externo USB, que está montado en /var/media/discousb y es donde Transmission almacena las descargas y Kodi las lee.

Creamos el directorio y subdirectorios para montar el volumen del contenedor contra el filesystem del host (lo separo por claridad):

En el host:

```mkdir /storage/transmission```
```mkdir /storage/transmission/config```
```mkdir /storage/transmission/watched```


Creamos la imagen de docker:

```
docker create --name=transmission \
--restart unless-stopped \
-v /storage/transmission/config:/config \
-v /var/media/discousb:/downloads \
-v /storage/transmission/watch:/watch \
-e PGID=0 -e PUID=0 \
-e TZ=America/Argentina/Buenos_Aires \
-p 9091:9091 -p 51413:51413 \
-p 51413:51413/udp \
lsioarmhf/transmission-aarch64
```

Nos aseguramos de correr el contendor para generar los archivos de configuración:

```docker start transmission```

Nos aseguramos que este detenido el contendor antes de modificar las settings

```docker stop transmission```

Editamos el settings.json en ```/storage/transmission/config/settings.json``` y agregamos el hostname (o fqdn si aplica) con el que accedemos al odroid/tranmission:

```
    "rpc-host-whitelist": "odroid",
```

Volvemos a levantar el equipo:

```
docker start transmission
```


Ya nos podemos conectar por WebUI o con un transmission remote:

Host: odroid
Port: 9091



Versiones: 
* LibreELEC 8.2.5
* Arch: Odroid_C2.aarch64
* Docker: 8.2.114 (Team LibreELEC)

