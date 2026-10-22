# Mi aprendizaje

## Antes y después de la práctica

Antes de empezar, yo sabía que Docker servía para levantar contenedores y que eran algo así como "mini servidores" que se creaban y se borraban fácil. Lo que no tenía claro era qué pasaba con la información cuando un contenedor desaparecía. Pensaba que, si lo eliminaba, perdía todo lo que había dentro.

Después de hacer la práctica entendí que los datos se pueden guardar fuera del contenedor y que ahí está la clave para que no se pierdan. Comprobé con WordPress que podía eliminar los contenedores, volver a crearlos y encontrar el tema, la entrada y toda la instalación tal como los había dejado. Eso fue lo que más me gustó, porque se ve claro para qué sirve.

## Los tres tipos de montaje

- **Bind mount:** se enlaza una carpeta de mi computadora con una carpeta del contenedor. En nginx vi que al montar la carpeta `html` vacía aparecía un 403, porque el montaje tapaba el `index.html` que venía en la imagen. Al poner la plantilla de html5up en esa carpeta, el sitio apareció al instante. Sirve mucho para desarrollar, porque edito los archivos en mi equipo y el contenedor los usa directamente.
- **Volumen nombrado:** lo administra Docker y se identifica con un nombre. Lo usé con Postgres y Drupal. No me tengo que preocupar por la ruta en mi computadora, y se puede reutilizar si recreo el contenedor.
- **Volumen anónimo:** es parecido, pero Docker le pone un nombre aleatorio. Es más difícil de reconocer después, así que sirve para datos temporales. Aprendí que con `docker rm -fv` se elimina el contenedor junto con su volumen, y que `docker volume prune` limpia los que ya nadie usa.

## Otras cosas que aprendí

- Hay que leer la documentación de cada imagen para saber qué carpetas conviene guardar: `/var/lib/mysql` en MySQL, `/var/www/html` en WordPress y `/usr/share/nginx/html` en nginx.
- Los contenedores de una misma red se comunican por nombre. Por eso en pgAdmin y en Drupal puse `server-postgres` como servidor en lugar de una dirección IP.
- Los volúmenes se definen al crear el contenedor, no se pueden cambiar después. Si algo está mal, toca eliminarlo y crearlo de nuevo.
- Las variables de entorno de las bases de datos (usuario, contraseña, nombre de la base) solo se aplican la primera vez que el volumen está vacío.

## Problemas que tuve y cómo los resolví

1. **Postgres se caía al iniciar.** El contenedor `server-postgres` terminaba con error. Con `docker logs server-postgres` vi que las versiones 18 o superiores de la imagen no aceptan el volumen en `/var/lib/postgresql/data`. Lo resolví montándolo en `/var/lib/postgresql`.
2. **No podía entrar a pgAdmin.** Había dejado el correo de ejemplo en el comando y, además, en el login estaba escribiendo algo que no era un correo. Los logs decían que el email debía tener un `@`. Eliminé el contenedor, lo recreé con mi correo real e inicié sesión con ese correo y la contraseña que definí. El usuario `user_drupal` solo se usa para registrar el servidor dentro de pgAdmin.
3. **El tema de WordPress no cambiaba.** Instalé un tema nuevo, pero no lo había activado. En WordPress instalar y activar son dos pasos distintos.
4. **Error de nombre en uso.** Al volver a ejecutar un `docker run` me salía que el nombre del contenedor ya existía. Entendí que el nombre es único: o el contenedor ya está corriendo, o hay que borrarlo con `docker rm -f` antes de crearlo otra vez.
5. **Se me pasó la captura del instalador de Drupal.** Tuve que eliminar los contenedores y volúmenes de Drupal y Postgres y volver a crearlos para que el instalador mostrara otra vez el paso 4.

## Comandos extra que usé

- `docker logs <contenedor>` para ver por qué fallaba un contenedor.
- `docker inspect <contenedor>` para revisar variables de entorno y montajes.
- `docker volume ls` para ver los volúmenes existentes.
- `docker volume rm` para eliminar volúmenes concretos.
- `docker restart <contenedor>` para reiniciar uno sin recrearlo.

## Para mi formación profesional

Me llevo la idea de que separar la aplicación de sus datos es algo básico al trabajar con contenedores. También aprendí a no quedarme con el primer error: mirar los logs y la documentación de la imagen casi siempre da la respuesta. Creo que esto me va a servir cuando tenga que desplegar sistemas con base de datos, porque ya sé cómo evitar perder información al actualizar o recrear un servicio.
