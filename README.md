Contents:
A README file is included in the GitHub repo containing the following information: IP address, URL, summary of software installed, summary of configurations made, and a list of third-party resources used to completed this project.


IP address: 35.197.118.219 
ssh port: 2200
URL: http://shmenu.tk/



1- Install & upgrade packages:
 -  `apt-get update`
 - `sudo apt-get upgrade`
 
2- Change the SSH port from 22 to 2200**
    
    - $ sudo nano /etc/ssh/sshd_config
      Edit the port from 22 to 2200
      $ sudo service ssh restart 
    - disable ssh login for root user
      $ sudo nano /etc/ssh/sshd_config
      Edit permitRottLogin from yes to no

3- Configure the Firewall (UFW)
    
    -  $ sudo ufw status -> inactive
    -  Block general incoming 
      $ sudo ufw default deny incoming
    - Allow general outgoing.
      $ sudo ufw default allow outgoing
    - Set out SSH port. 
       $ sudo ufw allow 2200/tcp
    - Set the apache port
       $ sudo ufw allow 80/tcp
    - Set the ntp
      $ sudo ufw allow 123/udp
    - enable the firewall
     $ sudo ufw enable

4 - Create a new user *grader* and give this user sudo permissions.

- Add a new user called *grader*
 `$ sudo adduser grader`.
- Create a new file under the suoders directory
  $ sudo nano /etc/sudoers.d/grader
- Edit the file with "grader ALL=(ALL:ALL) ALL"


 5 - Create ssh key pair for *grader* user

 - Generate an encryption key on the local machine:
  $ ssh-keygen
 -Login as grader user and create "authorized_keys" file
  $ touch ~/.ssh/authorized_keys
 - Copy the content of the *id_rsa.pub* file from the local    machine to the */home/grader/.ssh/authorized_keys* 
 - change some permissions:
	- $ sudo chmod 700 /home/grader/.ssh
	- $ sudo chmod 644 /home/grader/.ssh/authorized_keys
 - change the owner from to grader
  $ sudo chown -R grader:grader /home/grader/.ssh
 - Confirm that you can login without password
  $ ssh -i ~/.ssh/id_rsa grader@35.190.163.205 -p 2200
 - $ sudo nano /etc/ssh/sshd_config.
    Edit PasswordAuthentication from yes to *no*.
 - $ sudo service ssh restart


6- configure local timezone to UTC  
 $ dpkg-reconfigure tzdata


7- install and configure apache to serve a python mod_wsgi app
- $ sudo apt-get install apache2
- $ sudo apt-get install libapache2-mod-wsgi python-dev
- $ sudo a2enmod wsgi
- $ sudo service apache2 start


8- installing postgress
-  $ sudo apt-get install postgresql

- add user and make database
$ sudo -su postgres
$psql
>CREATE USER catalog WITH PASSWORD '1234';
>CREATE DATABASE catalogdb;
>GRANT ALL ON DATABASE catalogdb TO  catalog ;
>/q
$exit
```
- now change application connection to data base as like 
,,,
engine = create_engine('postgresql://catalog:1234@localhost/catalogdb')
,,,


9- configure mod-wsgi

- copy your project to /var/www/catalog
- create file catalog.wsgi

 $ sudo vim /var/www/catalog/catalog.wsgi

- insert in this file python code 
```
import sys


sys.path.insert(0, "/var/www/catalog/Item_Catalog")

from application import app as application
```
  
- touch file at /etc/apache2/site-avaliable/
``` 
sudo vim /etc/apache2/site-avaliable/catalog.conf
```
- configure mod_wsgi by insert this text on catalog.conf
```
<VirtualHost *:80>
    
    WSGIDaemonProcess catalog user=www-data group=www-data threads=5
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog>
        Require all granted
    </Directory>
    alias /static /var/www/catalog/Item_Catalog/static
    <Directory /var/www/flaskapp/production/static/>
        Require all granted
    </Directory>
</VirtualHost>
```
- Enable the virtual host with the following command
```
sudo a2ensite catalog
sudo service apache2 reload
```

Thanks for Udacity full stack web developer nanodegree

