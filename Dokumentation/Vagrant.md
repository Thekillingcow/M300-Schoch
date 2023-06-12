Die Installation von Vagrant erfolgte über das Setup GUI. Nach einem Reboot war dies Problemlos installiert. 

*Robin@DESKTOP-UH27VDK MINGW64 ~/Documents/TBZ/M300/VagrantVM*

Unter diesem Verzeichnis habe ich das File mit dem folgendem Command erstellt

`$ vagrant init ubuntu/xenial64`


"A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant."

Dieses Enviroment habe ich dann mit diesem Befehl gestartet: 

`vagrant up --provider virtualbox`

Anschliessend fing der Box download an. Dieser ging über das Schul Netzwerk wei gewohnt: sehr lange. 

Folgend ein paar nützliche Commands in Vagrant: 

`Vagrant status`
`Vagrant init`
`Vagrant up`
`Vagrant suspend`
`Vagrant resume`
`Vagrant destroy`


## Testumgebung

Die Testumgebung wurde mit folgendem Vagrantfile erstellt. 

``` bash 
agrant.configure("2") do |config|

  # The most common configuration options are documented and commented below.

  # For a complete reference, please see the online documentation at

  # https://docs.vagrantup.com.

  

  # Every Vagrant development environment requires a box. You can search for

  # boxes at https://vagrantcloud.com/search.

  config.vm.box = "ubuntu/xenial64"

  config.vm.network "forwarded_port", guest:80, host:8080, auto_correct: true

  config.vm.synced_folder ".", "/var/www/html"

  

  # Disable automatic box update checking. If you disable this, then

  # boxes will only be checked for updates when the user runs

  # `vagrant box outdated`. This is not recommended.

  # config.vm.box_check_update = false

  

  # Create a forwarded port mapping which allows access to a specific port

  # within the machine from a port on the host machine. In the example below,

  # accessing "localhost:8080" will access port 80 on the guest machine.

  # NOTE: This will enable public access to the opened port

  # config.vm.network "forwarded_port", guest: 80, host: 8080

  

  # Create a forwarded port mapping which allows access to a specific port

  # within the machine from a port on the host machine and only allow access

  # via 127.0.0.1 to disable public access

  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  

  # Create a private network, which allows host-only access to the machine

  # using a specific IP.

  # config.vm.network "private_network", ip: "192.168.33.10"

  

  # Create a public network, which generally matched to bridged network.

  # Bridged networks make the machine appear as another physical device on

  # your network.

  # config.vm.network "public_network"

  

  # Share an additional folder to the guest VM. The first argument is

  # the path on the host to the actual folder. The second argument is

  # the path on the guest to mount the folder. And the optional third

  # argument is a set of non-required options.

  # config.vm.synced_folder "../data", "/vagrant_data"

  

  # Provider-specific configuration so you can fine-tune various

  # backing providers for Vagrant. These expose provider-specific options.

  # Example for VirtualBox:

  #

  config.vm.provider "virtualbox" do |vb|

  #   # Display the VirtualBox GUI when booting the machine

  #   vb.gui = true

  #

  #   # Customize the amount of memory on the VM:

    vb.memory = "1024"

   end

  #

  # View the documentation for the provider you are using for more

  # information on available options.

  

  # Enable provisioning with a shell script. Additional provisioners such as

  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the

  # documentation for more information about their specific syntax and use.

   config.vm.provision "shell", inline: <<-SHELL

     apt-get update

     apt-get install -y apache2

     apt-get install -y webalizer

   SHELL

end
```

Um den Webserver zum laufen zu bringen, musste ich natürlich noch den Port 80 auf den Host Port 8080 weiterleiten. Somit ist die Seite dann über localhost:8080 verfügbar und kann aufgerufen werden. Natülich müsste dann auch noch ein Index file hinzugefügt werden. 