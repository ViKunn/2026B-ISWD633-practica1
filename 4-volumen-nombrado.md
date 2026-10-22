## VOLUMEN NOMBRADO
Un volumen nombrado (named volume) es un tipo de volumen gestionado por Docker que se almacena en una ubicación específica del sistema de archivos del host y se identifica mediante un nombre único. Los volúmenes nombrados no requieren que especifiques una ruta del sistema de archivos del host, y en su lugar, Docker se encarga de la gestión y el almacenamiento del volumen.


### Crear volumen
```
docker volume create <nombre volumen>
```

### Crear el volumen nombrado: vol-postgres
```
docker volume create vol-postgres
```

## MOUNTPOINT
Un mountpoint se refiere al lugar en el sistema de archivos donde un dispositivo de almacenamiento se une (o monta) al sistema de archivos. Es el punto donde los archivos y directorios almacenados en ese dispositivo de almacenamiento son accesibles para el sistema operativo y las aplicaciones.

Por ejemplo, en Windows las unidades de almacenamiento (como `C:`, `D:`, etc.) actúan como puntos de montaje principales para discos duros, unidades flash, unidades ópticas y otros dispositivos de almacenamiento.

Cuando creas un volumen nombrado, Docker asigna un punto de montaje específico en el sistema de archivos del host para ese volumen.

### Estructura del Punto de Montaje:
- /var/lib/docker/volumes/: Es la ubicación base donde Docker almacena todos los volúmenes en el sistema de archivos del host.
- nombreVolumen/: Es el nombre del volumen nombrado que has creado. Docker crea un directorio con este nombre dentro de /var/lib/docker/volumes/ para almacenar los datos del volumen.
- _data: Es el subdirectorio dentro de vol-postgres/ donde se almacenan los datos reales del volumen. El nombre _data es una convención utilizada por Docker para indicar el directorio donde se encuentran los datos del volumen.

### ¿Cómo acceder a ese Mountpoint?
En el contexto de WSL (Windows Subsystem for Linux), wsl$ se refiere al nombre de un recurso compartido de red especial que representa la raíz del sistema de archivos de Windows desde WSL. Cuando accedes a \\wsl$ desde el Explorador de archivos de Windows, puedes ver y acceder a los archivos del sistema de archivos de la distribución de Linux en WSL.
\\wsl.localhost\docker-desktop-data\data\docker\volumes

### Crear un contenedor vinculado a un volumen nombrado
```
docker run -d --name <nombre contenedor> -v <nombre volumen>:<ruta contenedor> <nombre imagen>
```
ó
```
docker run -d --name <nombre contenedor> --mount type=volume,src=<nombre >,dst=<mount-path>
```
- destination, dst, target: La ruta donde se monta el archivo o directorio en el contenedor.
- source, src: El origen del montaje. Para volúmenes con nombre, este es el nombre del volumen. Para volúmenes anónimos, este campo se omite.


### Crear la red net-drupal de tipo bridge
```
docker network create --driver bridge net-drupal
```

### Crear un servidor postgres vinculado a la red net-drupal, completar la ruta del contenedor
```
docker run -d --name server-postgres -e POSTGRES_DB=db_drupal -e POSTGRES_PASSWORD=12345 -e POSTGRES_USER=user_drupal --network net-drupal -v vol-postgres:/var/lib/postgresql postgres
```
_Desde la versión 18 de la imagen, los datos se guardan en un subdirectorio versionado, por lo que el volumen se monta en `/var/lib/postgresql` y no en `/var/lib/postgresql/data`. Con la ruta antigua el contenedor termina con error al iniciar._

_No es necesario exponer el puerto, debido a que nos vamos a conectar desde la misma red de docker_

### Crear un cliente postgres vinculado a la red drupal a partir de la imagen dpage/pgadmin4, completar el correo
```
docker run -d --name client-postgres --publish published=9500,target=80 -e PGADMIN_DEFAULT_PASSWORD=54321 -e PGADMIN_DEFAULT_EMAIL=alisson.espin@epn.edu.ec --network net-drupal dpage/pgadmin4
```

### Usar el cliente postgres para conectarse al servidor postgres, para la conexión usar el nombre del servidor en lugar de la dirección IP.
Se abre `http://localhost:9500` e inicia sesión con `alisson.espin@epn.edu.ec` y la contraseña `54321`. 

Luego, en **Add New Server**, se registra el servidor con estos datos:
- Name: `server-postgres`
- Host name/address: `server-postgres` (el nombre del contenedor, que Docker resuelve dentro de la red `net-drupal`)
- Port: `5432`
- Maintenance database: `db_drupal`
- Username: `user_drupal`
- Password: `12345`

### Crear los volúmenes necesarios para drupal, esto se puede encontrar en la documentación
```
docker volume create drupal-modules
docker volume create drupal-profiles
docker volume create drupal-themes
docker volume create drupal-sites
```

### Crear el contenedor server-drupal vinculado a la red, usar la imagen drupal, y vincularlo a los volúmenes nombrados
```
docker run -d --name server-drupal --publish published=9700,target=80 -v drupal-modules:/var/www/html/modules -v drupal-profiles:/var/www/html/profiles -v drupal-themes:/var/www/html/themes -v drupal-sites:/var/www/html/sites --network net-drupal drupal
```

### Ingrese al server-drupal y siga el paso a paso para la instalación.
![Paso 4 de la instalación de Drupal](volumen-nombrado-paso4.png)

_La instalación puede tomar varios minutos, mientras espera realice un diagrama de los contenedores que ha creado en este apartado._

```mermaid
flowchart LR
    subgraph red["Red net-drupal (bridge)"]
        pg["server-postgres<br/>postgres :5432"]
        pga["client-postgres<br/>dpage/pgadmin4 :80"]
        dr["server-drupal<br/>drupal :80"]
    end
    vpg[("vol-postgres")] -- "/var/lib/postgresql" --- pg
    vm[("drupal-modules")] -- "/var/www/html/modules" --- dr
    vp[("drupal-profiles")] -- "/var/www/html/profiles" --- dr
    vt[("drupal-themes")] -- "/var/www/html/themes" --- dr
    vs[("drupal-sites")] -- "/var/www/html/sites" --- dr
    host9500["localhost:9500"] --> pga
    host9700["localhost:9700"] --> dr
    pga -. "conecta por nombre" .-> pg
    dr -. "conecta por nombre" .-> pg
```

### Eliminar un volumen específico
```
docker volume rm <nombre volumen>
```

**Considerar**
Datos Persistentes: Asegúrate de que el volumen no contiene datos críticos antes de eliminarlo, ya que esta operación no se puede deshacer.
Contenedores Activos: No puedes eliminar un volumen que está actualmente en uso por un contenedor activo. Debes detener y/o eliminar el contenedor primero.
