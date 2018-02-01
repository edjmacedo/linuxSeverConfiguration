# linuxSeverConfiguration
1. WebAddress: http://ec2-18-219-75-216.us-east-2.compute.amazonaws.com
2. IPAddress: 18.219.75.216
3. SSH port: 2200

## Create a new ubuntu instance and user
1. `Create one ubuntu instance on AWS service`
2. `sudo adduser grader`
3. `sudo touch /etc/sudoers.d/grader`
4. `sudo vim /etc/sudoers.d/grader`, add `grader ALL=(ALL:ALL) ALL`, on first line

## Set ssh login using keys
1. Generate ssh keys on local machine `ssh-keygen`
2. Create on instance a file for authorized keys

	On Amazon instance:
	```
	$ su - grader
	$ mkdir .ssh
	$ touch .ssh/authorized_keys
	$ vim .ssh/authorized_keys
	```
	Copy the public key generated on your local machine to authorized_keys
	```
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys
	```
	
3. reload SSH using `sudo /etc/init.d/ssh restart`
4. Login from your local machine to instance using SSH connection

	`ssh -i [privateKeyFilename] grader@18.219.75.216`

## Update distribution with apt-get

	sudo apt update
	sudo apt upgrade

## Change the SSH port to 2200 and deny root login
1. Use `sudo vim /etc/ssh/sshd_config`, search by `Port` and update to `2200`.
2. In PermitRootLogin, update value to `no`
3. Reload SSH using `sudo /etc/init.d/ssh restart`
4. In next SSH connection use it will be necessary pass port parameter:

```
ssh -i [privateKeyFilename] grader@18.219.75.216 -p 2200
```

## Configure the Uncomplicated Firewall (UFW)

	sudo ufw allow 2200/tcp
	sudo ufw allow 80/tcp
	sudo ufw allow 123/udp
	sudo ufw enable

## Install and configure Apache to serve a Python mod_wsgi application
1. Install Apache: `sudo apt install apache2`
2. Install mod_wsgi: `sudo apt install python-setuptools libapache2-mod-wsgi`
3. Restart Apache: `sudo /etc/init.d/apache2 restart`

## Install and configure PostgreSQL
1. `sudo apt-get install postgresql`
2. `sudo su - postgres`
3. `psql`
4. Create a new database named catalog
5. Create a new user named catalog in postgreSQL
	
	```
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
5. Set a password for user catalog
	
	```
	postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
	```
6. Give user "catalog" permission to "catalog" application database
	
	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
7. Quit postgreSQL `postgres=# \q`
8. Exit from user "postgres" 
	
	```
	exit
	```
## Configuring locales
1. `export LC_ALL="en_US.UTF-8"`
2. `export LC_TYPE="en_US.UTF-8"`
3. `sudo dpkg-reconfigure locales`

## Install git, clone and setup your Catalog App project.
1. Install Git using `sudo apt install git`
2. got to: `cd /var/www`
3. create folder to application: `sudo mkdir ItemCatalog`
4. Inside ItemCatalog folder, clone the Catalog App `git clone https://github.com/edvanmacedo/ItemCatalog.git`
5. Rename `itemapplycation.py` to `__init__.py`
6. Edit `database_setup.py`, and `__init__.py` and change `engine = create_engine('sqlite:///catalog.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
7. Install pip `sudo apt install python-pip`
8. Install sqlAlchemy: `sudo pip install sqlalchemy`
9. Install Flask: ` sudo pip install Flask`
10. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
11. Create database schema `sudo python database_setup.py`

## Configure and Enable a New Virtual Host
1. Create FlaskApp.conf to edit: `sudo python /etc/apache2/sites-available/ItemCatalog.conf`
2. Add the following lines of code to the file to configure the virtual host. 
	
	```
	<VirtualHost *:80>
        ServerName 18.219.75.216
        ServerAlias ec2-18-219-75-216.us-east-2.compute.amazonaws.com
        ServerAdmin edvan.macedo.jr@gmail.com
        WSGIScriptAlias / /var/www/ItemCatalog/vagrant/flaskapp.wsgi
        <Directory /var/www/ItemCatalog/vagrant/ItemCatalog/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/ItemCatalog/vagrant/ItemCatalog/static
        <Directory /var/www/ItemCatalog/vagrant/ItemCatalog/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
	```
3. Enable the virtual host with the following command: `sudo a2ensite ItemApplication`
4. Restart Apache: `sudo /etc/init.d/apache2 restart`

## Create the .wsgi File
1. Inside ItemCatalog, create a vagrant folder: `sudo mkdir vagrant`
2. Create the .wsgi file inside folder /var/www/itemCatalog/vagrant:
3. Inside vagrant folder, create another folder: `sudo mkdir ItemCatalog`
4. Move all project to this folder
5. Add the following lines of code to the itemApplication.wsgi file:
	
	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/ItemCatalog/vagrant")

	from ItemCatalog import app as application
	application.secret_key = 'Add your secret key'
	```

6. Edit __init__.py and put the client.json with hardcoded path

## Restart Apache
1. Restart Apache `sudo /etc/init.d/apache2 restart `
