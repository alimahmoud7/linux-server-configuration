# Linux Server Configuration Project
This project was created for the *Full Stack Web Developer Nanodegree* at [**Udacity**](https://www.udacity.com/degrees/full-stack-web-developer-nanodegree--nd004).

### About the project
> A baseline installation of a Linux distribution on a virtual machine and prepare it to host web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers

* IP Address: [54.202.207.12](http://54.202.207.12/)
* SSH Port: 2200
* URL using DNS: [ec2-54-202-207-12.us-west-2.compute.amazonaws.com](http://ec2-54-202-207-12.us-west-2.compute.amazonaws.com/)

### Steps Followed to Configure the server

#### 1. Update all packages
```
sudo apt-get update
sudo apt-get upgrade
```
#### 2. Change timezone to UTC and Fix language issues 
```
sudo timedatectl set-timezone UTC
sudo update-locale LANG=en_US.utf8 LANGUAGE=en_US.utf8 LC_ALL=en_US.utf8
```
#### 3. Create a new user grader and Give him `sudo` access
```
sudo adduser grader
sudo nano /etc/sudoers.d/grader 
```
Then add the following text `grader ALL=(ALL) NOPASSWD:ALL`

#### 4. Setup SSH keys for grader
* On local machine 
`ssh-keygen`
Then choose the path for storing public and private keys
* On remote machine home as user grader
```
sudo su - grader
mkdir .ssh
cat .ssh/authorized_keys 
sudo chmod 700 .ssh
sudo chmod 600 .ssh/authorized_keys 
nano .ssh/authorized_keys 
```
Then paste the contents of the public key created on the local machine

#### 5. Change the SSH port from 22 to 2200 | Enforce key-based authentication | Disable login for root user
```
sudo nano /etc/ssh/sshd_config
```
Then change the following:
* Find the Port line and edit it to 2200.
* Find the PasswordAuthentication line and edit it to no.
* Find the PermitRootLogin line and edit it to no.
* Save the file and run `sudo service ssh restart`

#### 6. Configure the Uncomplicated Firewall (UFW)
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw allow 8000/tcp  `serve another app on the server`
sudo ufw enable
```
#### 7. Install Apache2 and mod-wsgi for python3 and Git
```
sudo apt-get install apache2 libapache2-mod-wsgi-py3 git
```
#### 8. Install and configure PostgreSQL
```
sudo apt-get install libpq-dev python3-dev
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres
psql
```
Then
```
CREATE USER catalog WITH PASSWORD '123534';
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit
```
#### 9. Clone the Catalog app from GitHub and Configure it
```
cd /var/www/
sudo mkdir catalog
sudo chown grader:grader catalog
git clone https://github.com/AliMahmoud7/item-catalog-fsnd catalog
cd catalog
git checkout production
nano catalog.wsgi
```
Then add the following in `catalog.wsgi` file
```python
#!/usr/bin/python3
import sys
sys.stdout = sys.stderr

activate_this = '/var/www/catalog/env/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

sys.path.insert(0,"/var/www/catalog")

from app import app as application
```
Setup virtual environment and Install app dependencies 
```
sudo apt-get install python3-pip
sudo -H pip3 install virtualenv
virtualenv env
source env/bin/activate
pip3 install -r requirements.txt
```
Edit Authorized JavaScript origins

#### 10. Clone the Neighborhood map app from GitHub
```
cd /var/www/
sudo mkdir map
sudo chown grader:grader map
git clone https://github.com/AliMahmoud7/neighborhood-map-fsnd
```
Go to [54.202.207.12:8000](http://54.202.207.12:8000/) to view it

#### 11. Configure apache server
```
sudo nano /etc/apache2/sites-enabled/000-default.conf
```
Then add the following content:
```
# serve catalog app
<VirtualHost *:80>
  ServerName 54.202.207.12
  ServerAlias ec2-54-202-207-12.us-west-2.compute.amazonaws.com
  ServerAdmin ali.mahmoud@engineer.com
  DocumentRoot /var/www/catalog
  WSGIDaemonProcess catalog user=grader group=grader python-home=/var/www/catalog/env
  WSGIScriptAlias / /var/www/catalog/catalog.wsgi

  <Directory /var/www/catalog>
    WSGIProcessGroup catalog
    WSGIApplicationGroup %{GLOBAL}
    Require all granted
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

# Serve another project on the server (different port)
LISTEN 8000
<VirtualHost *:8000>
  ServerName 54.202.207.12
  ServerAlias ec2-54-202-207-12.us-west-2.compute.amazonaws.com
  DocumentRoot /var/www/map
  ServerAdmin ali.mahmoud@engineer.com

 <Directory /var/www/map/.git>
    Require all denied
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

#### 12. Restart Apache Server
```
sudo service apache2 restart
```
