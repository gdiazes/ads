

### 1. Actualización del sistema e instalación de dependencias
openDCIM requiere Apache, MySQL (o MariaDB) y PHP con módulos específicos (SNMP es crítico).

```bash
# Actualizar repositorios
sudo apt update && sudo apt upgrade -y

# Instalar LAMP y dependencias de openDCIM
sudo apt install -y apache2 mariadb-server php php-mysql php-snmp php-gd php-curl php-mbstring php-xml php-ldap git
```

### 2. Configuración de la Base de Datos
Ejecutaremos los comandos que sugiere el manual, pero dentro del prompt de MariaDB.

```bash
# Entrar a MariaDB
sudo mysql -u root

# Dentro de MariaDB (copia y pega estas líneas):
CREATE DATABASE dcim;
CREATE USER 'dcim'@'localhost' IDENTIFIED BY 'dcim';
GRANT ALL ON dcim.* TO 'dcim'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 3. Descarga y Preparación de openDCIM
Lo instalaremos en `/var/www/html/opendcim` (estándar de Ubuntu).

```bash
# Clonar el repositorio oficial
cd /var/www/html
sudo git clone https://github.com/samilliken/openDCIM.git opendcim

# Entrar al directorio y configurar el archivo de base de datos
cd /var/www/html/opendcim
sudo cp db.inc.php-dist db.inc.php

# Ajustar permisos para que Apache pueda escribir
sudo chown -R www-data:www-data /var/www/html/opendcim
sudo chmod -R 755 /var/www/html/opendcim
```

### 4. Configuración de Autenticación de Apache
openDCIM delega la seguridad en el servidor web. Crearemos el usuario inicial para entrar.

```bash
# Crear archivo de contraseñas (el usuario será 'admin')
# Te pedirá una contraseña, elígela bien.
sudo htpasswd -c /etc/apache2/opendcim.htpasswd admin

# Habilitar módulos necesarios de Apache
sudo a2enmod rewrite authn_file authz_user auth_basic
```

### 5. Configuración del VirtualHost
Debemos configurar Apache para que reconozca el directorio y permita el uso de `.htaccess`.

```bash
# Crear el archivo de configuración
sudo nano /etc/apache2/sites-available/opendcim.conf
```

**Pega este contenido en el editor:**
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/opendcim

    <Directory "/var/www/html/opendcim">
        Options Indexes FollowSymLinks
        AllowOverride All
        AuthType Basic
        AuthName "openDCIM Login"
        AuthUserFile /etc/apache2/opendcim.htpasswd
        Require valid-user
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/opendcim_error.log
    CustomLog ${APACHE_LOG_DIR}/opendcim_access.log combined
</VirtualHost>
```

### 6. Activación y Reinicio
```bash
# Desactivar el sitio por defecto y activar el nuevo
sudo a2dissite 000-default.conf
sudo a2ensite opendcim.conf

# Reiniciar Apache para aplicar cambios
sudo systemctl restart apache2
```

### 7. Paso Final: Instalación vía Web
Ahora abre tu navegador y dirígete a la IP de tu servidor:
`http://tu-ip-servidor/`

1. Te pedirá el usuario y contraseña que creaste con `htpasswd` (**admin**).
2. El sistema detectará que la base de datos está vacía y te guiará por el proceso de instalación automática (haciendo click en los enlaces que aparezcan en pantalla para crear las tablas).

---

**Notas de SysAdmin:**
* **PHP SNMP:** En Ubuntu, al instalar `php-snmp`, el módulo se habilita automáticamente. No suele ser necesario editar el `php.ini` manualmente a menos que tengas varias versiones de PHP.
* **Seguridad:** Si esto va a producción, te recomiendo instalar un certificado SSL con Certbot (`sudo apt install python3-certbot-apache`).
* **db.inc.php:** Si decidiste cambiar la contraseña de la base de datos en el paso 2, asegúrate de editar `sudo nano /var/www/html/opendcim/db.inc.php` para que coincida.
