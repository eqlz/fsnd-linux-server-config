# Linux Server Configuration

## Connect to server
IP address: ~~`18.219.3.185`~~ `18.188.54.6`

SSH port: `2200`

URL: `http://project.emilyzhang.work`

## Secure the server
### Update all currently installed packages

`apt-get update`

`apt-get upgrade`

To automatically update, use [_unattended-upgrades_](https://help.ubuntu.com/lts/serverguide/automatic-updates.html) package 

### Disable logging in as `root` remotely
`cd` to `/etc/ssh`, `sudo nano sshd_config`

Find the line `PermitRootLogin`, change from `prohibit-password` to `no`.

Restart ssh server: `sudo service ssh restart`

Connect to server: `ssh <username>@<lightsail_public_ip> -i <path_to_your_private_key> -p 2200`.

### Configure the Uncomplicated Firewall

Check the status of the firewall: `sudo ufw status`

Block all incoming connections on all ports: `sudo ufw default deny incoming`

Allow outgoing connection on all ports: `sudo ufw default allow outgoing`

Allow incoming connection for SSH on port 2200: `sudo ufw allow 2200/tcp`

Allow incoming connections for HTTP on port 80: `sudo ufw allow www`

Check the rules that have been added before enabling the firewall use: `sudo ufw show added`

Enable the firewall: `sudo ufw enable`


### Change the SSH port from 22 to 2200

Lightsail accepts port 22 and 80 by default.  You need to set up Lightsail to accept 2200 connection.  Log into Amazon Lightsail homepage, click your instance.  Then find "Networking", under "Networking", find "Firewall".  Under "Firewall", click "+ Add another": make sure the Port range is 2200.

Now change SSH port 22 to 2200: `sudo nano /etc/ssh/sshd_config`, change `Port 22` to `Port 2200`.

Restart SSH server: `sudo service ssh restart`.

Connect to server: `ssh <username>@<lightsail_public_ip> -i <path_to_your_private_key> -p 2200`.

Diable SSH port 22: `sudo ufw deny 22`.

## Give `Grader` Access
### Create user `grader`

`sudo adduser grader`

### Give `grader` permission to `sudo`

`sudo nano /etc/sudoers.d/grader`.  Then write `grader ALL=(ALL) NOPASSWD:ALL` into the file.

### Set up SSH key pair for `grader`

As root user, on server, create a folder: `mkdir /home/grader/.ssh`

On your local machine, generate a key pair: `ssh-keygen`

Install public key on server, as user `grader`: 

`touch .ssh/authorized_keys`

`nano .ssh/authorized_keys`

Read out the content of public key file on your local machine, copy and then paste to `.ssh/authorized_keys` file on your server.  Then
save it.

Now log into the server as grader: `ssh grader@<lightsail_public_ip> -i <path_to_your_private_key> -p 2200`.

On server, as grader, force key based authentication: `sudo nano /etc/ssh/sshd_config`, change `PasswordAuthentication` to `no`.  Then save it.

On server, as grader, then modify file permission: `chmod 700 .ssh`, `chmod 600 .ssh/authorized_keys`.

Helpful resource: [Add new user in Linux instance](https://aws.amazon.com/premiumsupport/knowledge-center/new-user-accounts-linux-instance/)

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

Hepful resource: PostgreSQL official documents [Create database user](https://www.postgresql.org/docs/current/static/app-createuser.html), [Create database](https://www.postgresql.org/docs/current/static/app-createdb.html)

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

Helpful resource: [How to deloy a Flask application on an Ubuntu VSP](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

### Create `catalog.wsgi` File
Location: see directory structure above, under the first `catalog` file

Command: `sudo nano catalog.wsgi`

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


### Have `__init__.py` file
From the folder structure above, you'll notice there is an `__init__.py` file under the second `catalog` folder.
This `__init__.py` file is the python file that contains all your backend logic.

You have alreay created such a file while finishing catalog project.  It's just your file's name may not be `__init__.py`.  Mine is
`views.py`, you can see my `views.py` [here](https://github.com/eqlz/fsnd-project-mgt-app/blob/master/views.py)

Now, change `views.py` to `__init__.py`: `sudo mv views.py __init__.py`.

### Edit `__init__.py` file
`sudo nano __init__.py`

Change `engine = create_engine('sqlite:///projectmgtwithuser.db')` to

```
engine = create_engine('postgresql://catalog:<password you set up when creating postgresql user catalog>@localhost/catalog')
```

Change `CLIENT_ID = json.loads(open('client_secrets.json', 'r').read())['web']['client_id']`'s path to the __abosulte path__

```
CLIENT_ID = json.loads(open('/var/www/fullstack-nanodegree-vm/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']
```

### Edit `models.py` File
In my `models.py` file, change `engine = create_engine('sqlite:///projectmgtwithuser.db')` to

```
engine = create_engine('postgresql://catalog:<password you set up when creating postgresql user catalog>@localhost/catalog')
```

### Update Google Oauth `client_secrets.json` & Google API Credentials
In `client_secrets.json` file, update 

`"redirect_uris":["http://project.emilyzhang.work"]`, 

`"javascript_origins":["http://project.emilyzhang.work"]`.


In Google developer console, update your credentials:

Authorized JavaScript origins: `http://project.emilyzhang.work`,

Authorized redirect URIs: `http://project.emilyzhang.work`.


Error: Permission denied to generate login hint for target domain

Amazon Lightsail's public IP doesn't work for me, see the reason [here](https://stackoverflow.com/questions/36020374/google-permission-denied-to-generate-login-hint-for-target-domain-not-on-localh)

## Configure and Enable Apache2
To server catalog app via Apache web server, create a virtual host configuration file first

`sudo nano /etc/apache2/sites-available/catalog.conf`

`catalog.conf`'s content, pay attention to the path, mine may be different from yours:

```
<VirtualHost *:80>
                ServerName 18.219.164.45

                ServerAdmin admin@18.219.164.45

                WSGIDaemonProcess catalog user=www-data group=www-data threads=5
                WSGIProcessGroup catalog
                WSGIApplicationGroup %{GLOBAL}

                WSGIScriptAlias / /var/www/fullstack-nanodegree-vm/catalog/catalog.wsgi

                <Directory /var/www/fullstack-nanodegree-vm/catalog/catalog/>
                Require all granted
                </Directory>

                Alias /static /var/www/fullstack-nanodegree-vm/vagrant/catalog/catalog/static
                <Directory /var/www/fullstack-nanodegree-vm/catalog/catalog/static/>
                Require all granted
                </Directory>

                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Disable the default virtual host: `sudo a2dissite 000-default.conf`

Enable the virtual host just created: `sudo a2ensite catalog.conf`

To make these changes live restart Apache2: `sudo service apache2 restart`

## Run Catalog App
`python __init__.py`

`sudo service apache2 restart`

## Helpful resource from other Udacity students
[Tarja M](https://github.com/otsop110/fullstack-nanodegree-linux-server-configuration)

[Steve Wooding](https://github.com/SteveWooding/fullstack-nanodegree-linux-server-config)

