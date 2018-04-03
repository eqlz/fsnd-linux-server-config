# Linux Server Configuration

## Connect to server
IP address: `18.219.3.185`

SSH port: `2200`

URL: `http://project.emilyzhang.work`

## Secure the server
### Update all currently installed packages

`apt-get update`

`apt-get upgrade`


### Configure the Uncomplicated Firewall

Check the status of the firewall: `sudo ufw status`

Block all incoming connections on all ports: `sudo ufw default deny incoming`

Allow outgoing connection on all ports: `sudo ufw default allow outgoing`

Allow incoming connection for SSH on port 2200: `sudo ufw allow 2200/tcp`

Allow incoming connections for HTTP on port 80: `sudo ufw allow www`

Check the rules that have been added before enabling the firewall use: `sudo ufw show added`

Enable the firewall: `sudo ufw enable`


### Change the SSH port from 22 to 2200

First, log into Amazon Lightsail homepage, click your instance.  Then find "Networking", under "Networking", find "Firewall".  Under "Firewall", click "+ Add another": make sure the Port range is 2200.

Lightsail accepts port 22 and 80 by default.  You need to set up Lightsail to accept 2200 connection.

Now change SSH port 22 to 2200: `sudo nano /etc/ssh/sshd_config`, change `Port 22` to `Port 2200`.

Restart SSH server: `sudo service ssh restart`.

Connect to server: `ssh <username>@<lightsail_public_ip> -i <path_to_your_private_key> -p 2200`.

## Give `Grader` Access
### Create user `grader`

`sudo adduser grader`

### Give `grader` permission to `sudo`

`sudo nano /etc/sudoers.d/grader`.  Then write `grader ALL=(ALL) NOPASSWD:ALL` into the file.

### Set up SSH key pair for `grader`

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

Helpful resource: https://aws.amazon.com/premiumsupport/knowledge-center/new-user-accounts-linux-instance/

## Prepare to Deploy
Configure local timezone to UTC: `sudo timedatectl set-timezone UTC`

### Install Apache

Install Apache: `sudo apt-get install apache2`

Configure Apache to serve a Python mod_wsgi application: `sudo apt-get install libapache2-mod-wsgi`.  If you're coding in Python3, then:
`sudo apt-get install libapache2-mod-wsgi-py3`

### Install and Configure Postgresql

Install Postgresql: `sudo apt-get install postgresql postgresql-contrib`.

Create a PostgreSQL user `catalog`: `sudo -u postgres createuser -P catalog`.  You will be prompted for a password.

Create an database `catalog` whose owner is user `catalog`: `sudo -u postgres createdb -O catalog catalog`.

PostgreSQL official documents on creating database user and database iteself:

https://www.postgresql.org/docs/current/static/app-createuser.html
https://www.postgresql.org/docs/current/static/app-createdb.html

### Install Flask, Sqlalchemy, Python Libraries
```
sudo apt-get install python-psycopg2 
sudo apt-get install python-flask
sudo apt-get install python-sqlalchemy 
sudo apt-get install python-pip

sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2
```

### Install Git

`sudo apt-get install git`

### Clone Catalog Repository from Github

```
cd /var/www/
sudo mkdir fullstack-nanodegree-vm
sudo chown www-data:www-data fullstack-nanodegree-vm/
sudo -u www-data git clone https://github.com/eqlz/fsnd-project-mgt-app.git fullstack-nanodegree-vm
```

Make `.git` file inaccessible: `sudo nano .htaccess`, add a line of `RedirectMatch 404 /\.git` into the file.  Then save it.

### Set Up Directory Structure
Set your directory structure look like this, this is what my directory looks like on my server:
```
/var/www/fullstack-nanodegree-vm/
--------------------------------catalog
---------------------------------------catalog
----------------------------------------------__init__.py
----------------------------------------------models.py
----------------------------------------------static
----------------------------------------------templates
----------------------------------------------client_secrets.json
---------------------------------------catalog.wsgi
```

Folder structure:
1. Under `fullstack-nanodegree-vm`, there is only one item, folder `catalog`.
2. Under first `catalog` folder, there are two items, folder `catalog`, file `catalog.wsgi`.
3. Under second `catalog` folder, put all the catalog project files there.
4. It's necessary to have two `catalog` folders, and are structured in this way.

Helpful resource: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

### Create `catalog.wsgi` File
Location: see directory structure above, under the first `catalog` file

Content:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/fullstack-nanodegree-vm/catalog/") # pay attention to the path, yours may be different from mine.

from catalog import app as application

application.secret_key = 'YOUR_SECRET_KEY'
```


### `__init__.py`
From the folder structure above, you'll notice there is an `__init__.py` file under the second `catalog` folder.
This `__init__.py` file is the python file that contains all your backend logic.

You have alreay created such a file while finishing catalog project.  It's just your file's name may not be `__init__.py`.  Mine is
`views.py`, you can see my `views.py` here: https://github.com/eqlz/fsnd-project-mgt-app/blob/master/views.py

Now, change `views.py` to `__init__.py`: `sudo mv views.py __init__.py`.

### Edit `__init__.py`








