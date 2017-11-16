# Linux-Server Configuration
Udacity linux server project

# Server details
IP address: `http://18.195.133.108/`

SSH port: `2200`

You can visit http://18.195.133.108/ for the website deployed.

# Configurations

## Update all installed packages

`apt-get update` - to update the package indexes

`apt-get upgrade` - to actually upgrade the installed packages

## Add user
Add user `grader` with command: `sudo useradd grader`

## Create SSH keys from user 'grader'
```
sudo mkdir /home/grader/.ssh
chown grader:grader /home/grader/.ssh
chmod 700 /home/grader/.ssh
touch /home/grader/.ssh/authorized_keys
chown grader:grader /home/grader/.ssh/authorized_keys
chmod 644 /home/grader/.ssh/authorized_keys
```
To login as grader, we need to copy the public key (created with ssh-keygen command) into /home/grader/.ssh/authorized_keys on the AWS machine.
Now, go to the directory saving the private key file and open a terminal. One should be able to login the server by giving this command:
```
ssh -i private_key grader@18.195.133.108
```
## Disable root login and set UTC TimeZone
Change the following line in the file `/etc/ssh/sshd_config`:
From `PermitRootLogin without-password` to `PermitRootLogin no`.
To change the timezone, one only need to:
`sudo timedatectl set-timezone UTC`

## Change SSH port from 22 to 2200
Edit the file `/etc/ssh/sshd_config` and change the line `Port 22` to:

`Port 2200`

Since the port is changed, we need add this role to the server by editing the 'Network' menu on Amazon lightsail website. Now one might need to use the following command to login to the server:

`ssh -i private_key.pem grader@18.195.133.108 -p 2200`

Then restart the SSH service:

`sudo service ssh restart`

## Configuration Uncomplicated Firewall (UFW)
Using the following commands to finish the settings for UFW

`sudo ufw default deny incoming`
`sudo ufw default allow outgoing`
`sudo ufw allow 2200/tcp`
`sudo ufw allow www`
`sudo ufw allow ntp`
`sudo ufw enable`

To check ufw's rules, use `sudo ufw status`. one will get the following results:

```
To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123                        ALLOW       Anywhere
5000                       DENY        Anywhere
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123 (v6)                   ALLOW       Anywhere (v6)
5000 (v6)                  DENY        Anywhere (v6)

```
Again, add these rules to lightsail server too on amazon aws website.

## Install Apache to serve a Python mod_wsgi application

`sudo apt-get install apache2`

`sudo apt-get install libapache2-mod-wsgi`

## Install and configure PostgreSQL

`sudo apt-get install postgresql postgresql-contrib`

Because remote connection to PostgreSQL is not allowed, the configuration file `/etc/postgresql/9.5/main/pg_hba.conf` should only
allow connections from the local host addresses `127.0.0.1` for IPv4.

Go to psql environment typing:`psql -U postgres` to create new user.

```
postgres=# CREATE DATABASE catalog;
postgres=# ALTER ROLE postgres WITH PASSWORD 'password';
```
Quit postgreSQL by  `postgres=# \q`

## Install Git and Clone Catalog project file
Install GIT
`sudo apt-get install git`
and clone repositiry to ubuntu home:
```
git clone https://github.com/Lonitch/catelog.git
```
Use `cd /var/www` to move to the '/var/www' directory
Create the directory `sudo mkdir myApp`
Move inside this directory and create another myApp directory again and copy the files inside "catalog" directory of the cloned project.

Rename project.py to __init__.py by `sudo mv project.py __init__.py`
In order to let the app connect the database, I need to edit database_setup.py, project.py. 
In these files, we have some commands like:

`engine = create_engine('sqlite:///categoryApp.db')` 

Change it into

`engine = create_engine('postgresql://postgres:postgres@localhost/catalog')`

To load initial data, one only need to issue the following commands

```
sudo python database_setup.py
```

## Install Flask, SQLAlchemy and other Supporting Package
Use the following commands:
```
sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy python-pip
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2
sudo pip install flask-seasurf
```

## Configure Virtual Host
Create myApp.conf by: `sudo nano /etc/apache2/sites-available/myApp.conf`
And add following lines in the file

```
<VirtualHost *:80>
        ServerName 18.195.133.108
        #ServerAdmin mrganesh86@gmail.com
        WSGIScriptAlias / /var/www/myApp/myapp.wsgi
        <Directory /var/www/myApp/myApp/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/myApp/myApp/static
        <Directory /var/www/myApp/myApp/static/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable the host by: `sudo a2ensite myApp`

## Create the .wsgi File
Go back to /var/www/myApp and create a myapp.wsgi file:

```
cd /var/www/myApp
sudo nano myapp.wsgi 
```

Add the following lines

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/myApp/")

from myApp import app as application
application.secret_key = 'super_secret_key'
```

Restart Apache `sudo service apache2 restart`

## Update the Google OAuth client secrets file
Fill the `client_id` and `client_secret` fields in the file `client_secrets.json`.
Change the `javascript_origins` field to the IP address`18.195.133.108`.

These addresses also need to be entered into the Google Developers Console.

Besides, in the '__init__.py', there are some commands like:

```
open('client_secrets.json', 'r').read())['web']['client_id']
```

we need to add an absolute path to `client_secretes.jon` by changing the code line into:

```
open('/var/www/myApp/myApp/client_secrets.json', 'r').read())['web']['client_id']
```

Now, the configuration is finished. we can access the catalog app by opening a browser and directing it to `http://18.195.133.108/`