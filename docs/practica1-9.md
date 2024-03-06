# practica01.9-iaw
Este repositorio es para la práctica01.9 de Git del módulo IAW
Arquitectura de una aplicación web LAMP en dos niveles

1.	Creamos dos máquinas instancia EC2 en AWS iguales una para el frontend y otra para el backend.

2.	Le ponemos un nombre (practica01.9-frontend y practica01.9-backend respectivamente) y seleccionamos la Amazon Machine Image 

3. Creamos 2 grupos de seguridad diferentes para estas máquinas. Para la del frontend abriremos los puertos 80 (HTTP), 22 (SSH) y todos los ICMP IPv4. Mientras que para el backend abriremos los puertos 3306 (MYSQL/Aurora), 22 (SSH) y todos los ICMP IPv4.


5.	Seleccionamos el tamaño que queremos que tenga nuestra máquina, en este caso la t2.small. En par de claves seleccionaremos la de vockey y creando la instancia debemos abrir los puertos para conectarnos por SSH y poder acceder por HTTP/HTTPS.

6.  Creamos un par de claves (pública y privada) para conectar por SSH con las instancias. Nos dirigiremos a "Direcciones IP elásticas", creamos dos y las asignamos a las instancias EC2 correspondientes.

7.  Una vez asignada, iremos al laboratorio de nuestro AWS y descargaremos la key SSH en formato PEM. Renombramos el archivo a "vockey.pem" y la colocamos en una carpeta. 

8.  Nos conectamos a la máquina mediante ssh con el comando "ssh -i "vockey.pem" ubuntu@[IP publica de la máquina]".

9.  Ahora nos dirigiremos al Visual Studio Code, descargamos la extensión "Remote - SSH" para poder conectarnos a las máquinas. Con CTRL + SHIFT + P abriremos el archivo de configuración de SSH y colocamos los siguientes datos.

# Creación de la carpeta conf y el archivo 000-default.conf (Archivo de configuración)
### Configuración
~~~
ServerSignature Off
ServerTokens Prod
<VirtualHost *:80>
  #ServerName www.example.com
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/html

  <Directory "/var/www/html">
    AllowOverride All
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
~~~

# Creación de la carpeta htaccess y archivo .htaccess
~~~
# BEGIN WordPress
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
</IfModule>
# END WordPress
~~~

# Creación de la carpeta php y archivo index.php
~~~
<?php

phpinfo();

?>
~~~

# Creación de la carpeta scripts y archivo variables .env
~~~
MYSQL_PRIVATE_IP=172.31.90.21

WORDPRESS_DB_NAME=wordpress
WORDPRESS_DB_USER=alexg
WORDPRESS_DB_PASSWORD=1234
WORDPRESS_DB_HOST=172.31.90.21
IP_CLIENTE_MYSQL=172.31.88.174
TEMA=mindscape
PLUGIN=wps-hide-login
PLUGIN2=jetpack

WORDPRESS_TITLE="Sitio web de IAW"
WORDPRESS_ADMIN_USER=admin
WORDPRESS_ADMIN_PASS=admin
WORDPRESS_ADMIN_EMAIL=demo@demo.es

CB_MAIL=alexg@iaw.com
CB_DOMAIN=practica9.hopto.org
~~~


# Creación del install_lamp_frontend.sh
## Muestra todos los comandos que se van ejecutando
set -ex

## Actualizamos los repositorios
apt update

## Actualizamos los paquetes
apt upgrade -y

## Instalamos el servidor web Apache
apt install apache2 -y

## Instalamos PHP
apt install php libapache2-mod-php php-mysql -y

##  Copiamos le archivo conf de apache
cp ../conf/000-default.conf /etc/apache2/sites-available

## Habilitamos el módulo rewrite
a2enmod rewrite

## Reiniciamos el servicio de Apache
systemctl restart apache2

## Copiamos el archivo .php
cp ../php/index.php /var/www/html


# Creación del install_lamp_backend.sh
## Muestra todos los comandos que se van ejecutando
set -ex

## Añadimos .env
source .env

## Actualizamos los repositorios
apt update

## Actualizamos los paquetes
apt upgrade -y

## Instalamos el gestor de bases de datos MySQL
apt install mysql-server -y

## Configuramos MySQL para que solo acepte conexiones desde la IP privada
sed -i "s/127.0.0.1/$MYSQL_PRIVATE_IP/" /etc/mysql/mysql.conf.d/mysqld.cnf

## Reiniciamos el servicio de MySQL
systemctl restart mysql


# Creación del setup_letsencrypt_certificate.sh
## Muestra todos los comandos que se van ejecutando
set -ex

## Actualizamos los repositorios
apt update

## Actualizamos los paquetes
apt upgrade -y

## Importamos el archivo de variables .env
source .env

## Instalamos y actualizamos snapd
snap install core && snap refresh core

## Eliminamos cualquier instalación previa de certbot con apt
apt remove certbot

## Instalamos la aplicación certbot
snap install --classic certbot

## Creamos un alias para la aplicación certbot
ln -fs /snap/bin/certbot /usr/bin/certbot

## Ejecutamos el comando certbot
certbot --apache -m $CB_MAIL --agree-tos --no-eff-email -d $CB_DOMAIN --non-interactive


# Creación del deploy_frontend.sh
## Muestra todos los comandos que se van ejecutando
set -ex

## Actualizamos los repositorios
apt update

## Actualizamos los paquetes
apt upgrade -y

## Importamos el archivo de variables .env
source .env

## Eliminamos descargas previas de wp-cli
rm -rf /tmp/wp-cli.phar

## Descargamos la utilidad wp-cli
wget https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar -P /tmp

## Damos permisos de ejecución al archivo wp-cli.phar
chmod +x /tmp/wp-cli.phar

## Movemos la utilidad wp-cli al directorio /usr/local/bin
mv /tmp/wp-cli.phar /usr/local/bin/wp

## Eliminamos instalaciones previas de WordPress
rm -rf /var/www/html/*

## Descargamos el código fuente ed WordPress en /var/www/html
wp core download --locale=es_ES --path=/var/www/html --allow-root

## Creamos el archivo wp-config
wp config create \
  --dbname=$WORDPRESS_DB_NAME \
  --dbuser=$WORDPRESS_DB_USER \
  --dbpass=$WORDPRESS_DB_PASSWORD \
  --dbhost=$WORDPRESS_DB_HOST \
  --path=/var/www/html \
  --allow-root

# Instalamos WordPress
wp core install \
  --url=$CB_DOMAIN \
  --title="$WORDPRESS_DB_TITLE" \
  --admin_user=$WORDPRESS_ADMIN_USER \
  --admin_password=$WORDPRESS_ADMIN_PASS \
  --admin_email=$WORDPRESS_ADMIN_EMAIL \
  --path=/var/www/html \
  --allow-root

## Actualizamos los plugins
wp core update --path=/var/www/html --allow-root

## Actualizamos los temas
wp theme update --all --path=/var/www/html --allow-root

## Instalamos un tema
wp theme install $TEMA --activate --path=/var/www/html --allow-root

## Actualizamos los plugins
wp plugin update --all --path=/var/www/html --allow-root

## Instalamos los plugins
wp plugin install $PLUGIN --activate --path=/var/www/html --allow-root
wp plugin install $PLUGIN2 --activate --path=/var/www/html --allow-root

## Cambiamos la estructura de las url
wp rewrite structure '/%postname%/' --path=/var/www/html --allow-root

## Cambiamos el /login por lo que nosotros queramos del plugin hide-login
wp option update whl_page 'acceso' --path=/var/www/html --allow-root

## Copiamos el nuevo archivo .htaccess en /var/www/html
cp ../htaccess/.htaccess /var/www/html

## Cambiamos los permisos
chown -R www-data:www-data /var/www/html
chown -R www-data:www-data /var/www/html