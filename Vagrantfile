# coding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "debian/bullseye64"
  
  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.linked_clone = true
  end

  ####################################################################
  # dns.sistema.sol
  ####################################################################
  
  config.vm.define "dns" do |dns|

    dns.vm.provider "virtualbox" do |vb|
      vb.name = "http-lab-2-dns"
      vb.memory = "256"
    end

    dns.vm.hostname = "dns.sistema.sol"    
    dns.vm.network :private_network, ip: "192.168.56.100", hostname: true

    dns.vm.provision "shell", inline: <<-SHELL

      # instalar paquetes
      apt-get update
      apt-get install -y bind9 bind9-utils bind9-doc

      # copiar ficheros
      cp -v /vagrant/config/dns/named /etc/default/
      cp -v /vagrant/config/dns/named.conf.options /etc/bind
      cp -v /vagrant/config/dns/named.conf.local /etc/bind
      cp -v /vagrant/config/dns/db.* /var/lib/bind

      # reiniciar servicio
      systemctl restart bind9
      systemctl status bind9
    SHELL
    
  end

  ####################################################################
  # tierra.sistema.sol
  ####################################################################

  config.vm.define "tierra" do |tierra|
    tierra.vm.provider "virtualbox" do |vb|
      vb.name = "http-lab-2-tierra"
      vb.memory = "256"
    end

    tierra.vm.hostname = "tierra.sistema.sol"
    tierra.vm.network :private_network, ip: "192.168.56.101", hostname: true
    
    tierra.vm.provision "shell", inline: <<-SHELL
      apt-get update && apt-get upgrade -y
      apt-get install -y apache2

      cp -v /vagrant/config/tierra/apache2.conf /etc/apache2

      # TODO: Configurar sitio virtual
      cp -v /vagrant/config/tierra/discovery.sistema.sol.conf /etc/apache2/sites-available

      # Copiar los recursos de nuestra web al Document Root
      # TODO
        cp -v -r /vagrant/config/tierra/discovery.sistema.sol /var/www/

      # Habilitar/deshabilitar sitios virtuales      
      # TODO
        sudo a2dissite 000-default.conf
        sudo a2ensite discovery.sistema.sol.conf
        sudo systemctl restart apache2

      # Habilitar módulos necesarios
      # TODO
        sudo a2enmod auth_basic
        sudo a2enmod authz_groupfile
        sudo a2enmod auth_digest
        sudo systemctl restart apache2

      # Copiar ficheros de configuración de la autenticación
      # TODO
        cp -v /vagrant/.htgroups /etc/apache2/
        cp -v /vagrant/.htpasswd_basic /etc/apache2/
        cp -v /vagrant/.htpasswd_digest /etc/apache2/

      systemctl restart apache2
      systemctl status apache2
    SHELL
  end
end
