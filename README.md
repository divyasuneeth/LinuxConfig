# Linux Server Configuration Project
In this project, you will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

## Requirements:
* The Linux distribution is Ubuntu 16.04 LTS.
* The virtual private server is Amazon Lighsail.
* The web application is my Item Catalog project created earlier in this Nanodegree program.
* The database server is PostgreSQL.

# Get your server

* Login into Amazon Lightsail using an Amazon Web Services account.
* Once you are login into the site, click Create instance.
* Choose Linux/Unix platform, OS Only and Ubuntu 16.04 LTS.
* Choose an instance plan.
* Keep the default name provided by AWS or rename your instance.
* Click the Create button to create the instance.
* Wait for the instance to start up.

# SSH into your server

* From the Account menu on Amazon Lightsail, click on SSH keys tab and download the Default Private Key.
* Move this private key file named LightsailDefaultPrivateKey-.pem into the local folder ~/.ssh and rename it LightsailKey.pem.
* In your terminal, type: ``chmod 600 ~/.ssh/LightsailKey.pem``.
* To connect to the instance via the terminal: ``ssh -i ~/.ssh/LightsailKey.pem ubuntu@34.220.136.95``, where 34.220.136.95 is the public IP address of the instance.

# Secure your server.
* Update all currently installed packages.
  * ``sudo apt-get update``
  * ``sudo apt-get upgrade``
* Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.
  * Run ```sudo nano /etc/ssh/sshd_config```
  * Confirm by running ```ssh -i ~/.ssh/LightsailKey.pem -p 2200 ubuntu@34.220.136.95```
* Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
  * ``sudo ufw status ``
  * ``sudo ufw default deny incoming``
  * ``sudo ufw default allow outgoing``
  * ``sudo ufw allow 2200/tcp``
  * ``sudo ufw allow www ``
  * ``sudo ufw allow 123/udp``
  * ``sudo ufw deny 22  ``
  * ``sudo ufw enable ``
* Exit the SSH connection
* Click on the Manage option of the Amazon Lightsail Instance, navigate to the Networking tab, and then change the firewall configuration to match the internal firewall settings above.
* From your local terminal, run: ``ssh -i ~/.ssh/LightsailKey.pem -p 2200 ubuntu@34.220.136.95``

# Give grader access.

### Create a new user account named grader

While logged in as ubuntu, add user: ``sudo adduser grader``.
Enter a password (twice) and fill out information for this new user.

### Give grader the permission to sudo

* Edits the sudoers file: ``sudo visudo``.
* Search for the line that looks like this:

``root    ALL=(ALL:ALL) ALL``
* Below this line, add a new line to give sudo privileges to grader user.

``root    ALL=(ALL:ALL) ALL
grader  ALL=(ALL:ALL) ALL``
* Save and exit using CTRL+X and confirm with Y.

### Create an SSH key pair for grader using the ssh-keygen tool

* On the local machine:
* Run ssh-keygen
* Enter the file name for the key(i named it myLinuxProject) in the local directory ~/.ssh
* Enter in a passphrase(optional) twice. Two files will be created ( ~/.ssh/myLinuxProject and ~/.ssh/myLinuxProject.pub)
* Run ``cat ~/.ssh/myLinuxProject.pub`` and copy the contents of the file
* Log in to the grader's virtual machine
   * ``ssh -i ~/.ssh/myLinuxProject -p 2200 grader@34.220.136.95``
* On the grader's virtual machine:
* Create a new directory called ~/.ssh (mkdir .ssh)
* Run ``sudo nano ~/.ssh/authorized_keys ``and paste the previously copied content (myLinuxProject.pub)into this file, save and exit.
* change owner and group of authorized_keys
  * `` sudo chgrp grader authorized_keys``
  * ``sudo chown grader authorized_keys``
* Give the permissions: ``chmod 700 .ssh ``and ``chmod 644 .ssh/authorized_keys``
* Check in ``cat /etc/ssh/sshd_config `` file if PasswordAuthentication is set to no
* Restart SSH: sudo service ssh restart

On the local machine, run: ``ssh -i ~/.ssh/myLinuxProject -p 2200 grader@34.220.136.95``


# Configure the local timezone to UTC
* While logged in as grader, configure the time zone ``sudo dpkg-reconfigure tzdata``


# Prepare to deploy your project.

### Install and configure Apache to serve a Python mod_wsgi application

* While logged in as grader, install Apache: ``sudo apt-get install apache2``.
    * Enter public IP of the Amazon Lightsail instance into browser.
    If Apache is working
* Install mod_wsgi ``sudo apt-get install libapache2-mod-wsgi python-dev``
* Enable mod_wsgi with ``sudo a2enmod wsgi``
* Start the web server with sudo service apache2 start
* Install and configure PostgreSQL
  * While logged in as grader, install PostgreSQL: ``sudo apt-get install postgresql``.
* PostgreSQL should not allow remote connections. In the ``cat /etc/postgresql/9.5/main/pg_hba.conf ``

* Switch to the postgres user: sudo su - postgres.
* Open PostgreSQL interactive terminal with psql.
* Create the catalog user with a password and give them the ability to create databases:

``postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
postgres=# ALTER ROLE catalog CREATEDB;
CREATE DATABASE catalog OWNER catalog
Exit psql: \q.``
* Run ``python /var/www/catalog/catalog/database_setup.py``  to setup the database.
* Run ``python /var/www/catalog/catalog/itemsforcatalog.py`` to populate the tables.

* While logged in as grader, install git: ``sudo apt-get install git``

# Deploy the Item Catalog project

* Clone and setup the Item Catalog project from the GitHub repository
* create /var/www/catalog/ directory.
* Change to that directory and clone the catalog project:
git clone https://github.com/divyasuneeth/CatalogAPI.git catalog

* From the /var/www directory, change the ownership and group of the catalog directory to grader using: ``sudo chown grader catalog
sudo chgrp grader catalog``.
* Change to the /var/www/catalog/catalog directory.
* Rename the application.py file to \__init__.py using: ``mv application.py __init__.py``.

* In \__init__.py, replace line:

``app.run(host='0.0.0.0', port=8000, threaded=False)`` to ``app.run()``
* In database_setup.py, replace line :

``engine = create_engine("sqlite:///categorywithusers.db")`` to
``engine = create_engine('postgresql://catalog:password@localhost/catalog')``

### Install the virtual environment and dependencies
  * Change to the ``cd /var/www/catalog directory``
  * Install the virtual environment ``sudo pip install virtualenv``.
  * Create a new virtual environment with ``sudo virtualenv venv``
  * Activate the virtual environment ``source venv/bin/activate``
  * Change permissions ``sudo chmod -R 777 venv``

### Install Flask and other dependencies
  * ``pip install httplib2
pip install requests
pip install --upgrade oauth2client
pip install sqlalchemy
pip install flask
sudo apt-get install libpq-dev
pip install psycopg2``

### Set up and enable a virtual host
* Create /etc/apache2/sites-available/catalog.conf and add the following lines to configure the virtual host:
`` `<VirtualHost *:80>
    ServerName 34.220.136.95
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    ServerAlias ec2-34-220-136-95.us-west-2.compute.amazonaws.com
    ServerAdmin admin@34.220.136.95
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
</VirtualHost> `` `

* Enable virtual host: ``sudo a2ensite catalog``. The following prompt will be returned:

``Enabling site catalog.
To activate the new configuration, you need to run:
  service apache2 reload``

* Reload Apache: ``sudo service apache2 reload``.


### Set up the Flask application

* Create /var/www/catalog/catalog.wsgi file add the following lines:


```import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'supersecretkey'
```

### Disable the default Apache site:
* `` sudo a2dissite 000-default.conf``.
The following prompt will be returned:
``Site 000-default disabled.
To activate the new configuration, you need to run:
  service apache2 reload``

# View the Application
* Restart Apache: ``sudo service apache2 restart``.
* Open your browser to http://34.220.136.95 or http://ec2-34-220-136-95.us-west-2.compute.amazonaws.com

# Issues Encountered
* password authentication failed : this error was since catalog db user had a different password.
* Styling was lost after deploying the project on to the server.
  * solved by disabling the default apache site
  * Disable the default Apache site
    `` sudo a2dissite 000-default.conf``.

### To check error.log
sudo tail /var/log/apache2/error.log
