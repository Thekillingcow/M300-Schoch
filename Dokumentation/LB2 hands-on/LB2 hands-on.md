## Umgebung

Die Umgebung wurde mit folgendem Vagrantfile erstellt. 

``` ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.define "database" do |db|
    db.vm.box = "ubuntu/xenial64"
    db.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
    end
    db.vm.hostname = "db01"
    db.vm.network "private_network", ip: "192.168.55.100"
    db.vm.provision "shell", path: "db.sh"
    db.vm.provision "shell", inline: <<-SHELL
      # Debug ON!!!
      set -o xtrace
      echo '127.0.0.1 localhost db01\n192.168.55.101 web01' > /etc/hosts

      # Configure ufw
      sudo apt-get install ufw
      sudo ufw allow ssh
      sudo ufw allow from 192.168.55.101 to port 3306
      sudo ufw --force enable


    SHELL
  end

  # Web Server
  config.vm.define "web" do |web|
    web.vm.box = "ubuntu/xenial64"
    web.vm.hostname = "web01"
    web.vm.network "private_network", ip:"192.168.55.101"
    web.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
    end
    web.vm.synced_folder ".", "/var/www/html"
    web.vm.provision "shell", inline: <<-SHELL
      # Debug ON!!!
      set -o xtrace
      sudo apt-get update
      sudo apt-get -y install apache2 mysql-client nmap
      echo '127.0.0.1 localhost web01\n192.168.55.100 db01' > /etc/hosts

      sudo groupadd myadmin
      sudo useradd admin01 -g myadmin -m -s /bin/bash
      sudo useradd admin02 -g myadmin -m -s /bin/bash
      sudo chpasswd <<<admin01:admin
      sudo chpasswd <<<admin02:admin

      # Configure ufw
      sudo apt-get install ufw
      sudo ufw allow http
      sudo ufw allow https
      sudo ufw allow ssh
      sudo ufw --force enable
    SHELL
  end

  # Proxy
  config.vm.define "proxy" do |proxy|
    proxy.vm.box = "ubuntu/xenial64"
    proxy.vm.hostname = "proxy01"
    proxy.vm.network "private_network", ip:"192.168.55.102"
    #web.vm.network "forwarded_port", guest:80, host:8080, auto_correct: true
    #proxy.vm.synced_folder ".", "/var/www/html"
    proxy.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
    end
    proxy.vm.provision "shell", inline: <<-SHELL
      # Debug ON!!!
      set -o xtrace
      sudo apt-get update
      sudo apt-get -y install apache2
      a2enmod proxy
      a2enmod proxy_http
      
      sudo tee /etc/apache2/sites-available/reverse-proxy.conf > /dev/null <<EOL
<VirtualHost *:80>
        ServerName localhost
        ProxyPass / http://192.168.55.101/
        ProxyPassReverse / http://192.168.55.101/
</VirtualHost>
EOL

# Aktiviere die Konfigurationsdatei
sudo a2ensite reverse-proxy.conf

# Starte den Apache-Server neu
sudo systemctl restart apache2
      echo '127.0.0.1 localhost proxy01\n192.168.55.100 db01' > /etc/hosts
      echo '127.0.0.1 localhost proxy01\n192.168.55.101 web01' >> /etc/hosts

      sudo groupadd myadmin
      sudo useradd admin01 -g myadmin -m -s /bin/bash
      sudo useradd admin02 -g myadmin -m -s /bin/bash
      sudo chpasswd <<<admin01:admin
      sudo chpasswd <<<admin02:admin

      # Configure ufw
      sudo apt-get install ufw
      sudo ufw allow http
      sudo ufw allow https
      sudo ufw allow ssh
      sudo ufw --force enable
    SHELL
  end
end

```


### Database

Die Datenbank bewirkt in dieser Konfiguration nicht viel. Der Webserver greift zwar darauf zu, benötigt sie jedoch nicht. Jedoch habe ich dies trotzdem installiert und füge die Logs hier an: 

### Webserver
Der Webserver ist über die IP 192.168.55.102 erreichbar, hat jedoch die IP 192.168.55.101. Dies wird über den eingerichteten Proxy erreicht. Um eine Website einzurichten ist der html Ordner mit dem Host gesynct. Auf diesem ist aktuell lediglich eine Standard HTML seite eingerichtet. Mit der Shell wird dann auf dem Server apache2 installiert und es werden 2 admins und eine admin Gruppe eingerichtet. Das Passwort für die Admins ist admin. ebenfalls wird  eine Firewall installiert, um nur noch https, http und SSH zuzulassen.
web LOG:
root@web01:~# cat /var/log/apache2/access.log
192.168.55.1 - - [04/Jul/2023:22:12:44 +0000] "GET / HTTP/1.1" 200 3525 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36"
192.168.55.1 - - [04/Jul/2023:22:12:44 +0000] "GET /icons/ubuntu-logo.png HTTP/1.1" 200 3623 "http://192.168.55.101/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36"
root@web01:~#


### Proxy
Mit dem installierten Proxy Server kann die eigentliche IP des Webservers hinter der des Proxies verborgen werden. Anfragen auf die IP des Proxies "192.168.55.102" werden auf die IP des Webservers weitergeleitet. Die einrichtung der Weiterleitung ist im Vagrantfile ersichtlich und wird in ein config File geschrieben, damit dieses nach einem Neustart einfach wieder abgerufen werden kann.  
Proxy LOG:
root@proxy01:/var/log/apache2# cat access.log
192.168.55.1 - - [04/Jul/2023:22:12:39 +0000] "GET / HTTP/1.1" 200 3525 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36"
192.168.55.1 - - [04/Jul/2023:22:12:39 +0000] "GET /icons/ubuntu-logo.png HTTP/1.1" 200 3623 "http://192.168.55.102/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36"
192.168.55.1 - - [04/Jul/2023:22:12:39 +0000] "GET /favicon.ico HTTP/1.1" 404 492 "http://192.168.55.102/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36"
root@proxy01:/var/log/apache2#
