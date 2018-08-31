# Udacity Linux Server-Configuration Project

Final project for the Udacity Web Developer Nanodegree. For this project a Linux virtual machine will be configured to host the Item Catalog website done in a previous project.

IP Address: http://35.163.181.7
Link: http://ec2-35-163-181-7.us-west-2.compute.amazonaws.com


## How to SSH into the instance
Download Private Key from the SSH keys section in the Account section on Amazon Lightsail.

Move the private key file into the folder ~/.ssh (your home directory). For example, if you had downloaded the file to your Downloads folder, use the following command in your terminal: 
	
	mv ~/Downloads/Lightsail_key.pem ~/.ssh/

Open your terminal and type in:
	
	chmod 600 ~/.ssh/lightsail_key.rsa

In your terminal, type in: 
	
	ssh -i ~/.ssh/lightsail_key.rsa ubuntu@35.163.181.7


## Update and Upgrade installed packages

sudo apt-get update - to update the packages existing

sudo apt-get upgrade - to upgrade the installed packages


## Change the SSH port from 22 to 2200

Use: 
	sudo nano /etc/ssh/sshd_config

Change Port 22 to Port 2200. Press Ctrl X to exit, then Y. If faced with a screen asking how to save file, press enter and exit.

Reload SSH using:

	 sudo service ssh restart
	 logout

In the Amazon Web Services account head to the Networking tab and create a custom connection with the TCP protocol in port range 2200.

Reconnect through your personal terminal with the command:

	ssh -i ~/.ssh/lightrail_key.rsa ubuntu@35.163.181.7 -p 2200


## Configure the firewall

Check out firewall status (check to see if it works. Run before changing and after changing to confirm operation):
	
	sudo ufw status

Block all imconing connections coming in. Run:

	sudo ufw default deny incoming

Set utf to allow all outgoing connections. Run:
	
	sudo ufw default allow outgoing

Allow incoming SSH connections. Run:

	sudo ufw allow ssh 

Alloq tcp connections for port 2200. Run: 

	sudo ufw allow 2200/tcp 

Allow basic http server through firewall. Run:

	sudo ufw allow www 

Allow NTP through firewall. Run:
	
	sudo ufw allow 123/udp

Deny previous port 22 as the server now uses 2200. Run:
	
	sudo ufw deny 22 

Enable the ufw firewall. Run:

	sudo ufw enable

You can now go into your Amazon Lightsail account and go to thenetworking option and update the firewall configuration to match the changes created by the internal firewall settings above (ports 80(TCP), 123(UDP), and 2200(TCP). Simply create create custom ports and choose the appropriate item and number for the port. Relogin to your server with the following command:


## Change timezone to UTC
run sudo timedatectl set-timezone UTC


## Install unattended-upgrades to automate the update of the packages
	sudo apt install unattended-upgrades
	editor /etc/apt/apt.conf.d/50unattended-upgrades
	editor /etc/apt/apt.conf.d/20auto-upgrades


## Create a new user named grader 

Run the following command:
	
	sudo adduser grader

Create password for grade (password).
Switch to grade via: su - grader 
Endter the password (password)


## Give the grader user created sudo permissions

Head back to the primary user by typing: exit

	Run: sudo visudo

Add the following line after to the ones that look same :grader ALL=(ALL:ALL) ALL

Save and close the visudo file

To check whether the grader does have sudo permissions, su as grader (su - grader), enter the password, and run sudo -l;


## Allow for the grader to log in to the virtual machine

	Run ssh-keygen on the local machine

Choose a file name for the key pair (such as grader_key)

Enter in a passphrase twice (two files will be generated; the second one will end in .pub)

Log in to the virtual machine

Switch to grader's home directory, and create a new directory called .ssh (run mkdir .ssh)

Run touch .ssh/authorized_keys

On the local machine, run cat ~/.ssh/insert-name-of-file.pub

Copy the contents of the file, and paste them in the .ssh/authorized_keys file on the virtual machine by using:

	sudo nano .ssh/authorized_keys

Run chmod 700 .ssh on the virtual machine

Run chmod 644 .ssh/authorized_keys on the virtual machine

Make sure key-based authentication is forced (log in as grader, open the /etc/ssh/sshd_config file, and find the line that says, '# Change to no to disable tunnelled clear text passwords'; if the next line says, 'PasswordAuthentication yes', change the 'yes' to 'no'; save and exit the file; run sudo service ssh restart)

Log in as the grader using the following command:

ssh -i ~/.ssh/grader_key -p 2200 grader@35.163.181.7


## Diable Root Login

	sudo nano /etc/ssh/sshd_config

Change PermitRootLogin without-password to PermitRootLogin no.


## Install Apache

Run: 
	sudo apt-get install apache2 to install Apache

Check this by using the public IP of the Amazon Lightsail instance as as a URL in a browser; if Apache is working correctly, a page with apache title should load


## Install mod_wsgi
	
	sudo apt-get install libapache2-mod-wsgi python-dev

Make sure mod_wsgi is enabled by running:
	
	sudo a2enmod wsgi


## Install PostgreSQL

Run: 
	sudo apt-get install postgresql


## Install Python
Check if it's installed by running python
If not sudo apt-get python2 for python2 or sudo apt-get python3


## Create new PostgreSQL user named catalog and give permissions

	sudo su - postgres psql
	CREATE ROLE catalog WITH LOGIN;
	ALTER ROLE catalog CREATEDB;
	\password catalog (prompt for new password will come up_

Retrun by exiting psql (\q) and change to back to the root user. 
	
	sudo adduser catalog (create pw on prompt - password)
	sudo visudo
	catalog ALL=(ALL:ALL) ALL	

Add the above code below the previous ones for Root and Grader.


## Install git and clone the previous catalog item project
	
	in /var/www/ mkdir catalog
	sudo apt-get install git

clone the git repository in /var/www/catalog

	sudo git clone https://github.com/wintersnine/catalog.git

change ownership of the folder
	
	sudo chown -R ubuntu:ubuntu catalog/


## Add client_secrets.json

Generate a new project on the Google API console. Add in the new ip and amazon address(es) to authorized javascript. Download and change information in application.py file (renamed __init__.py)


## Set up vitual environment and install dependencies

	sudo apt-get install python-pip
	sudo apt-get install python-virtualenv
	. venv/bin/activate

While the virtual environment is active, install the following:

	pip install httplib2
	pip install requests
	pip install --upgrade oauth2client
	pip install sqlalchemy
	pip install flask
	sudo apt-get install libpq-dev 
	pip install psycopg2

Deactivate the virtual environment with the command : deactivate


## Set up and enable a virtual host

Create a file in /etc/apache2/sites-available/ called catalog.conf
	
	sudo nano /etc/apache2/sites-available/catalog.conf

Paste into the file:

<VirtualHost *:80>
		ServerName 35.163.181.7
		ServerAdmin xxxxxxxx@gmail.com
		WSGIScriptAlias / /var/www/catalog/catalog.wsgi
		<Directory /var/www/catalog/catalog/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		Alias /static /var/www/catalog/catalog/static
		<Directory /var/www/catalog/catalog/static/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

Save, exit, and run: 
	
	sudo service apache2 reload


## Generate a WSGI file

	sudo nano /var/www/catalog/catalog.wsgi

Paste into the file:

	activate_this = 	'/var/www/catalog/catalog/venv/bin/activate_this.py'
	execfile(activate_this, dict(__file__=activate_this))

	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/catalog/")

	from catalog import app as application
	application.secret_key = '...'

Restart apache with:

	sudo service apache2 restart


## Change database(s) in terminal to postgres:

	sudo nano (file name).py
	engine = create_engine('postgresql://catalog:password@localhost/catalog')


## Change directory ownership:

	sudo chown -R www-data:www-data catalog/


## Setup and populate database:
	
	. venv/bin/activate
	python lotsofgames.py (deactivate when finished)
	sudo service apache2 restar

Open project in browser and finish!!

IP Address: http://35.163.181.7
Link: http://ec2-35-163-181-7.us-west-2.compute.amazonaws.com


## Aknowledgements
Credit to Udacity for providing the framework and tutorials for getting started, youtube for providing helpful videos, andd Amazon for tutorials and their server.