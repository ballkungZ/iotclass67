# Install Server and Docker


## How to install Server
1. Download Ubuntu Server ISO
2. Create a Bootable USB Drive
3. Insert the USB drive to เครื่อง server
4. Boot from the USB Drive
5. Install Ubuntu Server

    -Boot into the installer

    -Select the language

    -Network configuration

    -Configure storage

    -Set up the user account

    -SSH setup
    
6.  Finish the installation


## How to install Docker
First, update your existing list of packages:
sudo apt update

Next, install a few prerequisite packages which let apt use packages over HTTPS:
sudo apt install apt-transport-https ca-certificates curl software-properties-common

Then add the GPG key for the official Docker repository to your system:
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

Add the Docker repository to APT sources:
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"

Make sure you are about to install from the Docker repo instead of the default Ubuntu repo:
apt-cache policy docker-ce

Finally, install Docker:
sudo apt install docker-ce

Docker should now be installed, the daemon started, and the process enabled to start on boot. Check that it’s running:
sudo systemctl status docker



