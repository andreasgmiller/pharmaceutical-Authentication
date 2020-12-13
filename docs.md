# Droplet
Digital Ocean Droplet, 1 CPU, 2 GB, 50 GB SSD
OS, Ubuntu 20.04 (LTS) x64

# Access
In windows terminal: ssh root@IP address

or

Configure IP address to putty, using ssh connection

# Preperation for droplet

#Update the OS

apt update

apt upgrade


#Install useful helpers

apt install tree

apt install jq


#Set correct timezone

timedatectl set-timezone Continent/City


#Check the time

date

# Install docker
#Set up repository

sudo apt-get install apt-transport-https

sudo apt-get install ca-certificates

sudo apt-get install curl

sudo apt-get install gnupg-agent

sudo apt-get install software-properties-common


#Add Docker's official GPG key

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -


#Set up the stable repository

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu

sudo add-apt repository $(lsb_release -cs)

sudo add-apt repository stable"


#Install docker engine 

apt-get update

apt-get install docker-ce docker-ce-cli containerd.io


#Check the docker version

docker --version

# Install Docker-Compose

#Install docker-compose

sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose


#Apply executable permission to binary

sudo chmod +x /usr/local/bin/docker-compose


#Check docker-compose version

docker-compose --version

# Install Go Programming Language

