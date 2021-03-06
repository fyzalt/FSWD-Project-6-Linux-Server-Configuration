# Linux-Server-Configuration
## About this Project

A baseline installation of a Linux server and prepare it to host [Item Catalog App](https://github.com/Fyzeey/FSWD-Project-4-Item-Catalog) from previous project.

## Server Details

IP address: 50.112.78.33

Port: 2200

URL: http://ec2-50-112-78-33.us-west-2.compute.amazonaws.com

Username: grader

## Configuration Steps
### Step 1 -SSH into server as the user ubuntu

1. Download Private Key from https://lightsail.aws.amazon.com/ls/webapp/account/keys  
2. Move the private key file into the folder ~/.ssh 	
3. Change the permissions of the key, only for the owner read and write

		$ chmod 600 ~/.ssh/private_key.pem
		
4. Log in the Development Environment

		$ ssh -i ~/.ssh/private_key.pem ubuntu@50.112.78.33

### Step 2 -User Management

1. Create a new user name grader

		$ sudo adduser grader

2. Give **grader** the sudo access. In the /etc/sudoers.d create a new file called grader
		
		$ sudo nano /etc/sudoers.d/grader        
		
	In the file put in:  `grader ALL=(ALL) NOPASSWD:ALL`, then save and exit.

3. Generate the private and public key pairs on local machine 

		$ ssh-keygen
		
	See the content in the public key
	
		$ cat ~/.ssh/keyname.pub
	
	Copy all the content in the terminal
	In the Develop Environment, switch to user **grader** 
	
		$ sudo su - grader
	
	Create the following directory and file
	
		$ mkdir ~/.ssh
		$ nano ~/.ssh/authorized_keys
		
4. Paste the content of the puclic from your local machine to the ~/.ssh/authorized_keys file you just created, then change some permissions:

		$ sudo chmod 700 /home/grader/.ssh
		$ sudo chmod 644 /home/grader/.ssh/authorized_keys
		
5. Change the SSH port from 22 to 2200

		$ sudo nano /etc/ssh/sshd-config
		
	Find 'Port 22' and change it to 'Port 2200'
		
6. Disable remote login of root user

	In same file, change the value PermitRootLogin to no
		
7. Restart ssh service
	
		$ sudo service sshd restart

### Step 3 -Security

1. Uncomplicated Firewall Set up
	
	Allow incoming connections for SSH (port 2200), HTTP (port 80) and  NTP (port 123)
	
		$ sudo ufw allow 2200/tcp.
		$ sudo ufw allow 80/tcp.
		$ sudo ufw allow 123/udp.
		$ sudo ufw enable.
		
2. Key-based SSH authentication enforcement.

	PasswordAuthentication in the /etc/ssh/sshd-config file is already set to no by default	
	
### Step 4 -Update Applications

1. Update package source list:

		$ sudo apt-get update

2. Upgrade all the application:

		$ sudo apt-get upgrade
		
		
### Step 5 -Configure the local timezone to UTC

1. Open time configuration dialog 

		$ sudo dpkg-reconfigure tzdata
		
2. Follow the instruction in the terminal to set up time zone


### Step 6 -Install Apache, mod_wsgi and git

1. Install Apache

		$ sudo apt-get install apache2
		
2. Install mod_wsgi

		$ sudo apt-get install libapache2-mod-wsgi
		
3. Restart Apache

		$ sudo service apache2 restart
		
4. Install Git

		$ sudo apt-get install git
		
### Step 7 -Clone Item Catalog App from Github Repo 

1. Create a directory named catalog under /var/www.

		$ sudo mkdir catalog
		
2. Change the owner of catalog directory

		$ sudo chown -R grader:grader catalog
		
3. Clone catalog repository into catalog directory

		$ git clone https://github.com/Fyzeey/FSWD-Project-4-Item-Catalog.git
		
	Make your repository name does not contain "-", it will cause invalid syntax error.
	I renamed my directory from "Item-Catalog" to "catalog"
	
		$ mv Item-Catalog catalog
	
		
4. Create catalog.wsgi file in the same directory and fill in with following content

```
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/catalog/catalog")

	from catalog import app as application
	application.secret_key = 'Secret_Key(fill your own secret here)'
	
```

### Step 8 -Install virtualenv and other necessary applications

1. Install pip , virtualenv (in /var/www/catalog)

		$ sudo apt-get install python-pip
		$ sudo pip install virtualenv
		
2. Create a new virtual environment
		
		$ sudo virtualenv venv
		
3. Activate the virtual environment and change venv directory's permission.

		$ source venv/bin/activate
		$ sudo chmod -R 777 venv
		
4. Install other necessary application

		$ sudo pip install flask 
		$ sudo pip install psycopg2 
		$ sudo pip install sqlalchemy
		$ sudo pip install oauth2client
		$ sudo pip install requests
		$ sudo pip install httplib2
		
### Step 9 -Configure Apache2 to serve the app

1. Create a virtual host configuration file 
	
		$ sudo nano /etc/apache2/sites-available/catalog.conf
		
2. Filled in the file with the following content

```
	<VirtualHost *:80>
	    ServerName 50.112.78.33
	    ServerAlias ec2-50-112-78-33.us-west-2.compute.amazonaws.com
	    ServerAdmin fanyinze@gmail.com
	    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
	    WSGIProcessGroup catalog
	    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
	    <Directory /var/www/catalog/catalog/>
		Order allow,deny
		Allow from all
	    </Directory>
	    Alias /static /var/www/catalog/catalog/static
	    <Directory /var/www/catalog/catalog/static/>
		Order allow,deny
		Allow from all
	    </Directory>
	    ErrorLog ${APACHE_LOG_DIR}/error.log
	    LogLevel warn
	    CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>

```
3. Disable the default virtual host

		$ sudo a2dissite 000-default.conf
		
4. Enable the catalog app virtual host

		$ sudo a2ensite catalog-app.conf
		
### Step 10 -Install and configure PostgreSQL

1. Install PostgreSQL

		$ sudo apt-get install postgresql postgresql-contrib
		
2. Switch to user 'postgres' which is automatically created for you

		$ sudo su - postgres
		
3. Connect to database 

		$ psql
		
4. Create a new role

		# CREATE USER catalog WITH PASSWORD 'password';
		# ALTER USER catalog CREATEDB;
		
5. Create database named catalog

		# CREATE DATABASE catalog WITH OWNER catalog;
		
6. Connect to databse and revike all rights

		# \c catalog
		# REVOKE ALL ON SCHEMA public FROM public;
		
7. Lock down the permissions to only let catalog role create tables	

		# GRANT ALL ON SCHEMA public TO catalog;
		
8. Exit PostgreSQL

9. Update database connection in catalog application(catalog.py database_populate.py etc)

```
	engine = create_engine('postgresql://catalog:password@localhost/catalog')
```
10. Specify the absolute path for client_secrets.json file in application

Change
    
```
	CLIENT_ID = json.loads(open('client_secrets.json', 'r').read())['web']['client_id']
	
```

to
	
```
	CLIENT_ID = json.loads(open(r'/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']
	
```

Also change
	
```
	oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='')
```

to
	
```
	oauth_flow = flow_from_clientsecrets(r'/var/www/catalog/catalog/client_secrets.json', scope='')
```

11. Setup and populate databse

```
	python /var/www/catalog/catalog/database_setup.py
	python /var/www/catalog/catalog/database_populate.py
```

### Step 11 -Update OAuth authorized JavaScript origins and redirect URI

1. Go to google developer console and add new url into Authorized JavaScript origins and Authorized redirect URIs

2. Download new json file, copy its content and paste into your previous old client_secrets.json which located at /var/www/catlog/catalog/client_secrets.json

### Step 12 -Restart Apache

1. Run the following command

		 $ sudo service apache2 restart 
		 
## Something Extra

1. Automatic Updates
	Install unattended-upgrades package
	
		$sudo apt-get install unattended-upgrades
		
	Enable unattended-upgrades
	
		$sudo dpkg-reconfigure --priority=low unattended-upgrades
		
	Edit /etc/apt/apt.conf.d/10periodic config file as follows
	```
	APT::Periodic::Update-Package-Lists "1";
	APT::Periodic::Download-Upgradeable-Packages "1";
	APT::Periodic::AutocleanInterval "7";
	APT::Periodic::Unattended-Upgrade "1";
	```
	The package list will update and upgrade on a daily basis.
	
## Third Party Resource

Udacity Forum

https://discussions.udacity.com/t/client-secret-json-not-found-error/34070

https://discussions.udacity.com/t/internal-server-error-after-changing-to-catalog-wsgi/248674/2

https://discussions.udacity.com/t/oath-error-origin-mismatch/221485/19

Flask Documentation

http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/

Ubuntu

https://help.ubuntu.com/community/AutomaticSecurityUpdates

Github Repo

https://github.com/SteveWooding/fullstack-nanodegree-linux-server-config

https://github.com/iliketomatoes/linux_server_configuration/blob/master/README.md
