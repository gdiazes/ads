
# 📘 Guía de Laboratorio: Instalación y Configuración de openDCIM
**Entorno validado:** Ubuntu Server 20.04 (Focal Fossa) | **Versión openDCIM:** 21.01 | **PHP:** 7.4

### 📝 Notas del Entorno (Troubleshooting aplicado)
*   *Se utiliza la versión de lanzamiento **21.01** de openDCIM en lugar de clonar el repositorio de GitHub (rama de desarrollo), ya que esta última exige PHP 8.0+, lo cual puede causar conflictos en entornos académicos con restricciones de red para añadir repositorios externos (PPA).*
*   *La versión 21.01 es totalmente estable y nativamente compatible con el PHP 7.4 que trae Ubuntu 20.04 por defecto.*

---

## FASE 1: Preparación del Sistema y Dependencias

**1. Actualizar los repositorios locales:**
```bash
sudo apt update && sudo apt upgrade -y
```

**2. Instalar el Stack LAMP y módulos de PHP necesarios:**
*Nota: `apache2-utils` es necesario para el comando htpasswd.*
```bash
sudo apt install -y apache2 mariadb-server wget tar apache2-utils \
php7.4 php7.4-mysql php7.4-snmp php7.4-gd php7.4-curl \
php7.4-mbstring php7.4-xml php7.4-ldap libapache2-mod-php7.4
```

---

## FASE 2: Configuración de la Base de Datos

**1. Ingresar al motor de base de datos MariaDB:**
```bash
sudo mysql -u root
```

**2. Crear la base de datos, el usuario y asignar permisos:**
*(Ejecuta estas líneas una por una dentro del prompt de MySQL `MariaDB [(none)]> `)*
```sql
CREATE DATABASE dcim;
CREATE USER 'dcim'@'localhost' IDENTIFIED BY 'dcim';
GRANT ALL ON dcim.* TO 'dcim'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## FASE 3: Descarga y Preparación de openDCIM

**1. Descargar la versión compatible (21.01) desde el repositorio oficial web:**
```bash
cd /tmp
wget https://opendcim.org/packages/openDCIM-21.01.tar.gz
```

**2. Crear el directorio y descomprimir el sistema:**
```bash
sudo mkdir -p /var/www/html/opendcim
sudo tar -zxvf openDCIM-21.01.tar.gz -C /var/www/html/opendcim --strip-components=1
```

**3. Configurar el archivo de conexión a la Base de Datos:**
```bash
cd /var/www/html/opendcim
sudo cp db.inc.php-dist db.inc.php
```

---

## FASE 4: Corrección de Permisos de Directorio
*Este paso previene el error de permisos denegados (rojo) en las carpetas de `assets` y fuentes de PDF durante la instalación.*

**1. Asegurar la creación de las subcarpetas requeridas:**
```bash
sudo mkdir -p /var/www/html/opendcim/assets/{drawings,pictures,reports}
```

**2. Asignar propiedad a Apache (`www-data`) y otorgar permisos de escritura (775):**
```bash
sudo chown -R www-data:www-data /var/www/html/opendcim
sudo chmod -R 775 /var/www/html/opendcim/assets/
sudo chmod -R 775 /var/www/html/opendcim/vendor/mpdf/mpdf/ttfontdata
```

---

## FASE 5: Seguridad y Configuración de Apache
*openDCIM no tiene login propio; delega la seguridad en la Autenticación Básica de Apache.*

**1. Crear el usuario de acceso (Administrador):**
*(Se solicitará ingresar y confirmar una contraseña)*
```bash
sudo htpasswd -c /etc/apache2/opendcim.htpasswd admin
```

**2. Habilitar los módulos de Apache requeridos:**
```bash
sudo a2enmod rewrite authn_file authz_user auth_basic
```

**3. Configurar el VirtualHost o Directorio de Apache:**
Edita el archivo de configuración por defecto:
```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```
Agrega el siguiente bloque `<Directory>` **dentro** de la etiqueta `<VirtualHost *:80>`:
```apache
    <Directory "/var/www/html/opendcim">
        Options Indexes FollowSymLinks
        AllowOverride All
        AuthType Basic
        AuthName "openDCIM Login"
        AuthUserFile /etc/apache2/opendcim.htpasswd
        Require valid-user
    </Directory>
```
*(Guarda con `Ctrl+O`, `Enter`, y sal con `Ctrl+X`)*

**4. Reiniciar Apache para aplicar todos los cambios:**
```bash
sudo systemctl restart apache2
```

---

## FASE 6: Instalación vía Interfaz Web

1. Abre un navegador web en una máquina que tenga alcance al servidor.
2. Ingresa a la URL: **`http://<IP_DEL_SERVIDOR>/opendcim`**
3. El navegador te pedirá credenciales. Ingresa el usuario **`admin`** y la contraseña que creaste en el paso de `htpasswd`.
4. Aparecerá la pantalla de pre-vuelo (Pre-flight check). Verifica que todas las dependencias y permisos (assets) estén con un **"Yes" en color verde**.
5. Desplázate hacia el final de la página y haz clic en el enlace para iniciar la creación automática de tablas en la base de datos (Upgrade/Install Database).
6. ¡Listo! Accederás al panel de control principal de openDCIM.

--- 
**¡Fin del Laboratorio!** 🚀
