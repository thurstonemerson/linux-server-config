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

1. Create a new user named grader and grant this user sudo permissions.
	1. Add user grader
    	```
		$ sudo adduser grader
    	```
    
    1. Grant grader sudo permissions
    	Add text 'grader ALL=(ALL) NOPASSWD:ALL' to graders file in sudoers directory
        ```
		$ sudo cp /etc/sudoers.d/vagrant /etc/sudoers.d/grader
		$ sudo nano /etc/sudoers.d/grader
    	```
    	
    1. Allow user grader to log in via ssh to the server
		Create grader's .ssh folder and change the owner/group to grader
		
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
    1. Force ssh log in with keypair
     	Change PasswordAuthentication to no and restart the ssh service
		```
		$ sudo nano /etc/ssh/sshd_config
		$ sudo service ssh restart
    	```

    1. Test grader ssh login
    	```
		$ ssh grader@52.32.98.214 -i ~/.ssh/linuxCourse
    	```	

1. Update all currently installed packages.
     1. Update the available package list
     
    	```
		$ sudo apt-get update
    	```	
     1. Upgrade the available packages
     
    	```
		$ sudo apt-get upgrade
    	```	
     1. Automatically remove unneeded packages
     
    	```
		$ sudo apt-get autoremove
    	```	

1. Configure the local timezone to UTC.
     1. Check what the current server timezone is with the date command
     
    	```
		$ date
    	```
    	
    	This told me that the date was already UTC time. If it had not been UTC, I could have
    	changed it with the command 'sudo dpkg-reconfigure tzdata'
		