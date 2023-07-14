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

