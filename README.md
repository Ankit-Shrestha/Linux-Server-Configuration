# Linux Server Configuration

This Project baselines the installation of a Linux distribution on a virtual machine and prepare it to host web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.


# Requirements

Knowledge of Linux, Database using PostgreSQL, Amazon lightsail service, A working python based application using flask framework, sqlAlchemy for ORM. Make sure the application runs on Mac or windows before configuring it into Linux. Below, are the steps to configure the Linux server.

---
# Address, IP, and SSH Port
**Project: ec2-35-183-99-138.ca-central-1.compute.amazonaws.com** <br />
**Public IP Address:** 35.183.99.138 <br/ >
**SSH Port:** 2200

---
## Getting Started

**Steps for Linux Server Configuration**
---
**1. Create a Amazon lightsail account for hosting the application. https://aws.amazon.com/lightsail/**

- Get the private key from the dashboard after successfully creating a amazon lightsail instance for key based authentication and save it to ~/.ssh folder where ~ is environment home directory. save the file as udacity_key.pem (free to choose a name).

**2. Launch the VM using SSH to the recently created amazon instance.** 

- First, give permission only to the owner to read and write the private key
``chmod 600 ~/.ssh/udacity.key.pem ``

- SSH to the instance
``ssh -i ~/.ssh/udacity_key.pem root@public IP``

**3. Create new user**

- Add new user grader 
 ``sudo adduser grader``

- grader should have sudo access
``sudo nano /etc/sudoers.d/grader``
Add ``grader ALL=(ALL:ALL) ALL`` in the file.
- Prevent "unable to resolve host error"
``sudo nano /etc/hosts``
Add ``127.0.1.1 ip-XX-XX-XX-XX`` to the file.

**4. Key-based authentication for user grader.** <br />
The public key for the instance is in the server instance.  Access it through ``cat /.ssh/authorized_keys`` while logged in as root. Now, we have to copy the public key to the grader user so that we can access grader user using the private key we save earlier in our local computer.

- Copy the public key to ``sudo nano /home/grader/.ssh/authorized_keys``. There wont be a ``.ssh`` folder for grader initially. so create the ``.ssh`` folder and ``sudo touch authorized_keys`` and ``sudo nano authorized_keys`` and copy the whole public key. Now, we can access grader user using the private key ``udacity_key.pem`` saved in our local computer.

**5. Remove passwordAuthentication. Allow only key-based.**

-``sudo nano /etc/ssh/sshd_config``. edit the PasswordAuthentication to no. Save and Exit. then, ``sudo service ssh restart`` to save all the changes.

**6. Change the SSH port 22 to 2200**

- ``sudo nano /etc/ssh/sshd_config``. Find Port 22 and change it to 2200. For this to change, make sure you open that port on the amazon lightspace networking section. 
- ``sudo servie ssh restart``.

**7. Disable login for root user**

- In ``sudo nano /etc/ssh/sshd_config`` find PermitRootLogin and change it to no.
- ``sudo service ssh restart``
- ``exit``
- ``ssh -i ~/.ssh/udacity_key.pem -p 2200 grader@PUBLIC IP`` to login as grader.

**8. Configure local timezone to UTC**

- ``sudo timedatectl set-timezone UTC``
- ``sudo apt-get install ntp``

**9. Update installed Packages**

- ``sudo apt-get update``
- ``sudo apt-get upgrade``

**10. Enable Firewall**

- ``sudo ufw default deny incoming``
- ``sudo ufw defauly allow outgoing``
- ``sudo ufw allow 2200/tcp``
- ``sudo ufw allow www``
- ``sudo ufw allow ntp``
- ``sudo ufw enable``

**11. Install and configure Apache2, mod-wsgi**


- ``sudo apt-get install apache2 libapache2-mod-wsgi`` 
- ``sudo a2enmod wsgi`` to enable mod_wsgi.

**12. Install and Configure PostgreSQL**

- ``sudo apt-get install libpq-dev python-dev``
- ``sudo apt-get install postgresql postgresql-contrib``
- ``sudo su - postgres``
- ``psql``
- ``CREATE USER catalog WITH PASSWORD '#######';``
- ``CREATE DATABASE catalog with OWNER catalog;``
- ``\c catalog``
- ``REVOKE ALL ON SCHEMA public FROM public;``
- ``GRANT ALL ON SCHEMA public to catalog;``
- ``\q`` and ``exit``

Now, in the python files for the application, engine should be modified to 
``engine = create_engine('postgresql://catalog:passwordCreated@localhost/catalog)``

**13. Install Flask and other dependencies**
- ``sudo apt-get install python-pip``
- ``sudo apt-get install git``
- ``sudo pip install Flask``
- ``sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils``
- ``sudo pip install requests``
make sure to get the correct dependencies and packages based on the version of python that you are using. Please refer the link https://packages.ubuntu.com/) .

**14. Clone the app from Github.**

 - ``sudo mkdir /var/www/catalog``
- ``sudo chown -R grader:grader /var/www/catalog``
- ``git clone ''repository URL'' catalog``
- ``cd catalog``
- ``touch project.wsgi && nano project.wsgi`` <br/><br/>
-  edit the file as 
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"var/www/catalog/")
from project import app as application
```
save and exit. Run the database_setup and populate the database:
``python3 database_setup.py``
``python3 lotsofmenus.py``

**15. Edit the default Virtual file**

``sudo nano /etc/apache2/sites-available/000-default.conf``
```
<VirtualHost *:80>
  ServerName XX.XX.XX.XX
  ServerAdmin XXXXX
  WSGIScriptAlias / /var/www/catalog/bookCatalog.wsgi
  <Directory /var/www/catalog/>
      Order allow,deny
      Allow from all
  </Directory>
</VirtualHost>
```

**16. Restart Apache service**

``sudo service apache2 restart``




