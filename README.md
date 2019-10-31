# Linux Server Configuration Project

## About the project

> A baseline installation of a Linux distribution on a virtual machine and prepare it to host web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers

* IP Address: 35.176.176.224
* SSH Port: 2200
* URL: `http://35.176.176.224`

### 1. Update all packages

```linux
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```

Enable automatic security updates

```linux
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

#### 2. Change timezone to UTC and Fix language issues

```linux
sudo timedatectl set-timezone UTC
sudo update-locale LANG=en_US.utf8 LANGUAGE=en_US.utf8 LC_ALL=en_US.utf8
```

##### 3. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow www
sudo ufw allow ntp
sudo ufw allow 2200/tcp
sudo ufw enable
```

##### 4. Go to Netowrking tab in your lightsail and do the following

1- go to Firewall
2- Click on +Add another
3- application >> Custom
4-Protocol >> TCP
5-port >> 2200

##### 5. Change the SSH port from 22 to 2200

Make sure to do the **previous step** before changing the port to 2200. Otherwise, you will lose your machine.

```linux
sudo nano /etc/ssh/sshd_config
```

* Locate the line **port 22** in the file */etc/ssh/sshd_config* and edit it to  **port 2200**, or any other desired port.
* Find the PermitRootLogin line and edit it to no.
* Find the PasswordAuthentication line and edit it to no.
* Save the file and run `sudo service ssh restart`

#### 6. Create a new user grader and Give him `sudo` access

```linux
sudo adduser grader
sudo nano /etc/sudoers.d/grader
```

Then add the following text `grader ALL=(ALL) NOPASSWD:ALL`

#### 7. Setup SSH keys for grader

* On local machine
`ssh-keygen`
Then choose the path for storing public and private keys
* On remote machine home as user grader

```linux
sudo su - grader
mkdir .ssh
touch .ssh/authorized_keys
sudo chmod 700 .ssh
sudo chmod 600 .ssh/authorized_keys
nano .ssh/authorized_keys
```

Then paste the contents of the public key created on the local machine

#### 8. Install Apache2 and mod-wsgi for python3 and Git

```linux
sudo apt-get install apache2 libapache2-mod-wsgi-py3 git
```

Note: For Python2 replace `libapache2-mod-wsgi-py3` with `libapache2-mod-wsgi`

#### 9. Install and configure PostgreSQL

```linux
sudo apt-get install libpq-dev python3-dev
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres
psql
```

Then

```linux
CREATE USER catalog WITH PASSWORD 'password';
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit
```

**Note:** In your catalog project you should change database engine to

```linux
engine = create_engine('postgresql://catalog:password@localhost/catalog')
```

#### 10. Clone the Catalog app from GitHub and Configure it

```linux
cd /var/www/
sudo mkdir catalog
sudo chown grader:grader catalog
git clone <your_repo_url> catalog
cd catalog
nano catalog.wsgi
```

Then add the following in `catalog.wsgi` file

```python
#!/usr/bin/python3
import sys
sys.stdout = sys.stderr

sys.path.insert(0,"/var/www/catalog")

from app import app as application
```

-If you don't have `requirements.txt` file, you can use

```python
pip3 install flask packaging oauth2client redis passlib flask-httpauth
pip3 install sqlalchemy flask-sqlalchemy psycopg2 bleach requests
```

Edit Authorized JavaScript origins

#### 12. Configure apache server

```linux
sudo nano /etc/apache2/sites-enabled/000-default.conf
```

Then add the following content:

```nano
# serve catalog app
<VirtualHost *:80>
  ServerName <IP_Address or Domain>
  ServerAdmin <Email>
  DocumentRoot /var/www/catalog
  WSGIDaemonProcess catalog user=grader group=grader
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
```

#### 13. Reload & Restart Apache Server

```linux
sudo service apache2 reload
sudo service apache2 restart
```

#### 14. run your site by enrtering the ip address (public ip) in your browser
