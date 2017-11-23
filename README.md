# Udacity Linux Server Configuration

This is the final project for "Full Stack Web Developer Nanodegree" on Udacity.

In this project, a Linux virtual machine needs to be configurated to support the Item Catalog website.

Public IP http://34.238.162.142.

## Instructions for SSH access to the instance

1. Download Private Key from the **SSH keys** section in the **Account** section on Amazon.

2. Move the private key file into the folder `~/.ssh`. If you downloaded the file to the Downloads folder, just execute
   the following command in your terminal.

```
mv ~/Downloads/LightsailDefaultPrivateKey-us-east-1.pem ~/.ssh/
```

3. Open your terminal and type in to change permissions.

```
chmod 400 ~/.ssh/LightsailDefaultPrivateKey-us-east-1.pem
```

4. In your terminal, type in

```
ssh -i ~/.ssh/LightsailDefaultPrivateKey-us-east-1.pem ubuntu@34.238.162.142
```

## Create a new user named grader

1. `sudo adduser grader`
2. `sudo touch /etc/sudoers.d/grader`
3. `sudo nano /etc/sudoers.d/grader`, then add `grader ALL=(ALL:ALL) NOPASSWD:ALL`, to the grader file, save and quit

## Set ssh login using keys

1. Generate keys on local machine using`ssh-keygen` ; then save the private key in `~/.ssh` on local machine
2. Deploy public key on developement enviroment

   On you virtual machine:

```
$ su - grader
$ mkdir .ssh
$ touch .ssh/authorized_keys
$ nano .ssh/authorized_keys
```

Copy the public key (_one with the extension .pub_) generated on your local machine to this file and save

```
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
```

3. Reload SSH using `sudo service ssh restart`
4. Now you can use ssh to login with the new user you created

   `ssh -i ~/.ssh/[privateKeyFilename] grader@34.238.162.142`

## Update all currently installed packages

```
sudo apt-get update
sudo apt-get upgrade
```

## Change the SSH port from 22 to 2200

1. Use `sudo nano /etc/ssh/sshd_config` and then change Port 22 to Port 2200 , save & quit.
2. Reload SSH using `sudo service ssh restart`

**Note:** Remember to add and save port 2200 with _Application **as** Custom and Protocol **as** TCP_ in the Networking
section of your instance on Amazon Lightsail.

## Configure the Uncomplicated Firewall (UFW)

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and
NTP (port 123)

```
sudo ufw allow ssh
sudo ufw allow www
sudo ufw allow ntp
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable
sudo ufw status
```

## Configure the timezone UTC

1. Configure the time zone

```
sudo dpkg-reconfigure tzdata
```

## Install and configure Apache to serve a Python mod_wsgi application

1. Install Apache

```
sudo apt-get install apache2
```

2. Install mod_wsgi

```
sudo apt-get install python-setuptools libapache2-mod-wsgi
```

3. Restart Apache

```
sudo service apache2 restart
```

## Install and configure PostgreSQL

1. Install PostgreSQL

```
sudo apt-get install postgresql
```

2. Check if no remote connections are allowed

```
sudo nano /etc/postgresql/9.3/main/pg_hba.conf
```

3. Login as user "postgres"

```
sudo su - postgres
```

4. Get into postgreSQL shell `psql`

5. Create a new database named catalog and create a new user named catalog in postgreSQL shell

```
postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;
```

6. Set a password for user catalog

```
postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
```

7. Give user "catalog" permission to "catalog" application database

```
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
```

8. Quit postgreSQL `postgres=# \q`

9. Exit from user "postgres"

```
exit
```

## Install git, clone and setup your Catalog App project.

1. Install Git

```
sudo apt-get install git
```

2. Use cd to move to /var/www directory

```
cd /var/www
```

3. Create the application directory

```
sudo mkdir flask-app
```

4. Move inside this directory using

```
cd flask-app
```

5. Clone the Catalog App to the virtual machine

```
git clone https://github.com/ajames20/fsnd-catalog-.git .
```

6. Move into FlaskApp directory using

```
cd flask-app
```

7. Rename `app.py` to `__init__.py` using `sudo mv app.py __init__.py`.

8. Install pip

```
sudo apt-get install python-pip
```

9. Use pip to install dependencies -

   * `sudo pip install sqlalchemy flask-sqlalchemy psycopg2 bleach requests`
   * `sudo pip install flask packaging oauth2client passlib flask-httpauth`

10. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`

11. Create database schema `sudo python models.py`

12. Fill database `sudo python dbsetup.py`

## Configure and Enable a New Virtual Host

1. Create FlaskApp.conf to edit: `sudo nano /etc/apache2/sites-available/flaskapp.conf`
2. Add the following lines of code to the file to configure the virtual host.

   ```
   <VirtualHost *:80>
   	ServerName dbsetup.py
   	ServerAdmin ajames20@gmail.com
   	WSGIScriptAlias / /var/www/flask-app/flaskapp.wsgi
   	<Directory /var/www/flask-app/>
   		Order allow,deny
   		Allow from all
   	</Directory>
   	Alias /static /var/www/flask-app/static
   	<Directory /var/www/flask-app/static/>
   		Order allow,deny
   		Allow from all
   	</Directory>
   	ErrorLog ${APACHE_LOG_DIR}/error.log
   	LogLevel warn
   	CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```

3. Enable the virtual host with the following command: `sudo a2ensite flaskapp`

## Create the .wsgi File

1. Create the .wsgi File under /var/www/flask-app:

   ```
   cd /var/www/flask-app
   sudo nano flaskapp.wsgi
   ```

2. Add the following lines of code to the flaskapp.wsgi file:

   ```
   #!/usr/bin/python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0,"/var/www/flask-app/")

   from flask-app import app as application
   application.secret_key = 'super_secret_key'
   ```

## Restart Apache

1. Disable the default virtual host with:

```
sudo a2dissite 000-default.conf
```

2. Restart Apache

```
sudo service apache2 restart
```

3. App Available at

```
http://34.238.162.142/
http://34.238.162.142/.us-east-2.compute.amazonaws.com
```

## References:

1. Udacity's FSND Forum
2. https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
