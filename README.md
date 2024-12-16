# Laboratorio HTTP: Configuración de Servidores Virtuales y Autenticación

Este laboratorio detalla los pasos para configurar servidores virtuales en Apache con autenticación HTTP Basic y Digest.

## Requisitos previos
- **Máquinas virtuales**: 
  - `dns.sistema.sol` (IP: 192.168.56.100): Servidor DNS.
  - `tierra.sistema.sol` (IP: 192.168.56.101): Servidor Web.
- **Red**: 192.168.56.0/24.
- **Software**: Apache2.

---

## Configuración inicial

1. Configurar el servidor DNS:
   - Establece autoridad sobre el dominio `sistema.sol`.
   - Resuelve `discovery.sistema.sol` como alias de `tierra.sistema.sol`.

2. Prueba la resolución DNS:
   ```bash
   nslookup discovery.sistema.sol
   dig discovery.sistema.sol

Creación del servidor virtual

    Crea la raíz de documentos:

sudo mkdir -p /var/www/discovery.sistema.sol/{basic/{ventas,desarrollo},digest}
echo "Welcome to Discovery" | sudo tee /var/www/discovery.sistema.sol/index.html
echo "Basic Auth - Ventas" | sudo tee /var/www/discovery.sistema.sol/basic/ventas/index.html
echo "Basic Auth - Desarrollo" | sudo tee /var/www/discovery.sistema.sol/basic/desarrollo/index.html
echo "Digest Auth - Hello" | sudo tee /var/www/discovery.sistema.sol/digest/hello.html

Crea el archivo de configuración del servidor virtual:

sudo nano /etc/apache2/sites-available/discovery.sistema.sol.conf

Contenido:

<VirtualHost *:80>

    ServerName discovery.sistema.sol
    ServerAdmin webmaster@apolo.sistema.sol

    DocumentRoot /var/www/discovery.sistema.sol

    # Configuración de logs
    ErrorLog ${APACHE_LOG_DIR}/discovery_error.log
    CustomLog ${APACHE_LOG_DIR}/discovery_access.log combined

    # Configuración del directorio principal
    <Directory /var/www/discovery.sistema.sol>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    # Configuración del directorio basic
    <Directory /var/www/discovery.sistema.sol/basic>
        AuthType Basic
        AuthName "Restricted Area"
        AuthUserFile /etc/apache2/.htpasswd_basic
        Require valid-user
        Options Indexes FollowSymLinks
        AllowOverride None
    </Directory>

    # Configuración del subdirectorio desarrollo
    <Directory /var/www/discovery.sistema.sol/basic/desarrollo>
        AuthType Basic
        AuthName "Restricted Area - Desarrollo"
        AuthUserFile /etc/apache2/.htpasswd_basic
        AuthGroupFile /etc/apache2/.htgroups
        Require group desarrollo
        Options Indexes FollowSymLinks
        AllowOverride None
    </Directory>

    # Configuración del subdirectorio ventas
    <Directory /var/www/discovery.sistema.sol/basic/ventas>
        AuthType Basic
        AuthName "Restricted Area - Ventas"
        AuthUserFile /etc/apache2/.htpasswd_basic
        AuthGroupFile /etc/apache2/.htgroups
        Require group ventas
        Options Indexes FollowSymLinks
        AllowOverride None
    </Directory>

    # Configuración del directorio digest
    <Directory /var/www/discovery.sistema.sol/digest>
        AuthType Digest
        AuthName "astronauts"
        AuthUserFile /etc/apache2/.htpasswd_digest
        Require user commander
        Options Indexes FollowSymLinks
        AllowOverride None
    </Directory>
</VirtualHost>


Activa el sitio y reinicia Apache:

    sudo a2ensite discovery.sistema.sol.conf
    sudo systemctl restart apache2

Configuración de autenticación HTTP Basic

Crea el archivo de contraseñas:

sudo htpasswd -c /etc/apache2/.htpasswd_basic arturo
sudo htpasswd /etc/apache2/.htpasswd_basic ana
sudo htpasswd /etc/apache2/.htpasswd_basic maria

Crea el archivo de grupos:

    echo -e "ventas:arturo\n" | sudo tee /etc/apache2/.htgroups
    echo -e "desarrollo:ana\n" | sudo tee -a /etc/apache2/.htgroups

Configuración de autenticación HTTP Digest

Habilita el módulo Digest:

    sudo a2enmod auth_digest

Crea el archivo de contraseñas Digest:

    sudo htdigest -c /etc/apache2/.htpasswd_digest astronauts commander

Verificación

    Accede a las siguientes URL en tu navegador:
        http://discovery.sistema.sol/
        http://discovery.sistema.sol/basic
        http://discovery.sistema.sol/basic/desarrollo
        http://discovery.sistema.sol/basic/ventas
        http://discovery.sistema.sol/digest

Recuerda cambiar en tu ordenador el resolv.conf:

    sudo nano /etc/resolv.conf

Escribe ahí la ip del dns:

    192.168.56.100

## Si quieres activar el https con certificado autofirmado sigue los siguientes pasos.

Crea el certificado y la clave:

    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/discovery-sol.key \
    -out /etc/ssl/certs/discovery-sol.crt

crearemos el documento discovery.ssl.conf:

<VirtualHost *:443>

    ServerName discovery.sistema.sol
    ServerAdmin webmaster@apolo.sistema.sol

    DocumentRoot /var/www/discovery.sistema.sol
    # Configuración SSL
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/discovery-sol.crt
    SSLCertificateKeyFile /etc/ssl/private/discovery-sol.key
    
    # Configuración de logs
    ErrorLog ${APACHE_LOG_DIR}/discovery_error.log
    CustomLog ${APACHE_LOG_DIR}/discovery_access.log combined

    # Configuración del directorio principal
    <Directory /var/www/discovery.sistema.sol>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    # Configuración del directorio basic
    <Directory /var/www/discovery.sistema.sol/basic>
        AuthType Basic
        AuthName "Restricted Area"
        AuthUserFile /etc/apache2/.htpasswd_basic
        Require valid-user
        Options Indexes FollowSymLinks
        AllowOverride None
    </Directory>

    # Configuración del subdirectorio desarrollo
    <Directory /var/www/discovery.sistema.sol/basic/desarrollo>
        AuthType Basic
        AuthName "Restricted Area - Desarrollo"
        AuthUserFile /etc/apache2/.htpasswd_basic
        AuthGroupFile /etc/apache2/.htgroups
        Require group desarrollo
        Options Indexes FollowSymLinks
        AllowOverride None
    </Directory>

    # Configuración del subdirectorio ventas
    <Directory /var/www/discovery.sistema.sol/basic/ventas>
        AuthType Basic
        AuthName "Restricted Area - Ventas"
        AuthUserFile /etc/apache2/.htpasswd_basic
        AuthGroupFile /etc/apache2/.htgroups
        Require group ventas
        Options Indexes FollowSymLinks
        AllowOverride None
    </Directory>

    # Configuración del directorio digest
    <Directory /var/www/discovery.sistema.sol/digest>
        AuthType Digest
        AuthName "astronauts"
        AuthUserFile /etc/apache2/.htpasswd_digest
        Require user commander
        Options Indexes FollowSymLinks
        AllowOverride None
    </Directory>
</VirtualHost>

y luego copiaremos el certificado y la clave en nuestro equipo 

    cd /etc/ssl/certs

    cp discovery-sol.crt /vagrant/ssl

    sudo su

    cd /etc/ssl/private

    cp discovery-sol.key /vagrant/ssl


en el vagrant file copiaremos el certificado y la clave en el de nuevo en la máquina tierra, y añadiremos las siguientes lineas.

    # Copiando los certificados y clave
    cp -v /vagrant/ssl/discovery-sol.crt /etc/ssl/certs
    sudo cp -v /vagrant/ssl/discovery-sol.key /etc/ssl/private

    # TODO: Configurar sitio virtual
    cp -v /vagrant/config/tierra/discovery.ssl.conf /etc/apache2/sites-available

    # Habilitar/deshabilitar sitios virtuales      
    sudo a2ensite discovery.ssl.conf

    # Habilitar módulos necesarios
    sudo a2enmod ssl

reiniciamos lás máquinas con:

    vagrant destroy -f && vagrant up

y al entrar a una de las siguientes páginas se vería el https encima:

        https://discovery.sistema.sol/
        https://discovery.sistema.sol/basic
        https://discovery.sistema.sol/basic/desarrollo
        https://discovery.sistema.sol/basic/ventas
        https://discovery.sistema.sol/digest
