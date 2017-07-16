# Linx-Server-Configuration
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

