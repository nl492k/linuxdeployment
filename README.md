## Linux Server Configuration:
- Udacity Full Stack Web Nano Degree
- Nathan Lott
- 5/15/2019

#### Specs
- Linux Distr: Ubuntu 16.04 LTS
- Virtual Server: Amazon Lightsail
- Application:  Item Catalog App
- Database: Catalog, Postgresql
- Local Machine - Windows 7 Professional 64 bit
- Python Version - Python 3

#### Address:
-Primary:
-http://3.15.31.49.xip.io/ ___(MUST USE TOP LEVEL DOMAIN TO SUPPORT GOOGLE LOGIN)___

-Secondary:
-http://3.15.31.49
-http://ec2-3-15-31-49.us-east-2.compute.amazonaws.com

___Facebook OAuth2.0 now only supports HTTPS___
- Functionality is still part of the app, but will not work until migrated to HTTPS


## Get Server 
#### 1| Start new Ubunto instance on Amazon Lightsail
- Login to Amazon Lightsail using AWS Account
- Click ```Create Instance```
- Select ```Linux/Unix```, ```OS Only```, and ```Ubunto 16.04 LTS```
- Create Amazon Lightsail Instance ```Ubunto 16.04LTS```
- Rename Instance
- Select Instance Plan
- Click ```Create```

#### 2| SSH Into Lightsail Ubunto Instance 
- Create SSH Private Key.
- Download Private Key File.
--_Reviewer will create key file using private key provided in reviewer notes._
- Move Private Key file to    ```~/.SSH``` directory on your local machine.
- Rename Private Key file to ```lightsail_key.rsa```.
- Using Terminal on local machine, change ```lightsail_key.rsa``` to ```-RW-------``` using ```chmod 600 ~/.ssh/lightsail_key.rsa```
- Using Terminal on local machine, connect to the Ubunto Instance using: ```ssh -i ~/.ssh/lightsail_key.rsa ubunto@18.191.202.184```

## Server Security and Firewall Setup
#### 3|Update and upgrade packages
- ```sudo apt-get update```
- ```sudo apt-get dist-ugrade```
- ```sudo shutdown -r now```
- SSH back into Server: ```ssh -i ~/.ssh/lightsail_key.rsa ubunto@18.191.202.184```

#### 4|Change SSH port to 2200
-Edit the ```/etc/ssh/sshd_config``` file: ```sudo nano /etc/ssh/sshd_config```.
-Change the port number on line 5 from ``22`` to ```2200```.
-Save and exit using CTRL+X and confirm with Y.
-Restart SSH: ```sudo service ssh restart```.


#### 5|Configure UFW
-Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
```sudo ufw status                  
sudo ufw default deny incoming   
sudo ufw default allow outgoing  
sudo ufw allow 2200/tcp          
sudo ufw allow www              
sudo ufw allow 123/udp      
sudo ufw deny 22
```                

-Enable firewall: ```sudo ufw enable```

-Verify: ```sudo ufw status```

```
Status: active
To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
123/udp                    ALLOW       Anywhere                  
22                         DENY        Anywhere                  
2200/tcp (v6)              ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
123/udp (v6)               ALLOW       Anywhere (v6)             
22 (v6)                    DENY        Anywhere (v6)
```

#### 6|Configure Firewall in Lightsail
Add the following setting in Lightsail firewall con
- HTTP - TCP - 80
- CUSTOM - UDP - 123
- CUSTOM - TDP - 2200
 
## Grant ```grader``` Access
#### 7|Create user ```grader``` and grant ```sudo``` access
- Create ```grader``` user: ```sudo adduser grader```
- Give ```grader``` superuser permissions: ```sudo visudo```
- Add the following line below ```root``` user: ```grader  ALL=(ALL:ALL) ALL```
#### 8|Create Private Key for ```grader```
- In Amazon Lightsail, create a new key for ```grader``` copy to local machine to provide include in submission.

## Prepare Server to Deploy App
#### 9|Configure Timezone:
- Login as ```grader``` configure timezone:
```
ubuntu@ip-172-26-14-216:~$ su - grader
Password:
grader@ip-172-26-14-216:~$ sudo dpkg-reconfigure tzdata
[sudo] password for grader:

Current default time zone: 'America/Chicago'
Local time is now:      Wed May 15 14:13:05 CDT 2019.
Universal Time is now:  Wed May 15 19:13:05 UTC 2019.

```

#### 10| Install Apache 2
```
grader@ip-172-26-14-216:~$ sudo apt-get install apache2
```

#### 11| Install MOD_WSGI for Python 3
```
grader@ip-172-26-14-216:~$ sudo apt-get install libapache2-mod-wsgi-py3
```
#### 12| Install PostgreSQL
- Install PSQL
 ```
 grader@ip-172-26-14-216:~$ sudo apt-get install postgresql
 ```
 also:
 ```
sudo apt-get install postgresql-server-dev-all
sudo apt-get install postgresql-common
 
 ```
- Create PostgreSQ: Terminal : ``sudo -u postgres psql``
- Create ```catalog``` user in Postgre w/  ```CREATEDB``` privileges.
```
postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
CREATE ROLE
postgres=# ALTER ROLE catalog CREATEDB;
ALTER ROLE
postgres=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 catalog   | Create DB                                                  | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

```
- Create Linux user ```catalog```: grader@ip-172-26-14-216:~$ sudo adduser catalog

- Grant ```catalog``` user ```sudo``` access
- Create ```catalog``` db: ```createdb catalog```
- Verify by running ```psql```, followed by ```\l```
```
catalog@ip-172-26-14-216:~$ createdb catalog
catalog@ip-172-26-14-216:~$ psql
psql (9.5.17)
Type "help" for help.

catalog=> \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 catalog   | catalog  | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(4 rows)

catalog=> \q
catalog@ip-172-26-14-216:~$ exit
logout

```
#### 13| Install Git:
```
ubuntu@ip-172-26-14-216:~$ sudo apt-get install git
Reading package lists... Done
Building dependency tree
Reading state information... Done
git is already the newest version (1:2.7.4-0ubuntu1.6).
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```

#### 14| Install Python3 Libraries:
- pscopg2: ```sudo pip3 install pscopg2```
- oauth2client: ```sudo pip3 install oauth2client```
- sqlalchemy: ```sudo pip3 install sqlalchemy```
- requests: ```sudo pip3 install requests```
- flask: ```sudo pip3 install flash```
- httplib2:  ```sudo pip3 install httplib2```

## Prepare Server to Deploy App

#### 15| Create ```itemcatalog``` directory in ```/var/www/```:
- ```cd /var/www/```
- ```mkdir itemcatalog```

#### 16| Clone GIT repository to ```itemcatalog``` directory: ```GIT CLONE https://github.com/nl492k/itemcataloglinux itemcategory```
- Verify:
```ubuntu@ip-172-26-14-216:/var/www/itemcatalog/itemcatalog$ tree
+-- client_secrets.json
+-- database_setup.py
+-- fb_client_secrets.json
+-- initial_db_load.py
+-- itemcatalog.db
+-- itemcatalog.py
+-- itemcatalog.wsgi
+-- README.md
+-- README.txt
+-- static
¦   +-- blank_user.gif
¦   +-- client_platform.js
¦   +-- styles.css
¦   +-- top-banner.jpg
+-- templates
    +-- catalog.html
    +-- deletecategory.html
    +-- deleteitem.html
    +-- editcategory.html
    +-- edititem.html
    +-- header.html
    +-- items.html
    +-- login.html
    +-- main.html
    +-- newcategory.html
    +-- newitem.html
    +-- publiccatalog.html
    +-- publicitems.html

2 directories, 26 files
```

#### 17| Create ```itemcatalog.wsgi``` in ```/var/www/itemcatalog/itemcatalog```
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/itemcatalog/itemcatalog/")

from itemcatalog import app as application
application.secret_key = '2286690089'

```

#### 17| Create Apache Config File for App:
```
sudo nano /etc/apache2/sites-available/itemcatalog.conf
```
```
<VirtualHost *:80>
   ServerName 3.15.31.49
   ServerAlias 3.15.31.49.xip.io
   ServerAdmin nl492k@att.com
   WSGIScriptAlias / /var/www/itemcatalog/itemcatalog/itemcatalog.wsgi
   <Directory /var/www/itemcatalog/itemcatalog/>
       Require all granted
   </Directory>
   Alias /static /var/www/itemcatalog/itemcatalog/static
   <Directory /var/www/itemcatalog/itemcatalog/static/>
       Require all granted
   </Directory>
   ErrorLog ${APACHE_LOG_DIR}/error.log
   LogLevel warn
   CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

#### 18| Start Virtual Host:
-```sudo a2ensite itemcatalog```

#### 19| Restart Apache Service:
```sudo service apache2 restart```

#### 20| Remove Root Login:
- Go to config file: ```nano /etc/ssh/sshd_config```
- Change ```PermitRootLogin prohibit-password``` to ```PermitRootLogin no```

#### 21| ```itemcatalog.py``` modifications:
- Update path of ```client_secrets.json``` to ```/var/www/itemcatalog/itemcatalog/client_secrets.json```
- Update path of ```fb_client_secrets.json``` to ```/var/www/itemcatalog/itemcatalog/fb_client_secrets.json```
- Update covert  ``result`` from by byte to string
- Update ```createengine``` from ```sqlite://catalog.db``` to ```postgresql://catalog:catalog@localhost/catalog```
- Update ```xrange(32)``` to ```range(32)```

#### 22| ```database_setup.py``` modifications:
- Update  ```createengine``` from ```sqlite://catalog.db``` to ```postgresql://catalog:catalog@localhost/catalog```




### References
- https://www.tecmint.com/disable-root-login-in-linux/
- https://github.com/eulerto/wal2json/issues/47
- http://initd.org/psycopg/
- https://knowledge.udacity.com/questions/5041
- https://github.com/SDey96/Udacity-Linux-Server-Configuration-Project



