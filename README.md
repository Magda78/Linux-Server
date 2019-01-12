# Linux-Server

# About the project
Baseline installation of a Linux server and prepare it to host web applications. Secure your server from a number of attack vectors, installation and configuration database server.
# Server settings after configuration
    IP address: 54.91.68.88
    URL to hosted web application: www.projectcatalog.de
    ssh port:2200
    
## 1. Creating an instance on AWS Lightsail

Configuration steps can be found on Udacity course [here](http://https://classroom.udacity.com/nanodegrees/nd004/parts/ab002e9a-b26c-43a4-8460-dc4c4b11c379/modules/357367901175462/lessons/3573679011239847/concepts/c4cbd3f2-9adb-45d4-8eaf-b5fc89cc606e)

## 2. Configuration performed on the Ubuntu Server.

- Update and upgrade packages

    `$ sudo apt-get update && apt-get upgrade`

- Install finger application

    `$ sudo apt-get install finger`

- Create new user

    `$ sudo adduser grader`

- Check if user was created

    `$ finger grader`

- Add user to the sudoers file

    `$ sudo visudo `

    under “User privilege specification” add

    `$ grader ALL=(ALL:ALL) ALL`

## 3. Configuration performed on the local machine

- Generate key pair

    `$ ssh-keygen`

- Copy content of the file project.pub using command

    `$ sudo cat .ssh/project.pub`

## 4. Configuration on the client side.

- Connecting to the new user and move to the home directory

    `$ sudo su grader `

- Install a public key

    `$ mkdir .ssh`

    `$ touch .ssh/authorized_keys`

- Edit authorized_keys file

    `$ sudo nano .ssh/authorized_keys `
    
    by pasting content of the .ssh/project.pub file from local machine.

- Setup file permission on ssh directory and authorized_keys file

    `$ chmod 700 .ssh `

    `$ chmod 644 .ssh/authorized_keys `

- Disable password base login by changing Password authentication to NO

    `$ sudo nano /etc/ssh/sshd_config `

- Restart service

    `$ sudo service ssh restart`

- Change ssh port from 22 to 2200

    `$ sudo nano /etc/ssh/sshd_config`

    Also change port in AWS Lightsail by adding custom application with protocol TCP and port 2200.

- Disabled ssh root login by changing PermitRootLogin to NO

    `$ sudo nano /etc/ssh/sshd_config`

- Configure firewall

    `$ sudo ufw status `

    `$ sudo ufw default deny incoming`

    `$ sudo ufw default allow outgoing`

    `$ sudo ufw allow 2200/tcp`

    `$ sudo ufw allow 80/tcp`

    `$ sudo ufw allow 123/udp`

    `$ sudo ufw enable`

    `$ sudo ufw status `

- Check timezone if is set to UFC

    `$ date +%Z `

## 5. Apache2 installation and configuration

- Install Apache2

    `$ sudo apt-get install apache2 `

- Install wsgi

    `$ sudo apt-get install libapache2-mod-wsgi`

- Enable wsgi

    `$ sudo a2enmod wsgi`

- Install python

    `$ sudo apt-get install libapache2-mod-wsgi python-dev `

- Restart Apache2

    `$ sudo apache2ctl restart `

- Create directory in the location /var/www/ to store web application

    `$ mkdir Catalog`

- Move to the directory create above

    `$ cd Catalog`

- Clone git repository

    `$ sudo git clone https://github.com/Magda78/catalog_app.git`

- Edit file Catalog.wsgi and save in the project directory

        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0, '/var/www/Catalog/catalog_app')

        from finalproject import app as application
        application.secret_key='*************'

- Edit file Catalog.conf

    `$ sudo nano /etc/apache2/sites-available/Catalog.conf`

        <VirtualHost *:80>
            ServerName  54.91.68.88
            #Location of the items-catalog WSGI file
            WSGIScriptAlias / /var/www/Catalog/catalog_app/Catalog.wsgi
            #Allow Apache to serve the WSGI app from our catalog directory
            <Directory /var/www/Catalog/catalog_app>
                Order allow,deny
                Allow from all
            </Directory>
            #Allow Apache to deploy static content
            <Directory /var/www/Catalog/catalog_app/static>
                Order allow,deny
                Allow from all
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>

- Enable and restart

    `$ sudo a2dissite 000-default.conf`

    `$ sudo a2ensite Catalog.conf `

    `$ sudo service apache2 reload `

##  6. Install the other software

    `$ sudo apt-get install python-pip python-flask python-sqlalchemy python- psycopg2`

    `$sudo pip install oauth2client requests httplib2`

##  7. Install Postgresql

- Install by typing

    `$sudo apt-get install postgresql`

- Move to postgres and type psqi

    `$sudo su - postgres`

- Type commands

    `$postgres=# CREATE USER catalog WITH PASSWORD 'catalog';`

    `$postgres=# ALTER USER catalog CREATEDB;`

    `$postgres=# CREATE DATABASE catalog WITH OWNER catalog;`

- Connect to database

    `$\c catalog`

- On database type commands

    `catalog=# REVOKE ALL ON SCHEMA public FROM public;`

    `catalog=# GRANT ALL ON SCHEMA public TO catalog;`

- Exit

    `postgres=# \q`

    `postgres@PublicIP~$ exit`

##  8. Change Oauth configuration in finalproject.py.

    CLIENT_ID = json.loads( open('/var/www/Catalog/catalog_app/ client_secrets.json', 'r').read())['web']['client_id']

    oauth_flow = flow_from_clientsecrets('/var/www/Catalog/catalog_app/client_secrets.json', scope='')

## 9. Change engine configuration in database_setup.py and finalproject.py

    engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
    
##  10. Restart Apache

    `sudo service apache2 restart`

##  11. Set up Route 53 domain
Help with domain registration can be found in [AWS documentation](http://https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/getting-started.html)


