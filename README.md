# TSDJetsonNano
The following are instructions on how to run The Spaghetti Detective, a machine learning AI program that detects failing prints, on a Jetson Nano.
https://github.com/TheSpaghettiDetective/TheSpaghettiDetective 

The instructions on the official GitHub for doing this are very lacking, and a lot of the commands don't work properly. (docker-compose for example is a MASSIVE pain as it's not native to ARM64, and there are a decent amount of missing dependencies)

I've made this guide as easy as possible, so some things are dumbed down.
This is taking into consideration that you are going to run this hooked into a spare Ethernet port on your router/switch, are setting up from a Windows environment, and of course already have the Octoprint plugin installed.



Parts/Software list:
- -Jetson Nano https://developer.nvidia.com/buy-jetson
- -Power supply (this is what I used) https://www.amazon.com/gp/product/B01N4HYWAM
- -Micro USB cable
- -MicroSD Card (I'd recommend 32GB or larger)
- -MicroSD Adapter
- -Putty https://www.chiark.greenend.org.uk/~sgtatham/putty/
- -WinSCP https://winscp.net/eng/index.php
- -Etcher  https://www.balena.io/etcher/
- -Modified docker-compose.yml file (located in this git)
- -The latest Jetson Nano SD Card Image  https://developer.nvidia.com/embedded/downloads
- Optional: A case. You can either 3D print one https://www.thingiverse.com/thing:3518410 or purchase one off of Amazon

Before you begin make sure that the jumper on the board is set up to accept the power from the PSU, not from USB. My jumper came already in place but just needed flipped upside-down.


Good? Good! Let's start.

- Flash the Jetson's SD card image using Etcher
- Put the micro SD back into the Jetson Nano. Plug in your ethernet cable, usb cable, and power cable. I'd make sure the power cable was plugged in last just to be safe.
- On your PC go to Device Management, then to the Com Ports drop box. You should soon see a port appear if it hadn't already. This is your Jetson's serial port
- Open up Putty and to connect through serial
- Change the COM port to the number found in device manager (COM6 for example), change the baud rate to 115200 then click Open

You're now connected through serial port directly to the Nano!

- Go through the initial setup. Tab/enter move around. Choose whatever username/password you'd like. When you get to the network configuration page, tab down and choose eth0
- The Jetson will reboot and the serial connection will drop. You can now unplug the Jetson from USB and close the Putty connection
- Find the IP address of your Jetson from your router, open back up Putty and go to that address. Login using whatever username/password you chose during the initial setup.


Alright, awesome! Now we have access to a SSH command line!

- First thing's first, lets get everything updated

sudo apt-get update -y && sudo apt-get upgrade -y
- After all that is finished, we need to install some dependencies.

sudo apt-get install -y curl 

sudo apt-get install -y python-pip

sudo apt-get install -y python3-pip

sudo apt-get install -y libffi-dev

sudo apt-get install -y python-openssl


Now to install Docker-Compose. Since normal install methods are broken as it's not built correctly for ARM64, we will use a precompiled fork.


- Download the forked docker-compose

wget https://github.com/nefilim/docker-compose-aarch64/releases/download/1.25.4/docker-compose-Linux-aarch64
- Correctly name it and move it to bin folder

sudo mv docker-compose-Linux-aarch64 /usr/local/bin/docker-compose
- And give it right permissions to be ran

sudo chmod +x /usr/local/bin/docker-compose

- Then clone the GIT of TheSpaghettiDetective

git clone https://github.com/TheSpaghettiDetective/TheSpaghettiDetective.git

You will now need to edit the docker-compose.yml file to include the edits from https://github.com/TheSpaghettiDetective/TheSpaghettiDetective/blob/master/docs/jetson_guide.md  as the docker-compose.override.yml file will not work for some reason. If you feel comfortable doing this on your own you can use your favorite text editor (like nano)
For simplicity sake this is why I've included WinSCP as a download, and a preconfigured docker-compose.yml file

- While leaving Putty open in the background, open up WinSCP
- Put in your Nano's IP address, username and password then click on Login
- Go to the TheSpaghettiDetective folder and move the preconfigured docker-compose.yml file into it, making sure to overwrite  current file.
- You can now exit WinSCP


- Back in Putty run the docker-compose file and let it run. This  part will take the longest (15+ minutes)

cd TheSpaghettiDetective && sudo docker-compose up -d

- After you get back to your normal command line, we need to set docker to run at boot with the command

sudo systemctl enable docker
- And then reboot

sudo reboot

And that's it! After giving the Jetson a good minute or two to reboot, you can now follow the instructions on TSD's github starting at the Basic Server Configuration section.
https://github.com/TheSpaghettiDetective/TheSpaghettiDetective#basic-server-configuration

Enjoy!
