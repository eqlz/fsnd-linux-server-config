# Linux Server Configuration

## Connect to server
IP address: `18.219.3.185`

SSH port: `2200`

URL: `http://project.emilyzhang.work`

## Secure the server
__Update all currently installed packages__

`apt-get update`

`apt-get upgrade`


__Configure the Uncomplicated Firewall__

Check the status of the firewall: `sudo ufw status`

Block all incoming connections on all ports: `sudo ufw default deny incoming`

Allow outgoing connection on all ports: `sudo ufw default allow outgoing`

Allow incoming connection for SSH on port 2200: `sudo ufw allow 2200/tcp`

Allow incoming connections for HTTP on port 80: `sudo ufw allow www`

Check the rules that have been added before enabling the firewall use: `sudo ufw show added`

Enable the firewall: `sudo ufw enable`


__Change the SSH port from 22 to 2200__

First, log into Amazon Lightsail homepage, click your instance.  Then find "Networking", under "Networking", find "Firewall".  Under "Firewall", click "+ Add another": make sure the Port range is 2200.

Lightsail accepts port 22 and 80 by default.  You need to set up Lightsail to accept 2200 connection.

Now change SSH port 22 to 2200: `sudo nano /etc/ssh/sshd_config`, change `Port 22` to `Port 2200`.

Restart SSH server: `sudo service ssh restart`.

Connect to server: `ssh <username>@<lightsail_public_ip> -i <path_to_your_private_key> -p 2200`.

## Give `Grader` Access
__Create user `grader`__

`sudo adduser grader`

__Give `grader` permission to `sudo`__

`sudo nano /etc/sudoers.d/grader`.  Then write `grader ALL=(ALL) NOPASSWD:ALL` into the file.

__Set up SSH key pair for `grader`__

As root user, on server, create a folder: `mkdir /home/grader/.ssh`

On your local machine, generate a key pair: `ssh-keygen`

Install publick key on server, as user `grader`: 

`touch .ssh/authorized_keys`

`nano .ssh/authorized_keys`

Read out the content of public key file on your local machine, copy and then paste to `.ssh/authorized_keys` file on your server.  Then
save it.

Now log into the server as grader: `ssh grader@<lightsail_public_ip> -i <path_to_your_private_key> -p 2200`.

On server, as grader, force key based authentication: `sudo nano /etc/ssh/sshd_config`, change `PasswordAuthentication` to `no`.  Then save it.

On server, as grader, then modify file permission: `chmod 700 .ssh`, `chmod 600 .ssh/authorized_keys`.

## Prepare to Deploy
Configure local timezone to UTC: `sudo timedatectl set-timezone UTC`

__Install Apache__

Install Apache: `sudo apt-get install apache2`

Configure Apache to serve a Python mod_wsgi application: `sudo apt-get install libapache2-mod-wsgi`.  If you're coding in Python3, then:
`sudo apt-get install libapache2-mod-wsgi-py3`

__Install and Configure Postgresql__

Install Postgresql: `sudo apt-get install postgresql postgresql-contrib`.

Create a PostgreSQL user `catalog`: `sudo -u postgres createuser -P catalog`.  You will be prompted for a password.

Create an database `catalog` whose owner is user `catalog`: `sudo -u postgres createdb -O catalog catalog`.

PostgreSQL official documents on creating database user and database iteself:

https://www.postgresql.org/docs/current/static/app-createuser.html
https://www.postgresql.org/docs/current/static/app-createdb.html

__Install Flask, Sqlalchemy, Python Libraries__
```
sudo apt-get install python-psycopg2 
sudo apt-get install python-flask
sudo apt-get install python-sqlalchemy 
sudo apt-get install python-pip

sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2
```






