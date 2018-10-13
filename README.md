# LinuxServerConfigForAWS
Configuration steps to configure a Ubuntu server and deploy a flusk app on on AWS Lightsail.

## Server Details
Public IP: 13.238.254.41  
SSH port : 2200  

## Table of Contents
- [Create an instance of Ubuntu server Lightsail](#create-an-instance-of-ubuntu-server-lightsail)
- [Setting Up Your SSH Key Pair](#setting-up-your-ssh-key-pair)
- [Setting Up Graders SSH Key Pair](#setting-up-graders-ssh-key-pair)
- [Change Default SSH Port From 22 to 2200](#change-default-ssh-port-from-22-to-2200)
- [Enforcing SSH Key Login](#enforcing-ssh-key-login)
- [Setting Up Uncomplicated Firewall](#setting-up-uncomplicated-firewall)

- [Contributing](#contributing)
- [Author](#author)

## Create an instance of Ubuntu server Lightsail
1. Go to AWS Management Console.
2. Select Lightsail from `All services/Compute/Lightsail`  
After selecting Lightsail you are redirected to Create instance page.
3. Select Instance Location - Optional as this is tutorial project
4. Pick your instance image `Linux/Unix`
5. Select a blueprint `OS Only`
6. Select `Ubuntu 16.04 LTS` as an operating system.
7. Choose a cheapest plan available.
8. Name your instance - Optional.  
I went for generic name.

## Setting Up Your SSH Key Pair
1. Visit your AWS Management Cpnsole then `Compute/Lightsail`
2. Select Account in right hand corner.
3. When redirected to your account page of Lightsail choose SSH Keys tab.
4. Hit Download button for your location of the instance.  
file will be named like `LightsailDefaultPrivateKey-<your-resource-location>.pam`
* Keep this file secure as this is your keys to the server.
5. Navigate to the directory where you placed your `LightsailDefaultPrivateKey-<your-resource-location>.pam` and rename it for ease of use   
`mv LightsailDefaultPrivateKey-<your-resource-location>.pam lightsail-ssh-key.rsa`
6. Change access mode to the file `chmod 600 lightsail-ssh-key.rsa`
7. Connect to your server from your terminal  
`ssh -i lightsail-ssh-key.rsa ubuntu@13.238.254.41`.

## Setting Up Graders SSH Key Pair

## Change Default SSH Port From 22 to 2200

## Enforcing SSH Key Login

## Setting Up Uncomplicated Firewall

## Update all packages

Run `sudo apt-get update` and `sudo apt-get upgrade` to get latest packages for your Ubuntu.

## Contributing
This is a tutorial report. Please consider other projects.

## Author
Alexander Frolov