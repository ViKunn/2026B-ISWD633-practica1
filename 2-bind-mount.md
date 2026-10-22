# BIND MOUNT
En un bind mount mapeamos (montar) un directorio o archivo específico del sistema de archivos del host con una parte del sistema de ficheros del contenedor.

```
docker run -d --name <nombre contenedor> -v <ruta carpeta host>:<ruta carpeta contenedor> <imagen> 
```
ó
```
docker run -d --name <nombre contenedor> --mount type=bind,source=<ruta carpeta host>,target=<ruta carpeta contenedor> <imagen>
```
- destination, dst, target: La ruta donde se monta el archivo o directorio en el contenedor.
- source, src: El origen del montaje.
  
### En tu computador crear una carpeta llamada nginx y dentro de esta carpeta crea otra llamada html. Como se aprecia en la figura.
![Volúmenes](directorio.PNG)

### Crear un contenedor con la imagen nginx:alpine, mapear todos por puertos, para la ruta carpeta host colocar el directorio en donde se encuentra la carpeta html en tu computador y para la ruta carpeta contenedor: /usr/share/nginx/html (esta ruta se obtiene al revisar la documentación de la imagen)
![Volúmenes](volumen-host.PNG)

```cmd
docker run -d --name server-nginx -P -v "C:/Users/Alisson Espín/Desktop/EPN/Practica1/nginx/html:/usr/share/nginx/html" nginx:alpine
```

### ¿Qué sucede al ingresar al servidor de nginx?
Muestra 403 Forbidden o una página vacía, porque la carpeta `html` del host está vacía y `nginx` no encuentra un `index.html`.
### ¿Qué pasa con el archivo index.html del contenedor?
Queda oculto por el montaje, porque el directorio del host sustituye al del contenedor. El archivo sigue existiendo en la imagen, pero no es visible.

### Ir a https://html5up.net/ y descargar un template gratuito, descomprirlo dentro de tu computador en la carpeta html

### ¿Qué sucede al ingresar al servidor de nginx?
Se ve la plantilla web. Los cambios en el host se reflejan al instante, sin reiniciar.

### Eliminar el contenedor
```cmd
docker rm -f server-nginx
```
### ¿Qué sucede al crear nuevamente un contenedor montado al directorio definidos anteriormente?
La plantilla sigue ahí, porque los datos están en el host y no en el contenedor.