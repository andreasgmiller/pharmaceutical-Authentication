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
