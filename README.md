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

Now change SSH port 22 to 2200:

`sudo nano /etc/ssh/sshd_config`, change `Port 22` to `Port 2200`

## Give `Grader` Access
