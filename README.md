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
    DocumentRoot /var/www/discovery.sistema.sol

    <Directory /var/www/discovery.sistema.sol/basic>
        AuthType Basic
        AuthName "Basic Auth"
        AuthUserFile "/etc/apache2/.htpasswd_basic"
        Require valid-user
    </Directory>

    <Directory /var/www/discovery.sistema.sol/basic/desarrollo>
        Require group desarrollo
        AuthGroupFile "/etc/apache2/.htgroups"
    </Directory>

    <Directory /var/www/discovery.sistema.sol/basic/ventas>
        Require group ventas
        AuthGroupFile "/etc/apache2/.htgroups"
    </Directory>

    <Directory /var/www/discovery.sistema.sol/digest>
        AuthType Digest
        AuthName "astronauts"
        AuthUserFile "/etc/apache2/.htpasswd_digest"
        Require user commander
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
Remember to change in your resolv.conf the namerserver.
sudo nano /etc/resolv.conf
and write: namerserver 192.168.56.100