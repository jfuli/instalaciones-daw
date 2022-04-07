# apache2+php+mariadb+wordPress+drupal_JorgeFernándezUlibarrenaDAW

El acrónimo **LAMP** se refiere a un entorno configurado en un servidor que nos posibilita servir aplicaciones web escritas en PHP.

Para ello usamos las siguientes tecnologías:

- Linux, sistema operativo;
- Apache o nginx, servidor web;
- MySQL, MariaDB, gestores de bases de datos;
- PHP, lenguaje de programación.

## Instalación base

Partimos de una configuración base

vagrant init bento/ubuntu-20.04

### Edita el Vagrantfile para hacer el forward del puerto 8080 al puerto 80

Vagrant**.**configure**(**"2"**)** **do** **|**config**|**

config**.**vm**.**box **=** "bento/ubuntu-20.04"

config**.**vm**.**network "forwarded_port"**,** guest**:** **80,** host**:** **8080**

**end**

Vagrant.configure("2") do |config|
config.vm.box = "bento/ubuntu-20.04"
config.vm.network "forwarded_port", guest: 80, host: 8080

# Enable provisioning with a shell script. Additional provisioners such as

# Ansible, Chef, Docker, Puppet and Salt are also available. Please see the

# documentation for more information about their specific syntax and use.

# config.vm.provision "shell", inline: &lt;<-SHELL

# apt-get update

# apt-get install -y apache2

# SHELL

end

### Levanta y actualiza el sistema

vagrant up

vagrant ssh

sudo apt update

La pass es vagrant

## Instalar todo en una línea de comandos

sudo su \
root@jorge:~# apt install mariadb-client mariadb-server php7.0 php7.0-mysql apache2 libapache2-mod-php7.0

## Instalación mariadb

Instalación del servidor de base de datos

sudo apt install mariadb-server

## Instalación Apache y PHP

Instalación de apache2 y del módulo que permite que apache2 interprete PHP (apache2 hará el papel de servidor web y de servidor de aplicaciones).

sudo apt install apache2 libapache2-mod-php php php-mysql

### Configuración de PHP

Archivos de configuración de PHP (según versión. En este caso 7.4):

- /etc/php/7.4/cli: Configuración de php para php7.4-cli (ejecución de php desde la línea de comandos)
- /etc/php/7.4/apache2: Configuración de php para apache2 usado como módulo
- /etc/php/7.4/apache2/php.ini: Configuración de php
- /etc/php/7.4/fpm: Configuración de php para php-fpm
- /etc/php/7.4/mods-available: Módulos disponibles de php

### Comprobación

Creamos un fichero info.php en el documentRoot (/var/www/html ?) con el siguiente contenido:

**&lt;?**php phpinfo**();** _?>_

[http://localhost:8080/info.php](http://localhost:8080/info.php)

## COMANDOS DE INTERÉS

vagrant logout

vagrant ssh

Practica> vagrant halt (para parar la máquina)

vagrant destroy ( para borrar)

vagrant@vagrant: \$ tail-f /var/log/apache2/ ver las peticiones de apache

**C:\Windows\System32\drivers\etc\hosts**

exit Cierra la conexión vagrant

## Instalación WordPress

### Entramos de nuevo y levantamos vagrant

cd Desktop \
cd DAW 2 \
cd DespliegueDAW \
cd Tareas \
cd server1 \
vagrant up \
vagrant ssh \
sudo su

### Accedemos a nuestra base de datos

mysql -u root -p \
Enter password: (dejamos vacío, enter)

### Ejecutamos instrucciones para dar permisos

create database wordpress; \
use wordpress; \
create user ‘user’@’localhost’;

grant all privileges on wordpress.\* to ‘user’@’localhost’ identified by ‘password’; \
flush privileges;

exit

### Descargamos wordpress

vamos al document root, directorio dónde trabaja virtualhost por defecto e instalamos wordpress (hay que abrir la pag de descargas de wordpress y copiar el enlace de descarga [https://wordpress.org/download/](https://wordpress.org/download/) ) ( [https://wordpress.org/latest.zip](https://wordpress.org/latest.zip) )

cd /var/www/html/ \

### Usamos herramienta wget para descargarlo

wget [https://wordpress.org/latest.zip](https://wordpress.org/latest.zip)

Descargamos herramienta zip para poder descomprimir la descarga

apt install unzip \
unzip latest.zip \
ls

hay que dar permisos a apache en el documentroot

cd ..

estando en : /var/www#

chown -R www-data:www-data html/

comprobamos accediendo a la web localhost/wordpress (servidor.example.org/wordpress)

Completamos la instalación en la base de datos desde la pag

## Instalación Drupal

Volvemos a la carpeta inicial Vagrant

cd ..

cd ..

cd home/vagrant

Buscamos actualizaciones del sistema, y si hay, las descargamos

sudo apt update \
sudo apt upgrade

### Descargar Drupal 9 y extraer

para Ubuntu 20.04 LTS desde la [sección de descargas del sitio oficial](https://www.drupal.org/download/)

wget --content-disposition [https://www.drupal.org/download-latest/tar.gz](https://www.drupal.org/download-latest/tar.gz) \

extraemos e instalamos en la ruta \
tar xf drupal-9.x.x.tar.gz -C /var/www

Cambiamos nombre por drupal simplemente

ln -s /var/www/drupal-9.X.X /var/www/drupal

le damos permisos

sudo chown -R www-data: /var/www/drupal/

Activamos módulos de apache

a2enmod expires headers rewrite

Hacemos aplicación navegable a través de un alias

nano /etc/apache2/sites-available/drupal.conf

Por tanto el contenido quedaría así:

Alias /drupal /var/www/drupal

&lt;Directory /var/www/drupal>

AllowOverride all

&lt;/Directory>

Guardamos los cambios y activamos la configuración:

a2ensite drupal.conf

Y reiniciamos el servicio web para aplicar todos estos ajustes:

systemctl restart apache2

### Conectamos Drupal a MySQL

Conectamos al servicio con el cliente mysql y un usuario administrador:

~\$ mysql -u root -p

password: (enter)

Creamos la base de datos:

> create database drupal9 charset utf8mb4 collate utf8mb4_unicode_ci;

creamos usuario MySQL7

create user ‘drupal9’@’localhost’;

grant all privileges on drupal9.\* to ‘drupal9’@’localhost’ identified by ‘password’;

Si usamos MariaDB o MySQL 5 creamos el usuario de forma trivial:

> create user drupal9@localhost identified by 'XXXXXXXX';

En caso de tratarse de MySQL 8 es importante asegurarse de usar el plugin de conexión adecuado:

> create user drupal9@localhost identified with mysql_native_password by 'XXXXXXXX';

Concedemos los privilegios al usuario sobre la base:

> grant all privileges on drupal9.\* to drupal9@localhost;

Y cerramos la conexión:

> exit

((Creamos un rol con contraseña para Drupal 9:

~\$ sudo -u postgres createuser -P drupal9

Y creamos la base asociándola al rol anterior:

~\$ sudo -u postgres createdb drupal9 -O drupal9))

### Instalamos las extensiones necesarias.

En el caso de trabajar con la versión nativa de PHP en Ubuntu 20.04:

~\$ sudo apt install -y php-apcu php-gd php-mbstring php-uploadprogress php-xml

Si trabajamos con alguna otra versión del repositorio alternativo, será necesario indicarla en el nombre de los paquetes; por ejemplo, si se trata de la versión 8.0:

~\$ sudo apt install -y php8.0-apcu php8.0-gd php8.0-mbstring php8.0-uploadprogress php8.0-xml

Se necesitará también la extensión que corresponda al motor de bases de datos sobre el que corra Drupal 9, aplicándose lo ya dicho para la nomenclatura de los paquetes, siendo en el caso de MariaDB/MySQL:

~\$ sudo apt install -y php-mysql

Terminada la instalación hay que recargar la configuración del servicio web:

~\$ sudo systemctl reload apache2

Instalador web

Para acceder al instalador web de Drupal 9 en Ubuntu 20.04 LTS utilizaremos la URL de acceso a la aplicación según la forma de integrarla en el servicio web, en este caso añadiendo el alias a la dirección IP o nombre DNS del servidor.

Por ejemplo, la máquina Ubuntu 20.04 que hemos usado para elaborar este artículo es accesible en el subdominio ubuntu2004.local.lan, y hemos configurado el alias /drupal, por lo que usamos http://ubuntu2004.local.lan/drupal como URL

Configuramos Drupal.

FIN
