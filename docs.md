# Droplet
Digital Ocean Droplet, 1 CPU, 2 GB, 50 GB SSD

OS, Ubuntu 20.04 (LTS) x64

# Access
In windows terminal: ssh root@IP address

or

Configure IP address to putty, using ssh connection

# Preperation for droplet

``` 
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

```
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
Go version 1.14.x is required.

#Download and Extract Go, latest version 0.4.10.20 1.14.9

sudo wget -c https://dl.google.com/go/go1.14.9.linux-amd64.tar.gz -O - | tar -xz -C /usr/local

#Add the Go binary to the path

vi $HOME/.profile

export PATH="$PATH:/usr/local/go/bin:/root/fabric/fabric-samples/bin"

#Point the GOPATH env var to the fabric workspace folder

export GOPATH=$HOME/fabric

#Save and Leave VIM editor

ESC & Shift ZZ

#Reload the profile

source vi $HOME/.profile

#Check Go version

go version

#Check the vars

printenv | grep PATH

# Install Node.js

#Add PPA from Nodesource

curl -sL https://deb.nodesource.com/setup_12.x -o nodesource_setup.sh

#Call the install script

sudo bash nodesource_setup.sh

#Install node.js

sudo apt-get install -y nodejs

#Check node version

node -v

# Install Samples, Binaries, and Docker Images

#Create and access new directory

mkdir fabric

cd fabric

#Install fabric version 2.2 

curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.2.1 1.4.9

#Install latest fabric version

curl -sSL https://bit.ly/2ysbOFE | bash -s

#Check downloaded docker images

docker images

#Check the bin cmd

peer version

#Delete docker images

docker rmi -f image ID #This deletes docker images one by one.

docker rmi -f $(docker images -a -q) #This deletes all docker images at once.

# Test the Installation

#Switch to base folder

cd fabric-samples/test-network

#Help

./network.sh --help

#Bring up the network

./network.sh up createChannel -c channel1

#Bring network down

./network.sh down

