# Proyecto: Infraestructura en Dos Niveles con Vagrant y Debian 13

## Descripción General

Este proyecto despliega una infraestructura de dos niveles para una aplicación de gestión de usuarios usando Vagrant, con dos máquinas virtuales sobre Debian 13:

- **JaimeIglesiasApache**: servidor web Apache + PHP, con acceso a Internet por NAT.
- **JaimeIglesiasDB**: servidor MariaDB, sin acceso a Internet, solo red privada.

La aplicación se accede mediante port forwarding desde el host al servidor web.

## Estructura del Proyecto
```
├── README.md
├── Vagrantfile // Archivo de configuracion de las maquinas con el script de despliegue integrado
└── screenshots/
    ├── screencast.mp4    //Video del funcionamiento de la web
    └── app_functioning.png // Captura de funcionamiento de la web
``
## Configuración Vagrantfile

- Dos máquinas declaradas con `config.vm.define` llamadas `web` y `db`.
- Máquina `web` usa box `bento/debian-13`, con hostname `JaimeIglesiasApache`.
- Configurada con IP privada `192.168.56.100` y forwarded port 8080 (host) a 80 (guest).
- Máquina `db` usa misma box, hostname `JaimeIglesiasDB`.
- IP privada `192.168.56.101`, aislada de Internet.
- Scripts inline de provisionamiento con Bash para instalación, configuración, clonación repositorio y ajustes.
- Servidor web instala Apache, PHP, git; clona repositorio de aplicación y configura Apache.
- Servidor DB instala MariaDB, configura bind-address para accesos remotos y crea usuario y base de datos.

## Scripts de Provisionamiento

### Servidor Web (Apache + PHP)
apt-get update
apt-get install -y apache2 php libapache2-mod-php php-mysql git

git clone https://github.com/josejuansanchez/iaw-practica-lamp.git /var/www/html/iaw-practica-lamp
chown -R www-data:www-data /var/www/html/iaw-practica-lamp

cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/usuariosapp.conf
sed -i 's/DocumentRoot .*/DocumentRoot /var/www/html/iaw-practica-lamp/src/' /etc/apache2/sites-available/usuariosapp.conf
a2enmod rewrite
a2ensite usuariosapp.conf
a2dissite 000-default.conf
systemctl restart apache2

sed -i "s|'localhost'|'192.168.56.101'|g; s|'database_name_here'|'iawdb'|g; s|'username_here'|'iawuser'|g; s|'password_here'|'iawpass'|g" /var/www/html/iaw-practica-lamp/src/config.php

### Servidor Base de Datos (MariaDB)
apt-get update
apt-get upgrade -y
apt-get install -y mariadb-server git

git clone https://github.com/josejuansanchez/iaw-practica-lamp.git
sed -i "s|bind-address\s*=.*|bind-address = 0.0.0.0|g" /etc/mysql/mariadb.conf.d/50-server.cnf
systemctl restart mariadb

mysql -u root -e "
CREATE DATABASE IF NOT EXISTS iawdb;
CREATE USER IF NOT EXISTS 'iawuser'@'192.168.56.%' IDENTIFIED BY 'iawpass';
GRANT SELECT, INSERT, DELETE, UPDATE ON iawdb.* TO 'iawuser'@'192.168.56.%';
FLUSH PRIVILEGES;
"
mysql -u root iawdb < iaw-practica-lamp/db/database.sql

## Uso

- Ejecutar `vagrant up` para levantar las dos máquinas.
- Acceder a la aplicación en el navegador con `http://localhost:8080`.
- Validar que ambos servidores estén activos con `vagrant status` y que la aplicación funcione correctamente.

## Evidencias

Se incluye la carpeta `screenshots` con imágenes que muestran:

- Servidor Apache en ejecución.
- Servidor MariaDB activo con base creada.
- Aplicación web funcionando correctamente.

También se incluirá un screencast mostrando la navegación y funcionamiento de la aplicación desplegada.

## URL del Repositorio

El código completo, scripts y documentación están disponibles en:  
`https://github.com/jiglesiasr05/PilaLAMP/`

---

Este documento explica detalladamente tanto la infraestructura como el aprovisionamiento y configuración para cumplir con los objetivos de la práctica.


