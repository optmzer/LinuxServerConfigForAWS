# LinuxServerConfigForAWS
Configuration steps to configure a Ubuntu server and deploy a flusk app on on AWS Lightsail.

## Server Details
Public IP: 13.236.9.9  
SSH port : 2200  

## Table of Contents
- [Create an instance of Ubuntu server Lightsail](#create-an-instance-of-ubuntu-server-lightsail)
- [Setting Up Your SSH Key Pair](#setting-up-your-ssh-key-pair)
- [Change Default SSH Port From 22 to 2200](#change-default-ssh-port-from-22-to-2200)
- [Setting Up Uncomplicated Firewall](#setting-up-uncomplicated-firewall)

- [Setting Up Graders SSH Key Pair](#setting-up-graders-ssh-key-pair)
- [Enforcing SSH Key Login](#enforcing-ssh-key-login)

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
>NOTE:
>- Keep this file secure as this is your keys to the server.
5. Navigate to the directory where you placed your `LightsailDefaultPrivateKey-<your-resource-location>.pam` and rename it for ease of use   
`mv LightsailDefaultPrivateKey-<your-resource-location>.pam lightsail-ssh-key.rsa`
6. Change access mode to the file `chmod 600 lightsail-ssh-key.rsa`
7. Connect to your server from your terminal  
`$ ssh -i lightsail-ssh-key.rsa ubuntu@13.236.9.9`.

## Change Default SSH Port From 22 to 2200
>WARNING:
> Because AWS has its own fire wall setup you may be locked out. So in your
> browser open the instance of Ubunt. Hit Networking tab. Then in Firewall
> section hit `+ Add another` rule button.  
> 
>| Application  |  Protocol  |  Port Range  |
>| :---        | :---     | :---        | 
>| Custom      | TCP      | 2200        |

1. Edit `/etc/ssh/sshd_config` file by `$ sudo nano /etc/ssh/sshd_config`  
2. Edit line with `Port 22` to `Port 2200`  
3. Save the change by Ctrl + X and exit from nano with Y  
4. Restart SSH with `$ sudo service ssh restart`  

Now you should be able to login with  
`$ ssh -i lightsail-ssh-key.rsa ubuntu@13.236.9.9 -p 2200`.

## Setting Up Uncomplicated Firewall
Configure Ubuntu internal firewall (UFW) to only allow incoming traffic to  

| Protocol    | Port  | 
| :---        | :---  |  
| SSH         | 2200  | 
| HTTP        | 80    | 
| NTP         | 123   |

`$ sudo ufw status` - Checks the status of firewall. It should be inactive by default.

* Part I - Deny all traffic
1. Deny all incoming traffic - `$ sudo ufw default deny incoming `
2. Deny all outgoing traffic - `$ sudo ufw default deny outgoing`

* Part II - Allow Only Specific Ports
1. Allow incoming ssh on port 2200 - `$ sudo ufw allow 2200/tcp`
2. Allow HTTP on port 80 - `$ sudo ufw allow 80/tcp`
3. Allow NTP on port 123 - `$ sudo ufw allow 123/udp`

* Part III - Enable The Firewall
1. Enable the firewall - `sudo ufw enable`
2. Check firewall rules - `sudo ufw status`

## Creating New User "Grader"
`$ sudo apt-get install finger` - To check if user was created correctly.


## Setting Up Graders SSH Key Pair


## Enforcing SSH Key Login
By default AWS supplies Ubuntu instance with password login off. But, it is always good to check.  
So, run `$ sudo cat /etc/ssh/sshd_config` and look for `PasswordAuthorization no` if it is set to `no` got ot [Update all packages](#update-all-packages) section.  
If PasswordAuthorization is set to yes:
1. Edit `/etc/ssh/sshd_config` file by `$ sudo nano /etc/ssh/sshd_config`  
2. Find line with `PasswordAuthorization yes` and change it to `PasswordAuthorization no`
3. Save the change by Ctrl + X and exit from nano with Y  
4. Restart SSH with `$ sudo service ssh restart`  

## Update all packages
Run 
1. `$ sudo apt-get update` and once this completes
2. `$ sudo apt-get upgrade`  
to get latest packages for your Ubuntu.
3. `$ sudo apt-get install git` - Install git.



## Contributing
This is a tutorial report. Please consider other projects.

## Author
Alexander Frolov

## References
[How To Set Up SSH Keys on Ubuntu 16.04 ](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1604)  
[Udacity, Generating Key Pairs](https://classroom.udacity.com/nanodegrees/nd004/parts/b2de4bd4-ef07-45b1-9f49-0e51e8f1336e/modules/56cf3482-b006-455c-8acd-26b37b6458d2/lessons/4331066009/concepts/48010894770923)  