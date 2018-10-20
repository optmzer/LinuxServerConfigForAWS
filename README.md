# LinuxServerConfigForAWS
Configuration steps to configure a Ubuntu server and deploy a flusk app on on AWS Lightsail.

## Server Details
Public IP: 52.65.144.252  
SSH port : 2200  

## Table of Contents
- [Create an instance of Ubuntu server Lightsail](#create-an-instance-of-ubuntu-server-lightsail)
- [Setting Up Your SSH Key Pair](#setting-up-your-ssh-key-pair)
- [Change Default SSH Port From 22 to 2200](#change-default-ssh-port-from-22-to-2200)
- [Setting Up Uncomplicated Firewall](#setting-up-uncomplicated-firewall)
- [Creating New User Grader](#creating-newuser-grader)
- [Give grader sudo access](#give-grader-sudo-access)
- [Setting Up Graders SSH Key Pair](#setting-up-graders-ssh-key-pair)
- [Enforcing SSH Key Login](#enforcing-ssh-key-login)
- [Install Git](#install-git)
- [Install Apache 2 Server and mod_wsgi](#install-apache-2-server-and-mod_wsgi)
- [Clone Your Project](#clone-your-project)
- [Install virtual environment and other dependencies](#install-virtual-environment-and-other-dependencies)
- [Configure Virtual Host](#[configure-irtual-host)
- [Install and Set Up Data Base](#install-and-set-up-data-base)
- [Update Authorized JavaScript Origins](#update-authorized-javascript-origins)
- [Create user_secrets.json](#create-user_secrets.json)
- [Update all packages](#update-all-packages)
- [Restart the server](#restart-the-server)
- [Contributing](#contributing)
- [Author](#author)
- [References](#references)

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
`$ ssh -i lightsail-ssh-key.rsa ubuntu@52.65.144.252`.

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

>NOTE:
>While we are at it we can disable ssh login as root user. As, by default AWS Ubuntu is
>instanciated with `PermitRootLogin prohibit-password`. So we'll change that to
>`PermitRootLogin no`

Now you should be able to login with  
`$ ssh -i lightsail-ssh-key.rsa ubuntu@52.65.144.252 -p 2200`.

## Setting Up Uncomplicated Firewall
Configure Ubuntu internal firewall (UFW) to only allow incoming traffic to  

| Protocol    | Port  | 
| :---        | :---  |  
| SSH         | 2200  | 
| HTTP        | 80    | 
| NTP         | 123   |

`$ sudo ufw status` - Checks the status of firewall. It should be inactive by default.

* Part I - Deny all traffic
1. `$ sudo ufw default deny incoming ` - Deny all incoming traffic
2. `$ sudo ufw default deny outgoing` - Deny all outgoing traffic

* Part II - Allow Only Specific Ports
1. `$ sudo ufw allow 2200/tcp` - Allow incoming ssh on port 2200
2. `$ sudo ufw allow 80/tcp` - Allow HTTP on port 80
3. `$ sudo ufw allow 123/udp` - Allow NTP on port 123

* Part III - Enable The Firewall
1. `sudo ufw enable` - Enable the firewall
2. `sudo ufw status` - Check firewall rules

## Creating New User Grader
`$ sudo apt-get install finger` - for later use to check if user was created correctly.  
`$ sudo adduser grader` - Adds new user to the system.

## Give grader sudo access
1. On AWS Ubuntu instance run `$ sudo ls -al /etc/sudoers.d` this will list file/s that contains a list of users who have `sudo` privileges. By default it called `90-cloud-init-users`

2. `sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader`
3. `sudo visudo -f /etc/sudoers.d/grader`  
Edit the file and leave only 1 line `grader ALL=(ALL) NOPASSWD:ALL`  
4. `ctrl + x` - to save and `Y` to exit.  
Now your grader should have `sudo` access to Ubuntu instance.

## Setting Up Graders SSH Key Pair
On your local Linux system run `$ ssh-keygen`  
The dialog will ask you to name the file. I named mine `grader-ssh-key.rsa`  
Leave passphrase empty as we are not going to use it.  
Your system will generate 2 files 
1. `grader-ssh-key.pub` - This is your public key. This will go to AWS ububnu instance.
2. `grader-ssh-key` - This is your private key, so keep it secret.

On your AWS ubuntu instance:  
1. Create `/home/grader/.ssh` folder by `$ sudo mkdir /home/grader/.ssh`  
2. Create file with graders ssh public key by `$ sudo nano /home/grader/.ssh/authorized_keys`  
3. Copy content of `grader-ssh-key.pub` into `/home/grader/.ssh/authorized_keys`
4. `ctrl + x` - to exit and press Y to save.
5. Change mode of .ssh directory - `$ sudo chmod 700 /home/grader/.ssh`
6. Change mdoe of authorized_keys file - `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`
7. Finally transfer the ownership of the .ssh directory from root to grader: `$ sudo chown -R grader:grader /home/grader/.ssh` - Recursivly changes ownership of the .ssh directory and it's content.
8. Open another terminal and try to connect to AWS ubuntu instance as a grader by:  
`$ ssh -i grader-ssh-key.rsa grader@52.65.144.252 -p 2200`

## Enforcing SSH Key Login
By default AWS supplies Ubuntu instance with password login off. But, it is always good to check.  
So, run `$ sudo cat /etc/ssh/sshd_config` and look for `PasswordAuthorization no` if it is set to `no` go to [Update all packages](#update-all-packages) section.  
If PasswordAuthorization is set to yes:
1. Edit `/etc/ssh/sshd_config` file by `$ sudo nano /etc/ssh/sshd_config`  
2. Find line with `PasswordAuthorization yes` and change it to `PasswordAuthorization no`
3. Save the change by Ctrl + X and exit from nano with Y  
4. Restart SSH with `$ sudo service ssh restart`  

## Install Git
1. `$ sudo apt-get install git` - Installs git.
2. `$ git config --global user.name <username>` - Configure your username.
3. `$ git config --global user.email <email>` - Configure your email.
4. `$ git congin --list` - to check your git configuration.

## Install Apache 2 Server and mod_wsgi
1. `$ sudo apt-get install apache2`
2. `$ sudo apt-get install libapache2-mod-wsgi-py3` - Installs `mod_wsgi` libraries for python 3 that enables Apache to serve Flask applications. 
3. `$ sudo a2enmod wsgi` - Enable mod_wsgi.
4. `$ sudo service apache2 start` - Start the Apache2 server.

## Clone Your Project
On AWS Lightsail instance create a folder `/var/www/thecatalog`  
`$ sudo git clone https://github.com/optmzer/catalog-web-app /var/www/thecatalog/thecatalog`  
Now CD to the project folder  
`$ cd /var/www/thecatalog/thecatalog`  
and move thecatalog.wsgi one level up.  
`$ sudo mv thecatalog.wsgi ../thecatalog.wsgi`  
So the folder structure will look like this.  
```
thecatalog/
    thecatalog.wsgi
    thecatalog/ - This is project folder to clone to
    . - The other project files
    .
```

## Install virtual environment and other dependencies
1. `sudo apt install python3-pip` - Install pip3 for python 3 with
2. `sudo pip3 install virtualenv`
3. CD to thecatalog project folder `sudo cd /var/www/thecatalog/thecatalog` and create virtual environment for this project.  
`sudo virtualenv venv`
4. `source venv/bin/activate` - Activate virtual environment.
5. `$ sudo chmod -R 777 venv` - Change permissions to virual environment folder.
6. `sudo pip3 install Flask` - Installs the Flask app
7. `sudo pip3 install bleach httplib2 request oauth2client sqlalchemy python-psycopg2` - Installs the rest of dependencies for the project to your virtual invironment.

## Configure Virtual Host
1. `sudo nano /etc/apache2/sites-available/thecatalog.conf` - Create thecatalog.conf file
2. Add the folowing content to it:
```
<VirtualHost *:80>
    ServerName 52.194.44.145
    ServerAlias www.lightsailthecatalogapp.com
    ServerAdmin your@email.com
    WSGIDaemonProcess thecatalog python-path=/var/www/thecatalog:/var/www/thecatalog/thecatalog:/var/www/thecatalog/thecatalog/venv/lib/python3.5/site-packages:/usr/local/lib/python3.5/dist-packages
    WSGIProcessGroup thecatalog
    WSGIScriptAlias / /var/www/thecatalog/thecatalog.wsgi
    <Directory /var/www/thecatalog/thecatalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/thecatalog/thecatalog/static
    <Directory /var/www/thecatalog/thecatalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```
>NOTE:
>1. The `WSGIDaemonProcess thecatalog python-path=path01:path02:path03` variable specifies where to look for your packages and dependencies for the project.
>2. The `ServerAlias www.lightsailthecatalogapp.com` is a preregistered domain name that has its own DNS zone record that points to static IP in `ServerName 52.194.44.145`. Reffer to [Making Route 53 the DNS Service for an Inactive Domain](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/migrate-dns-domain-inactive.html) for details.
3. `sudo a2ensite thecatalog` - Enables virtual environment. (To stop virtual environment in The Catalog project folder run `$ deactivate`)

## Install and Set Up Data Base
1. `sudo apt-get install postgresql` - Installs PostgreSQL
- Part I. Add User To The Data Base
1. `sudo su - postgres` - Log to your instance as superuser `postgres`
2. `psql` - Open PostgreSQL terminal
3. `postgres=# CREATE USER thecatalog WITH PASSWORD 'sequrePass';` - Creates a user named "thecatalog" with password "sequrePass". The user "thecatalog" is the flask app that is going to operate the DB. Namely, uploading images and data to it.
4. `postgres=# CREATE DATABASE thecatalog WITH OWNER thecatalog;` - Creates Data Base named "thecatalog".
- Part II. Give User Rights To Create/Modify Data Base and Lock It Up
1. `postgres=# ALTER USER thecatalog CREATEDB;` - Give right to "thecatalog" user to create Data Base.
2. `postgres=# CREATE DATABASE thecatalog WITH OWNER thecatalog;` - Creates a data base named "thecatalog" and owned by the user "thecatalog"
3. `postgres=# \c thecatalog` connects to "thecatalog" database.
- Part III. Revoke All Public Access Rights To This Data Base.
1. `thecatalog=# REVOKE ALL ON SCHEMA public FROM public;` - Revokes all public access rites from this DB.
2. `thecatalog=# GRANT ALL ON SCHEMA public TO thecatalog;` - Transfers all access rights to "thecatalog" user.
3. `thecatalog=# \q` - Quits PostgreSQL 
4. `ctrl + D` or type `$ exit` to exit superuser mode and return to grader.
5. `python3 /var/www/thecatalog/thecatalog/catalog_db_setup.py` - Populates the DB with predifined entries.

## Update Authorized JavaScript Origins
1. With your OAuth provider update JavaScript origin entry to allow authorization and redirection to your new address.
JavaScript origins: `http://www.lightsailthecatalogapp.com`  
Authorized Redirect: `http://www.lightsailthecatalogapp.com/thecatalog/`
2. Download JSON file from your OAuth provider with new configuration info.

## Create user_secrets.json
1. `sudo nano /var/www/thecatalog/thecatalog/user_secrets.json`, so the new project structure will look like this.
```
thecatalog/
    thecatalog.wsgi
    thecatalog/ - This is project folder to clone to
    user_secrets.json - New file
    . - The other project files
    .
```
2. Copy/Paste content of new JSON file from previous step into `user_secrets.josn`.

## Update all packages
Run 
1. `$ sudo apt-get update` and once this completes
2. `$ sudo apt-get upgrade`  
to get latest packages for your Ubuntu.

## Restart the server
1. Run `sudo service apache2 restart` - this restarts the server
2. Check your app at http://52.194.44.145/ or http://www.lightsailthecatalogapp.com/thecatalog/

## Contributing
This is a tutorial report. Please consider other projects.

## Author
Alexander Frolov

## References
* [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [How To Set Up SSH Keys on Ubuntu 16.04 ](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1604)  
* [Udacity, Generating Key Pairs](https://classroom.udacity.com/nanodegrees/nd004/parts/b2de4bd4-ef07-45b1-9f49-0e51e8f1336e/modules/56cf3482-b006-455c-8acd-26b37b6458d2/lessons/4331066009/concepts/48010894770923)  
* [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)