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

Check the status of the firewall, use: `sudo ufw status`

Block all incoming connections on all ports: `sudo ufw default deny incoming`

Allow outgoing connection on all ports: `sudo ufw default allow outgoing`

Allow incoming connection for SSH on port 2200: `sudo ufw allow 2200/tcp`

Allow incoming connections for HTTP on port 80: `sudo ufw allow www`

Check the rules that have been added before enabling the firewall use: `sudo ufw show added`

Enable the firewall: `sudo ufw enable`


__Change the SSH port from 22 to 2200__

`sudo nano /etc/ssh/sshd_config`, change `Port 22` to `Port 2200`

## Give `Grader` Access
