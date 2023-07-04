## Umgebung

Die Umgebung wurde mit folgendem Vagrantfile erstellt. 

``` 
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

Die Datenbank bewirkt in dieser Konfiguration nicht viel. Der Webserver greift zwar darauf zu, ermöglich