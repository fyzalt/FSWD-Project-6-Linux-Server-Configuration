## Linx-Server-Configuration
### About this Project

A baseline installation of a Linux server and prepare it to host [Item Catalog App](https://github.com/Fyzeey/FSWD-Project-4-Item-Catalog) from previous project.

### Server Details

IP address: 50.112.78.33

Port: 2200

URL: http://ec2-50-112-78-33.us-west-2.compute.amazonaws.com

Username: grader

### Configuration Steps
#### Step 1 -SSH into server as the user ubuntu

1. Download Private Key from https://lightsail.aws.amazon.com/ls/webapp/account/keys  
2. Move the private key file into the folder ~/.ssh 	
3. Change the permissions of the key, only for the owner read and write

		$ chmod 600 ~/.ssh/private_key.pem
		
4. Log in the Development Environment

		$ ssh -i ~/.ssh/private_key.pem ubuntu@50.112.78.33

#### Step 2 -User Management

1. Create a new user name grader

		$ sudo adduser grader

2. Grant **grader** the permission to sudo. In the /etc/sudoers.d create a new file called grader
		
		$ sudo nano /etc/sudoers.d/grader        
		
	In the file put in:  `grader ALL=(ALL) NOPASSWD:ALL`, then save and exit.
