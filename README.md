# Practica-iaw-1.9
Repositorio para la práctica 1.9. de IAW

Antonio Jesus Galvez Rodriguez

Durante esta practica aprenderemos a automatizar la instalación y configuración de una aplicación web LAMP en dos máquinas virtuales EC2 de Amazon Web Services (AWS). En una de las máquinas deberá instalar Apache HTTP Server y los módulos necesarios de PHP y en la otra máquina deberá instalar MySQL Server.

# 1 Maquina Frontend
En esta maquina instalaremos Apache HTTP Server con los modulos PHP necesarios y con su respectivo certificado con Certbot.

## 1.1 Archivo `install_lamp_frontend.sh`
En la cabecera de nuestro archivo creado pondremos lo siguiente:
```
#!/bin/bash
```
Esto indica que el script debe ejecutarse con el intérprete de Bash e irá siempre en la cabecera del archivo.
```
set -ex
```
Este comando hace que el script muestre cada comando antes de ejecutarlo `(-x)` y salga si algún comando falla `(-e)`.

## 1.1.1 Actualización del Sistema
### 1.1.1.1 Actualizar los repositorios
```
apt update
```
Actualiza la lista de paquetes disponibles en los repositorios configurados.

### 1.1.1.2 Actualizar los paquetes
```
apt upgrade -y
```
Actualiza todos los paquetes instalados a sus versiones más recientes. El `-y` automáticamente acepta todas las confirmaciones.

## 1.1.2 Instalación de Apache
### 1.1.2.1 Instalar el servidor web Apache
```
apt install apache2 -y
```
Instala el servidor web Apache. El -y hace lo mismo que se ha explicado anteriormente

### 1.1.2.1.1 Habilitar el módulo rewrite de Apache
```
a2enmod rewrite
```
Habilita el módulo rewrite de Apache, que es útil para la reescritura de URL's.

### 1.1.2.2 Creación del archivo de configuración de Apache
Crearemos una carpeta llamada `conf` en la que crearemos un primer archivo `000-default.conf` donde incluiremos el siguiente contenido:
```
<VirtualHost *:80>
    #ServerName www.example.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/


    DirectoryIndex index.php index.html


    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
### 1.1.2.2.1 Funcionamiento del archivo
`<VirtualHost *:80>`
Nos indica que este bloque de configuración se aplicará a las solicitudes HTTP en el puerto 80.

`# ServerName www.example.com`
Esta línea está comentada y no se ejecuta, pero puede ser usada para especificar el nombre del servidor (dominio).

`ServerAdmin webmaster@localhost`
Define la dirección del administrador del sitio. Los errores del servidor podrían enviarse a esta dirección.

`DocumentRoot /var/www/html/`
Especifica el directorio raíz desde el cual se servirán los archivos. Aquí es donde Apache buscará los archivos para darlos a los visitantes.

`DirectoryIndex index.php index.html`
Define la lista de archivos que Apache buscará y servirá como página predeterminada en un directorio. En este caso, buscará index.php primero, y si no lo encuentra, buscará index.html.

`ErrorLog ${APACHE_LOG_DIR}/error.log`
Establece la ubicación del archivo de registro de errores. ${APACHE_LOG_DIR} es una variable que apunta al directorio de logs de Apache.

`CustomLog ${APACHE_LOG_DIR}/access.log combined`
Define la ubicación del archivo de registro de accesos y el formato del log. combined es un formato de registro que incluye información detallada sobre cada solicitud.

### 1.1.2.2.2 Copiar archivo de configuración de Apache
```
cp ../conf/000-default.conf /etc/apache2/sites-available
```
Con esto vamos a conseguir copiar el archivo de configuración personalizado de Apache al directorio donde Apache busca sus configuraciones de sitios disponibles.

## 1.1.3 Instalación de PHP
### 1.1.3.1 Instalar PHP y los módulos de PHP para Apache y MySQL
```
apt install php libapache2-mod-php php-mysql -y
```
Instala PHP y los módulos necesarios para que Apache pueda procesar scripts PHP y para que PHP pueda interactuar con MySQL.

## 1.1.3.2 Reiniciar Apache
### 1.1.3.2.1 Reiniciar el servicio de Apache
```
systemctl restart apache2
```
Reinicia el servicio de Apache para aplicar los cambios de configuración y cargar los nuevos módulos instalados.

### 1.1.4 Configuración Adicional
### 1.1.4.1 Copiar el script de prueba de PHP a `/var/www/html`
```
cp ../php/index.php /var/www/html
```

Copia un script de prueba PHP a la raíz del servidor web, permitiendo verificar que PHP está funcionando correctamente.

### 1.1.4.2 Modificar el propietario y el grupo del archivo `index.php`
```
chown -R www-data:www-data /var/www/html
```
Cambia el propietario y el grupo del archivo index.php y de todos los archivos en el directorio /var/www/html a www-data, que es el usuario y grupo predeterminados de Apache.

# 1.2 Creacion del certificado mediante Let's Encrypt en el archivo `setup_letsencrypt_certificate.s`

## 1.2.1 Realizamos la instalación y actualización de snapd.
```
sudo snap install core
sudo snap refresh core
```

## 1.2.2 Eliminamos si existiese alguna instalación previa de certbot con apt.
```
sudo apt remove certbot -y
```

## 1.2.3 Instalamos el cliente de Certbot con snapd.
```
sudo snap install --classic certbot
```

También es posible especificar como argumentos las respuestas que nos hará certbot durante el proceso de instalación. Por ejemplo, las mismas respuestas que hemos dado durante la instalación manual se podrían haber indicado con los siguientes parámetros.

* Dirección de correo: `-m $LE_EMAIL`
* Aceptamos los términos de uso: `--agree-tos`
* No queremos compartir nuestro email con la Electronic Frontier Foundation: `--no-eff-email`
* Dominio: `-d $LE_DOMAIN`
Además, vamos a añadir el parámetro `--non-interactive` para que al ejecutar el comando no solicite al usuario ningún dato por teclado. Esta opción es útil cuando queremos automatizar la instalación de Certbot mediante un script.

```
certbot --apache -m $LE_EMAIL --agree-tos --no-eff-email -d $LE_DOMAIN --non-interactive
```

La dirección de correo y el dominio estarán en el archivo `.env` donde las declararemos con sus respectivos datos, de esta manera:

![](/img/Captura%20de%20pantalla%202024-12-01%20202512.png)

## 1.2.4 Creamos una alias para el comando certbot.
```
sudo ln -fs /snap/bin/certbot /usr/bin/certbot
```

Y este sería el resultado:

![](/img/Captura%20de%20pantalla%202024-12-01%20202606.png)

# 1.3 Despliegue de WordPress a través de WP-CLI con el archivo `deploy_wordpress_frontend.sh`

## 1.3.1 Eliminar instalaciones previas de WP-CLI
```
rm -rf /tmp/wp-cli.phar
```

Esto nos permitira que no haya dos archivos con el mismo contenido para evitar así errores en la instalación.

## 1.3.2 Descarga del codigo fuente de WP-CLI
```
wget https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar -P /tmp
```

Descargamos el codigo fuente de WP-CLI y lo guardamos en la carpeta `/tmp`.

## 1.3.3 Damos permisos de ejecucion al archivo `wp-cli.phar`
```
chmod +x /tmp/wp-cli.phar
```

Para poder ejecutarlo y que se pueda instalar WP-CLI en nuestra máquina deberemos darle permisos de ejecución.

## 1.3.4 Movemos el script WP-CLI al  directorio `/usr/local/bin`
```
mv /tmp/wp-cli.phar /usr/local/bin/wp
```

Para poder tener WP-CLI en el directorio que queremos deberemos moverlo al directorio indicado.

## 1.3.5 Eliminamos descargas precias del codig fuente de WordPress
```
rm -rf ${WORDPRESS_DIRECTORY}/*
```

Al igual que con WP-CLI deberemos eliminar descargas previas de WordPress para evitar fallos o errores durante su instalación.

## 1.3.6 Descargamos el codigo fuente de WordPress
```
wp core download --locale=es_ES --path=/var/www/html --allow-root
```

Descargamos el codigo fuente de WordPress, especificamos que el idioma sea en Español y que esté en el directorio `/var/www/html`.

## 1.3.7 Cambiamos el propietario y el grupo al directorio `/var/www/html`
```
chown -R www-data:www-data /var/www/html
```

## 1.3.8 Creamos el archivo de configuracion de WordPress
```
wp config create \
  --dbname=$WORDPRESS_DB_NAME \
  --dbuser=$WORDPRESS_DB_USER \
  --dbpass=$WORDPRESS_DB_PASSWORD \
  --dbhost=$WORDPRESS_DB_HOST \
  --path=$WORDPRESS_DIRECTORY \
  --allow-root
```

Mediante este comando configuramos las variables que va a tomar WordPress en cuando a nombre de la base da datos, usuario y contraseña, cual va a ser el host o su directorio, además, con `--allow-root` permitimos que pueda ser ejcutado con `root`.

Todas estas variables como se ha mencionado antes de configuran en el archivo `.env`.

## 1.3.9 Instalacion de WordPress con los datos proporcionados.
```
wp core install \
  --url=$LE_DOMAIN \
  --title=$WORDPRESS_TITLE \
  --admin_user=$WORDPRESS_ADMIN_USER \
  --admin_password=$WORDPRESS_ADMIN_PASS \
  --admin_email=$WORDPRESS_ADMIN_EMAIL \
  --path=$WORDPRESS_DIRECTORY \
  --allow-root 
```

Aqui vamos a especificar como queremos que se llame la pagina, el nombre del dominio, el usuario y contraseña del admin y el directorio donde va a estar, que es `/var/www/html`.

## 1.3.10 Instalacion y activacion de un tema y plugin WordPress
```
wp theme install mindscape --activate --path=$WORDPRESS_DIRECTORY --allow-root

wp plugin install wps-hide-login --activate --path=$WORDPRESS_DIRECTORY --allow-root
```

Descargamos un tema de WordPress y lo activamos para que al acceder a la pagina nos salga con este tema, asi como el plugin `wps-hide-login`.

## 1.3.11 Confuramos el plugin, los enlaces y `.htaccess`
```
wp option update whl_page "$WORDPRESS_HIDE_LOGIN_URL" --path=$WORDPRESS_DIRECTORY --allow-root

wp rewrite structure '/%postname%/' --path=$WORDPRESS_DIRECTORY --allow-root

cp ../htaccess/.htaccess $WORDPRESS_DIRECTORY
```
El primer comando configura el plugin anteriormente instalado con una URL personalizada para el inicio de sesión. El segundo configura que los enlaces sean permanentes y el tercero opia un archivo `.htaccess` predefinido para reglas adicionales del servidor.

## 1.3.12 Cambiamos el propietario y el grupo al directorio de WordPress
```
chown -R www-data:www-data $WORDPRESS_DIRECTORY
```

Por ultimo este sería el resultado de la instalación junto con su certificado.
![](/img/Captura%20de%20pantalla%202024-12-01%20202215.png)

![](/img/Captura%20de%20pantalla%202024-12-01%20202416.png)

Si ponemos /nadaimportante despues de nuestro nombre de dominio, nos redirigirá a la pagina de inicio de sesión del admin de WordPress

![](/img/Captura%20de%20pantalla%202024-12-01%20202315.png)

Y ya habriamos concluido la instalacion y configuracion de WordPress con WP-CLI.

# 2 Maquina Backend
En esta maquina deberemos instalar MySQL Server.

## 2.1 Archivo `install_lamp_backend.sh`
En la cabecera de nuestro archivo creado pondremos lo siguiente:
```
#!/bin/bash
```
Esto indica que el script debe ejecutarse con el intérprete de Bash e irá siempre en la cabecera del archivo.
```
set -ex
```
Este comando hace que el script muestre cada comando antes de ejecutarlo `(-x)` y salga si algún comando falla `(-e)`.

## 2.1.1 Actualización del Sistema
### 2.1.1.1 Actualizar los repositorios
```
apt update
```
Actualiza la lista de paquetes disponibles en los repositorios configurados.

### 2.1.1.2 Actualizar los paquetes
```
apt upgrade -y
```
Actualiza todos los paquetes instalados a sus versiones más recientes. El `-y` automáticamente acepta todas las confirmaciones.

## 2.1.2 Instalación de MySQL
### 2.1.2.1 Instalar MySQL Server
```
apt install mysql-server -y
```
Instala el servidor de base de datos MySQL.

## 2.1.3 Configuramos el archivo /etc/mysql/mysql.conf.d/mysqld.cnf
```
sed -i "s/127.0.0.1/$BACKEND_PRIVATE_IP/" /etc/mysql/mysql.conf.d/mysqld.cnf
```
En la configuración por defecto, MySQL sólo permite conexiones desde localhost (127.0.0.1). Habrá que modificar este valor por la dirección IP de la máquina donde se está ejecutando el servicio de MySQL. En nuestro caso será la IP privada del Backend

## 2.1.4 Reiniciamos el servicio de MySQL
```
systemctl restart mysql
```
Reiniciamos el servicio de MySQL.

## 2.2 Archivo `deploy_wordpress_backend.sh`
```
mysql -u root <<< "DROP DATABASE IF EXISTS $WORDPRESS_DB_NAME"
mysql -u root <<< "CREATE DATABASE $WORDPRESS_DB_NAME"
mysql -u root <<< "DROP USER IF EXISTS $WORDPRESS_DB_USER@$FRONTEND_PRIVATE_IP"
mysql -u root <<< "CREATE USER $WORDPRESS_DB_USER@$FRONTEND_PRIVATE_IP IDENTIFIED BY '$WORDPRESS_DB_PASSWORD'"
mysql -u root <<< "GRANT ALL PRIVILEGES ON $WORDPRESS_DB_NAME.* TO $WORDPRESS_DB_USER@$FRONTEND_PRIVATE_IP"
```

Configuramos la base de datos de MySQL, donde la IP para acceder debera ser la IP de la maquina Frontend.

![](/img/Captura%20de%20pantalla%202024-12-01%20210015.png)
