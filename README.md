Udacity - Linux Server Configuration Project
An Udacity Full Stack Web Developer II Nanodegree Project.
About
This tutorial will guide you through the steps to take a baseline installation of a Linux server and prepare it to host your Web applications. You will then secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing Flask-based Web applications onto it.
In this project, I have set up an Ubuntu 18.04 image on a DigitalOcean droplet. The technical details of the server as well as the steps that have been taken to set it up can be found in the succeeding sections.
Technical Information About the Project
�	Server IP Address: 142.93.220.178
�	SSH server access port: 2200
�	SSH login username: grader
�	Application URL: http://142.93.220.178.xip.io
Steps to Set up the Server
1. Creating the RSA Key Pair
On your local machine, you will first have to set up the public and private key pair. This key pair will be used to authenticate yourself while securely logging in to the server via SSH. The private key will be kept with you in your local machine, and the public key will be stored in the server.
To generate a key pair, run the following command:
$ ssh-keygen
When it asks to enter a passphrase, you may either leave it empty or enter some passphrase. A passphrase adds an additional layer of security to prevent unauthorized users from logging in.
The whole process would look like this:
Generating public/private rsa key pair.
Enter file in which to save the key (/home/nikhil/.ssh/id_rsa): /home/nikhil/.ssh/udacity_project
Created directory '/home/nikhil/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/nikhil/.ssh/id_rsa.
Your public key has been saved in /home/nikhil/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:JnpV8vpIZqstoTm8aaDAqVmGSF/tGhtDfXAfL2fs/U8 nikhil@nikhil-VirtualBox
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|       . . .     |
|      o + o +    |
| .   o o = o =   |
|+.o o o S . = .  |
|+o+. =.= .   . . |
|o= o.oB.=       E|
|+   *=.= +     ..|
|   .oo.o+ .     o|
+----[SHA256]-----+
You now have a public and private key that you can use to authenticate. The public key is called id_rsa.pub and the corresponding private key is called id_rsa. The key pair is stored inside the ~/.ssh/ directory.
2. Setting Up a DigitalOcean Droplet
1.	Log in or create an account on DigtalOcean.
2.	Go to the Dashboard, and click Create Droplet.
3.	Choose Ubuntu 18.04 x64 image from the list of given images.
4.	Choose a preferred size. In this project, I have chosen the 1GB/1 vCPU/25GB configuration.
5.	In the section Add Your SSH Keys, paste the content of your public key, id_rsa.pub:
6.	Click Create to create the droplet. This will take some time to complete. After the droplet has been created successfully, a public IP address will be assigned. In this project, the public IPv4 address that I have been assigned is 142.93.220.178.
3. Logging In as root via SSH and Updating the System
3.1. Logging in as root via SSH
As the droplet has been successfully created, you can now log into the server as root user by running the following command in your host machine:
  $ ssh root@142.93.220.178
This will look for the private key in your local machine and log you in automatically if the corresponding public key is found on your server.
3.2. Updating the System
Run the following command to update the virtual server:
 # sudo apt update 
 # sudo apt upgrade
This will update all the packages. If the available update is a kernel update, you might need to reboot the server by running the following command:
# reboot
4. Changing the SSH Port from 22 to 2200
1.	Open the /etc/ssh/sshd_config file with nano or any other text editor of your choice:
2.	# nano /etc/ssh/sshd_config
3.	Find the line #Port 22 (would be located around line 13) and change it to Port 2200, and save the file.
4.	Restart the SSH server to reflect those changes:
5.	# service ssh restart
6.	To confirm whether the changes have come into effect or not, run:
7.	# exit
This will take you back to your host machine. After you are back to your local machine, run:
$ ssh root@142.93.220.178 -p 2200
You should now be able to log in to the server as root on port 2200. The -p option explicitly tells at what port the SSH server operates on. It now no more operates on port number 22.
5. Configure Timezone to Use UTC
To configure the timezone to use UTC, run the following command:
# sudo dpkg-reconfigure tzdata
It then shows you a list. Choose None of the Above and press enter. In the next step, choose UTC and press enter.
You should now see an output like this:
Current default time zone: 'Etc/UTC'
Local time is now:      Thu May 24 11:04:59 UTC 2018.
Universal Time is now:  Thu May 24 11:04:59 UTC 2018.
6. Setting Up the Firewall
Now we would configure the firewall to allow only incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123):
# ufw allow 2200/tcp
# ufw allow 80/tcp
# ufw allow 123/udp
To enable the above firewall rules, run:
# ufw enable
To confirm whether the above rules have been successfully applied or not, run:
# ufw status
You should see something like this:
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/udp                    ALLOW       Anywhere
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/udp (v6)               ALLOW       Anywhere (v6)
7. Creating the User grader and Adding it to the sudo Group
7.1. Creating the User grader
While being logged into the virtual server, run the following command and proceed:
  # adduser grader
The output would look like this:
  Adding user `grader' ...
  Adding new group `grader' (1000) ...
  Adding new user `grader' (1000) with group `grader' ...
  Creating home directory `/home/grader' ...
  Copying files from `/etc/skel' ...
  Enter new UNIX password:
  Retype new UNIX password:
  passwd: password updated successfully
  Changing the user information for grader
  Enter the new value, or press ENTER for the default
	  Full Name []: Grader
	  Room Number []:
	  Work Phone []:
	  Home Phone []:
	  Other []:
  Is the information correct? [Y/n]
Note: Above, the UNIX password I have entered for the user grader is, root.
7.2. Adding grader to the Group sudo
Run the following command to add the user grader to the sudo group to grant it administrative access:
  # usermod -aG sudo grader
8. Adding SSH Access to the user grader
To allow SSH access to the user grader, first log into the account of the user grader from your virtual server:
# su - grader
You should see a prompt like this:
grader@vandy-s-1vcpu-2gb-blr1-01:~$
Now enter the following commands to allow SSH access to the user grader:
$ mkdir .ssh
$ chmod 700 .ssh
$ cd .ssh/
$ touch authorized_keys
$ chmod 644 authorized_keys
After you have run all the above commands, go back to your local machine and copy the content of the public key file ~/.ssh/id_rsa.pub. Paste the public key to the server's authorized_keys file using nano or any other text editor, and save.
After that, run exit. You would now be back to your local machine. To confirm that it worked, run the following command in your local machine:
ssh grader@142.93.220.178 -p 2200
You should now be able to log in as grader and would get a prompt to enter commands.
Next, run exit to go back to the host machine and proceed to the following step to disable root login.
9. Disabling Root Login
1.	Run the following command on your local machine to log in as root in the server:
2.	$ ssh root@142.93.220.178 -p 2200
3.	After you are logged in, open the file /etc/ssh/sshd_config with nano:
4.	# nano /etc/ssh/sshd_config
5.	Find the line PermitRootLogin yes and change it to PermitRootLogin no.
6.	Restart the SSH server:
7.	# service ssh restart
8.	Terminate the connection:
9.	# exit
10.	After you are back to the host machine, when you try to log in as root, you should get an error:
10. Installing Apache Web Server to serve a Python mod_wsgi application
To install the Apache Web Server, run the following command after logging in as the grader user via SSH:
$ sudo apt update
$ sudo apt install apache2
To confirm whether it successfully installed or not, enter the URL http:// 142.93.220.178 in your Web browser:
If the installation has succeeded, you should see the Apache default Webpage.
Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
`sudo service apache2 restart`

11. Installing and Configuring PostgreSQL
11.1. Installing PostgreSQL
Install PostgreSQL:
$ sudo apt install postgresql-10
11.2. Configuring PostgreSQL
1.	Log in as the user postgres that was automatically created during the installation of PostgreSQL Server:
2.	$ sudo su - postgres
3.	Open the psql shell:
4.	$ psql
5.	This will open the psql shell. Now type the following commands one-by-one:
6.	postgres=# CREATE DATABASE catalog;
7.	postgres=# CREATE USER catalog;
8.	postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
Then exit from the terminal by running \q followed by exit.
12 Install git, clone and setup your Catalog App project
sudo apt-get install git
�	to install git
1.	Move into cd /var/www/
2.	Create a directory called FlaskApp and change the working directory to it:
3.	$ sudo mkdir FlaskApp
4.	$ cd FlaskApp/
5.	Clone this repository directory FlaskApp:
6.	$ sudo git https://github.com/Sharma-nikhil06/Item-Catalog-Project.git catalog
7.	Move inside the newly created directory:
8.	$ cd catalog/
9.	Checkout to the development branch:
10.	$ sudo git checkout development
11.	Install pip
      sudo apt-get install python-pip
7.	Install required packages:
8.	$ sudo pip install --upgrade Flask SQLAlchemy httplib2 oauth2client requests psycopg2 psycopg2-binary
13. Setting Up Virtual Hosts
1.	Run the following command in terminal to set up a file called FlaskApp.conf to configure the virtual hosts:
2.	$ sudo nano /etc/apache2/sites-available/FlaskApp.conf
3.	Add the following lines to it:
4.	
5.	<VirtualHost *:80>
6.	   ServerName 142.93.220.178
7.	   ServerAlias 142.93.220.178.xip.io
8.	   ServerAdmin grader@142.93.220.178
9.	   WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv
10.	WSGIProcessGroup catalog WSGIScriptAlias / /var/www/catalog/catalog.wsgi
11.	 <Directory /var/www/catalog/catalog/> 
12.	Order allow,deny Allow from all </Directory> Alias /static /var/www/catalog/catalog/static <Directory /var/www/catalog/catalog/static/> 
13.	Order allow,deny Allow from all </Directory> ErrorLogWSGIProcessGroupcatalogWSGIScriptAlias//var/www/catalog/catalog.wsgi
14.	<Directory/var/www/catalog/catalog/>
15.	Orderallow,denyAllowfromall</Directory>Alias/static/var/www/catalog/catalog/static
16.	<Directory/var/www/catalog/catalog/static/>
17.	Orderallow,denyAllowfromall
18.	</Directory>ErrorLog{APACHE_LOG_DIR}/error.log
19.	   LogLevel warn
20.	   CustomLog ${APACHE_LOG_DIR}/access.log combined
21.	</VirtualHost>
22.	
23.	Enable the virtual host:
24.	$ sudo a2ensite catalog
25.	Restart Apache server:
26.	$ sudo service apache2 restart
27.	Creating the .wsgi File
Apache uses the catalog.wsgi file to serve the Flask app. Move to the /var/www/catalog/ directory and create a file named catalog.wsgi with following commands:
$ cd /var/www/catalog/
$ sudo nano catalog.wsgi
Add the following lines to the catalog.wsgi file:
28.	import sys
29.	import logging
30.	logging.basicConfig(stream=sys.stderr)
31.	sys.path.insert(0, "/var/www/catalog/")
32.	
33.	from catalog import app as application
34.	application.secret_key = 'secret'
35.	Restart Apache server:
36.	$ sudo service apache2 restart
Now you should be able to run the application at http://142.93.220.178.xip.io.
Note: You might still see the default Apache page despite setting everything up correctly. To resolve it and see your Flask app running, run the following commands in order:
$ sudo a2dissite 000-default.conf
$ sudo service apache2 restart
Debugging
If you are getting an Internal Server Error or any other error(s), make sure to check out Apache's error log for debugging:
$ sudo cat /var/log/apache2/error.log
There might be some errors due to different version of python or flask as well. Look for the same version. Install the same version as used while making Item Catalog Project.
Do check that all the libraries used within your project are installed as well. Even if one is missing it would lead to error.
References
[1] https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04
[2] http://terokarvinen.com/2016/deploy-flask-python3-on-apache2-ubuntu
[3] https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
[4] https://github.com/kongling893/Linux-Server-Configuration-UDACITY

