# Linux Server Configuration (Ubuntu) for Amazon AWS

The following instructions detail the configuration of an [Ubuntu](http://www.ubuntu.com/) server (Amazon AWS Ec2 instance) including installation of
applications and securing the firewall. Web application [TheCatalog](https://github.com/thurstonemerson/the-catalog) is installed to an [Apache](https://httpd.apache.org/) web server. 

## Completed Environment

The web application TheCatalog can be accessed via the following URLS:

- http://52.32.98.214/ 
- http://ec2-52-32-98-214.us-west-2.compute.amazonaws.com/

It is possible to ssh into the server as user 'grader' after downloading the private key. This should
be moved into the .ssh directory:

```
$ mv ~/Downloads/udacity_key.rsa ~/.ssh/
$ ssh -p 2200 -i ~/.ssh/udacity_key.rsa grader@52.32.98.214
```

## Basic Configuration

1. **Create a new user named grader and grant this user sudo permissions.**
	
	Add user grader
	
    ```
	$ sudo adduser grader
    ```
    
    Grant grader sudo permissions by adding  text 'grader ALL=(ALL) NOPASSWD:ALL' to graders file in sudoers directory
    ```
	$ sudo cp /etc/sudoers.d/vagrant /etc/sudoers.d/grader
	$ sudo nano /etc/sudoers.d/grader
    ```
    	
    Allow user grader to log in via ssh to the server. Create grader's .ssh folder and change the owner/group to grader
	```
	$ mkdir .ssh
	$ chgrp grader .ssh
	$ chown grader .ssh
    ```
    	
    Copy root's public key to grader's .ssh folder and change the owner/group to grader, and set the permissions
    ```
	$ cp /root/.ssh/authorized_keys /home/grader/.ssh/authorized_keys
	$ chown grader authorized_keys
	$ chgrp grader authorized_keys
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys
    ``` 	
    
 	Force ssh log in with keypair by changing PasswordAuthentication to no and restart the ssh service
	```
	$ sudo nano /etc/ssh/sshd_config
	$ sudo service ssh restart
    ```

 	Test grader ssh login
    ```
	$ ssh grader@52.32.98.214 -i ~/.ssh/linuxCourse
    ```	

1. **Update all currently installed packages.**

    Update the available package list
     
    ```
	$ sudo apt-get update
    ```	
    
    Upgrade the available packages
     
    ```
	$ sudo apt-get upgrade
    ```	
    
    Automatically remove unneeded packages
     
    ```
	$ sudo apt-get autoremove
    ```	

1. **Configure the local timezone to UTC.**

    Check what the current server timezone is with the date command
     
    ```
	$ date
    ```
    	
    This told me that the date was already UTC time. If it had not been UTC, I could have
    changed it with the command 'sudo dpkg-reconfigure tzdata'
    	
## Configure the firewall

1. **Change the SSH port from 22 to 2200**

	```
	$ sudo nano /etc/ssh/sshd_config
	$ sudo service ssh restart
	```
	
1. **Configure firewall to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)**

	```
	$ sudo ufw status
	$ sudo ufw default deny incoming
	$ sudo ufw default allow outgoing
	$ sudo ufw allow ssh
	$ sudo ufw allow 2200/tcp
	$ sudo ufw allow 80/tcp
	$ sudo ufw allow ntp
	$ sudo ufw show added
	$ sudo ufw enable
	$ sudo ufw status
	```
	
## Install TheCatalog application and configure

1. **Install Apache webserver**

	Install necessary modules for apache, enable mod wsgi 
	```
	$ sudo apt-get install apache2
    $ sudo apt-get install libapache2-mod-wsgi
    $ sudo a2enmod wsgi
	```

1. **Install and configure PostgreSQL**

	Install necessary modules for PostgreSQL
	```
	$ sudo apt-get install postgresql
	$ sudo apt-get install python-psycopg2
	$ sudo apt-get install libpq-dev
	```
	
	Setup postgres user and add a password
	```	
	$ sudo -u postgres psql postgres
	$ \password
	```

1. **Install and configure Git, Python and virtualenv**

    ```
	$ sudo apt-get install python-pip python-dev build-essential 
	$ sudo pip install --upgrade pip 
	$ sudo pip install --upgrade virtualenv
	```
	
1. **Setup the virtual environment for TheCatalog application**

	```
	$ sudo pip install virtualenvwrapper
	$ mkdir ~/virtualenvs
	$ echo "export WORKON_HOME=~/virtualenvs" >> ~/.bashrc
	$ echo "source /usr/local/bin/virtualenvwrapper.sh" >> ~/.bashrc
	$ echo "export PIP_VIRTUALENV_BASE=~/virtualenvs" >> ~/.bashrc
	$ source ~/.bashrc
	$ mkvirtualenv the-catalog
	```
	
1. **Install TheCatalog application**

	Clone TheCatalog application from github
	
	```
	$ git clone https://github.com/thurstonemerson/the-catalog.git
	```
	
	Install the requirements for the catalog application
	
	```
	$ pip install -r requirements.txt
	```
	
1. **Setup PostgreSQL database for the application**

	Run the database setup script to create the database and unique catalog user for connecting to the database

    ```
	$ sudo su postgres
	$ psql 
	$ \i catalog_create_database.sql
    ```
    
    Seed the database with the provided python script

    ```
	$ python catalog_seed_database.py
    ```
    
1. **Copy the application to the apache www directory**

	Copy both the application files and the python virtual environment
    ```
	$ sudo cp -avfr ~/the-catalog /var/www/the-catalog
	$ sudo cp -avfr ~/virtualenvs/the-catalog /var/www/the-catalog/env
    ```
    
    Remove the .git directory
    
    ```
	$ cd /var/www/the-catalog
	$ sudo rm -rf .git
    ```
    
    Configure the application to save files in the correct directory by modifying the UPLOAD_FOLDER parameter to '/var/www/the-catalog/files'

    ```
	$ sudo nano /var/www/thecatalog/config.py
    ```
    
    Make sure that the apache user www-data has permission to save files in the specified location
    
    ```
	$ sudo chown www-data:www-data /var/www/the-catalog/files
    ```
    
1. **Configure Apache webserver to serve TheCatalog application**

	Move the apache configuration file found the TheCatalog source
	
	```
	$ sudo mv /var/www/thecatalog/the-catalog.conf /etc/apache2/sites-available/the-catalog.conf
    ```
    
    This file contains the following configuration. Note the addition of a server alias pointing
    to the amazonaws url and the WSGIDaemonProcess has a python path pointing to the virtual environnment.
    For this application it is required to configure apache to pass on the authentication header (WSGIPAssAuthorization).
    
    ```
    <VirtualHost *:80>
		ServerName 52.32.98.214
		ServerAdmin admin@52.32.98.214
		ServerAlias http://ec2-52-32-98-214.us-west-2.compute.amazonaws.com
    	WSGIDaemonProcess the-catalog python-path=/var/www/the-catalog:/var/www/the-catalog/env/lib/python2.7/site-packages
    	WSGIProcessGroup the-catalog 
		WSGIPAssAuthorization On
		WSGIScriptAlias / /var/www/the-catalog/thecatalog.wsgi
		<Directory /var/www/the-catalog/catalog/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/the-catalog/catalog/static
		<Directory /var/www/the-catalog/catalog/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```
	
	Note that thecatalog.wsgi is already found in the catalog app source and contains the following configuration
	
	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/the-catalog/")

	from catalog import app as application
	```

	Enable the apache configuration file found the TheCatalog source and disable the default configuration
	
	```
	$ sudo a2ensite the-catalog
	$ sudo a2dissite 000-default.conf
	$ sudo service apache2 reload
    ```
    
1. **Configure hosts file for Amazon AWS**

	Add the amazon url to the hosts file adding the line '127.0.0.1 localhost ec2-52-32-98-214.us-west-2.compute.amazonaws.com'

	```
	$ sudo nano /etc/hosts
	```

1. **Configure 3rd party authentication**

	- Log into Google developer console and add authorized redirect URI and javascript origins http://ec2-52-32-98-214.us-west-2.compute.amazonaws.com
	- Log into Facebook apps and add site URL http://ec2-52-32-98-214.us-west-2.compute.amazonaws.com
	- Log into Twitter application management and add callback URL http://ec2-52-32-98-214.us-west-2.compute.amazonaws.com
		