# udacity-linux-server-configuration

### Project Description

Take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

- IP address: 52.57.242.110

- Accessible SSH port: 2200

- Application URL: ec2-52-57-242-110.eu-central-1.compute.amazonaws.com

## DEscription
1. Create new user named grader and give it the permission to sudo
  - Run `$ sudo adduser grader` to create a new user 
  - Create a new file in the sudoers directory with `sudo nano /etc/sudoers.d/grader`
  - Add the following text `grader ALL=(ALL:ALL) ALL`
  
2. Update all currently installed packages
  - Download package lists with `sudo apt-get update`
  - Fetch new versions of packages with `sudo apt-get upgrade`

3. Change SSH port from 22 to 2200
  - Run `sudo vim /etc/ssh/sshd_config`
  - Change the port from 22 to 2200
  - Confirm by running `ssh -i D:/key -p 2200 grader@35.167.27.204`
  
4. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
  - `sudo ufw default deny incoming`
  - `sudo ufw default allow outgoing`
  - `sudo ufw allow 2200/tcp`
  - `sudo ufw allow www`
  - `sudo ufw allow ntp`
  - `sudo ufw allow ssh`
  - `sudo ufw enable`
  
5. Configure the local timezone to UTC
  - Run `sudo sudo unlink /etc/localtime` to unlink the file time 
  - Run `sudo ln -s /usr/share/zoneinfo/Etc/UTC /etc/localtim' to link files
  
6. Configure key-based authentication for grader user
  - `mkdir /home/grader/.ssh`
  - `vim /home/grader/.ssh/authorized_keys` and write inside your public key
  -`chmod 700 /home/grader/.ssh && chmod 644 /home/grader/.ssh/authorized_keys`

7. Disable ssh login for root user
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change `PermitRootLogin without-password` line to `PermitRootLogin no`
  - Restart ssh with `sudo service ssh restart`
  - Now you are only able to login using `ssh -i D:/key -p 2200 grader@35.167.27.20`
 
8. Install Apache
  - `sudo apt-get install apache2`

9. Install mod_wsgi
  - Run `sudo apt-get install libapache2-mod-wsgi python-dev`
  - Enable mod_wsgi with `sudo a2enmod wsgi`
  - Start the web server with `sudo service apache2 start`

  
10. Clone the Catalog app from Github
  - Install git using: `sudo apt-get install git`
  - `cd /var/www`
  - `sudo mkdir catalog`
  - Change owner of the newly created catalog folder `sudo chown -R grader:grader catalog`
  - `cd /catalog`
  - Clone your project from github `git clone https://github.com/rrjoson/udacity-item-catalog.git catalog`
  - Create a catalog.wsgi file, then add this inside:
  ```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")
  
  from catalog import app as application
  application.secret_key = 'supersecretkey'
  ```
  - Rename application.py to __init__.py `mv application.py __init__.py`
  
11. Install virtual environment
  - Install the virtual environment `sudo pip install virtualenv`
  - Create a new virtual environment with `sudo virtualenv venv`
  - Activate the virutal environment `source venv/bin/activate`
  - Change permissions `sudo chmod -R 777 venv`

12. Install Flask and other dependencies
  - Install pip with `sudo apt-get install python-pip`
  - Install Flask `pip install Flask`
  - Install other project dependencies `sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils`

13. Update path of client_secrets.json file
  - `nano __init__.py`
  - Change client_secrets.json path to `/var/www/catalog/catalog/client_secrets.json`
  
14. Configure and enable a new virtual host
  - Run this: `sudo nano /etc/apache2/sites-available/catalog.conf`
  - Paste this code: 
  ```
  <VirtualHost *:80>
      ServerName 52.57.242.110
      ServerAlias ec2-52-57-242-110.eu-central-1.compute.amazonaws.com
      ServerAdmin admin@52.57.242.110
      WSGIDaemonProcess Catalog  user=grader group=grader  python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
      WSGIScriptAlias / /var/www/catalog/catalog.wsgi
      <Directory /var/www/catalog/catalog/>
          Order allow,deny
          Allow from all
      </Directory>
      ErrorLog ${APACHE_LOG_DIR}/error.log
      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```
  - Enable the virtual host `sudo a2ensite catalog`
  - Change THe Group and user Of Apache TO grader `sudo vim /etc/apache2/envvars`
  - change this lines to 
  ```
  #export APACHE_RUN_USER=www-data
  #export APACHE_RUN_GROUP=www-data
  export APACHE_RUN_USER=grader
  export APACHE_RUN_GROUP=grader
  ```
  - `sudo service apache2 restart`
15. Install and configure PostgreSQL
  - `sudo apt-get install libpq-dev python-dev`
  - `sudo apt-get install postgresql postgresql-contrib`
  - `sudo su - postgres`
  - `psql`
  - `CREATE USER catalog WITH PASSWORD 'password';`
  - `ALTER USER catalog CREATEDB;`
  - `CREATE DATABASE catalog WITH OWNER catalog;`
  - `\c catalog`
  - `REVOKE ALL ON SCHEMA public FROM public;`
  - `GRANT ALL ON SCHEMA public TO catalog;`
  - `\q`
  - `exit`
