## Esquema para el ejercicio
![Imagen](esquema-ejercicio3.PNG)

### Crear red net-wp
docker network create net-wp
### Para que persista la información es necesario conocer en dónde mysql almacena la información.
En el esquema del ejercicio carpeta del contenedor (a) es `/var/lib/mysql`

Ruta carpeta host: .../ejercicio3/db

### ¿Qué contiene la carpeta db del host?
Está vacía al inicio. Es donde MySQL guardará sus archivos de datos.

### Crear un contenedor con la imagen mysql:8  en la red net-wp, configurar las variables de entorno: MYSQL_ROOT_PASSWORD, MYSQL_DATABASE, MYSQL_USER y MYSQL_PASSWORD
```powershell
docker run -d --name server-mysql --network net-wp -e MYSQL_ROOT_PASSWORD=rootpass -e MYSQL_DATABASE=db_wp -e MYSQL_USER=user_wp -e MYSQL_PASSWORD=wppass -v "C:/Users/Alisson Espín/Desktop/EPN/Practica1/ejercicio3/db:/var/lib/mysql" mysql:8
```

### ¿Qué observa en la carpeta db que se encontraba inicialmente vacía?
La carpeta `db` deja de estar vacía: al arrancar por primera vez, MySQL inicializa su directorio de datos (`/var/lib/mysql` del contenedor), que por el bind mount es en realidad esta carpeta del host. Se observa:

- **Carpetas de bases de datos:**
  - `db_wp`: la base creada con `MYSQL_DATABASE`. Contiene las tablas de WordPress como archivos `.ibd` (`wp_posts.ibd`, `wp_users.ibd`, `wp_options.ibd`, etc.) una vez instalado.
  - `mysql`, `sys` y `performance_schema`: bases internas del sistema (usuarios, privilegios, métricas).
- **Almacenamiento de InnoDB:** `ibdata1`, `mysql.ibd`, `undo_001`, `undo_002`, `#innodb_redo`, `#innodb_temp` y los archivos `#ib_*.dblwr`.
- **Registros binarios:** `binlog.000001`, `binlog.000002` y `binlog.index`.
- **Certificados y claves TLS:** `ca.pem`, `server-cert.pem`, `client-cert.pem`, `private_key.pem` y otros, que MySQL genera automáticamente.
- **Otros:** `auto.cnf` (identificador único del servidor) y `mysql.sock`.

Esto demuestra que los datos de la base ya no viven dentro del contenedor, sino en el host. Por eso, al eliminar `server-mysql` y crearlo otra vez con el mismo montaje, MySQL detecta que el directorio ya está inicializado, reutiliza estos archivos y conserva `db_wp` con todo su contenido.

### Para que persista la información es necesario conocer en dónde wordpress almacena la información.
En el esquema del ejercicio la carpeta del contenedor (b) es `/var/www/html`

Ruta carpeta host: .../ejercicio3/www

### Crear un contenedor con la imagen wordpress en la red net-wp, configurar las variables de entorno WORDPRESS_DB_HOST, WORDPRESS_DB_USER, WORDPRESS_DB_PASSWORD y WORDPRESS_DB_NAME (los valores de estas variables corresponden a los del contenedor creado previamente)
```powershell
docker run -d --name server-wp --network net-wp -p 8080:80 -e WORDPRESS_DB_HOST=server-mysql -e WORDPRESS_DB_USER=user_wp -e WORDPRESS_DB_PASSWORD=wppass -e WORDPRESS_DB_NAME=db_wp -v "C:/Users/Alisson Espín/Desktop/EPN/Practica1/ejercicio3/www:/var/www/html" wordpress
```

### Personalizar la apariencia de wordpress y agregar una entrada

### Eliminar el contenedor y crearlo nuevamente, ¿qué ha sucedido?

La instalación de WordPress, el tema `Twenty Twenty-Five` y la entrada `Mi primera entrada` siguen intactos. La base de datos persiste en `db` y los archivos de WordPress en `www`, ambos en el host, por lo que al recrear los contenedores con los mismos montajes se recupera toda la información.

