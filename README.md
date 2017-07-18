# Linux Server Configuration Project - R Duran - Udacity FSND

Note: The server that was created for this project is now offline.  I have changed references to the IP in this document to X.X.X.X as the static IP is no longer attached to the server instance.  If you are using this readme to set up your own instance, simply substitute in your IP address.

## OS Setup

Created an account on Amazon Lightsail - https://amazonlightsail.com/

Selected OS only / Ubuntu

Logged into the OS using the browser based ssh terminal in the Lightsail console

Created a static IP address and attached it to the OS instance

## Server setup and configuration

Updated the software source listing

    sudo apt-get update

Upgrade software 

    sudo apt-get upgrade
    sudo apt-get dist-upgrade

Clean up packages not needed

    sudo apt-get autoremove

Rebooted the server using the Lightsail console

Downloaded the .pem file and used it to log into the server using a local terminal (Replace 'pem_filename.pem' with the filename of your actual private key file.

    ssh ubuntu@X.X.X.X -i 'pem_filename.pem' 


Created an account for the Udacity grader

    sudo adduser grader

Set full name on 'grader' account to 'Udacity Grader'
Left all the other default user info (room, phone, etc.) blank

Created a 'grader' file in /etc/sudoers.d/
The 'grader' sudoers file has one rule

    grader ALL=(ALL) NOPASSWD: ALL

On the local terminal, created a public/private key pair for 'grader' and saved it to provide to the Udacity grader (pub/priv key not included in this readme)

    ssh-keygen

Tested that grader could access the system using the key (replace grader_priv_key with the actual key filename)

    ssh grader@X.X.X.X -i 'grader_priv_key'

Logged out of 'grader' account and logged back in as ubuntu user

Forced key based authentication (This was already configured by default in ASW Lightsail).  In /etc/ssh/sshd_config file made sure that PasswordAuthentication was set to 'no'

Restart SSH service

    sudo service ssh restart

Added custom tcp port 2200 in the Lightsail console

Configured UFW

    sudo ufw status
    (was inactive)
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    sudo ufw deny 22
    sudo ufw allow 2200/tcp
    sudo ufw allow ntp
    sudo ufw allow www

Enabled UFW and check status

    sudo ufw enable
    sudo ufw status

Modified /etc/ssh/sshd_config

    Changed 'Port 22' to 'Port 2200'

Tested ssh connection to port 2200

    ssh grader@X.X.X.X -i 'grader_priv_key' -p 2200

Removed port 22 from the Lightsail firewall

Restarted SSH service

    sudo service ssh restart

Configured time zone to UTC 

    sudo dpkg-reconfigure tzdata

Selected 'None of the Above' from first screen

    Selected UTC from second screen (UTC was the default setting)

Installed and set up NTP

    sudo apt-get install ntp
    sudo vim /etc/ntp.conf
    sudo service ntp restart

Checked NPT logging

    sudo tail -f /var/log/syslog

Installed PostgreSQL

    sudo apt-get install postgresql

Ensured only local log in to PostgreSQL allowed.  Verified /etc/postgresql/9.5/main/pg_hba.conf

Set the 'postgres' user password

    sudo passwd postgres

Switch to postgres user and start psql

    su - postgres
    psql
    
Changed PostgreSQL admin password

    (while logged into psql as the postgres user)
    \password postgres
    (entered and confirmed password)


Created a PostgreSQL 'catalog' user (while logged into psql as postgres user)

    CREATE USER catalog;

Gave 'catalog' user a password (while logged into psql as postgres user)
    
    ALTER ROLE catalog PASSWORD 'put_password_here';

Created the catalog database and give 'catalog' user access (while logged into psql as postgres user)
    
    CREATE DATABASE catalog;
    GRANT ALL PRIVILEGES ON DATABASE catalog to catalog;

Exited psql

    \q

Created directory for the catalog app

    cd /var/www
    sudo mkdir Catalog
    cd Catalog

Before uploading the catalog project to GitHub, I created a clean version of the project without any credentials or logs.  This is to prevent roll back through the logs that might reveal credentials that were not removed from initial commits.

Created a GitHub repo and pushed the clean catalog project

Cloned my GH catalog repo into catalog directory on the server

    git clone https://github.com/rdUdacity-FSND/catalog.git

Renamed my project directory to Catalog

    sudo mv catalog Catalog

Installed required packages for the catalog

    cd Catalog
    sudo pip install -r requirements.txt
    sudo pip install psycopg2

Converted the catalog app database from SQLite to PostgreSQL by modifying models.py, init_database_data.py, and application.py files (cat_pass is the catalog user password that was set earlier)

        Changed from:
            engine = create_engine('sqlite:///cat_app.data.db')
        To:
            engine = create_engine('postgresql://catalog:cat_pass@localhost/catalog')
        

Set up the database models and initialized the database

    sudo python models.py
    sudo python init_database_data.py

Manual check of the database (while logged into psql as postgres user) to ensure database is populated

    \c catalog
    SELECT * FROM catagories;
    (data was present)
    SELECT * FROM items;
    (data was present)

Configured Apache conf file

    cd /etc/apache2/sites-available/
    sudo vim Catalog.conf
    (edited the conf file)

Enabled catalog site

    sudo a2ensite Catalog.conf

Created wsgi file

    cd /var/www/Catalog
    sudo vim Catalog.wsgi
    (edited the wsgi file)

Renamed the main application py file

    cd /var/www/Catalog/Catalog
    sudo mv application.py __init__.py

## Issues and Solutions

Initially I could get a simple ‘Hello World’ type .wsgi file to run but couldn’t get my application to start up.  I erroneously put the Catalog.conf file in the sites-enabled directory instead of the sites-available directory.  Found the cause and solution here:

https://serverfault.com/questions/83508/purpose-of-debian-sites-available-and-sites-enabled-dirs/825297#825297


In my main application code, I had to change the relative path to the Google OAuth credential file to an absolute path.

When creating the database (running models.py), after changing over to use PostgreSQL, I had to increase the size of the password hash.  This was not an issue when using the SQLite database.  I used these resources while setting up PostgreSQL:

https://discussions.udacity.com/t/tips-for-configuring-postgresql-and-setting-up-item-catalog-project/223436

https://stackoverflow.com/questions/21354136/dataerror-value-too-long-for-type-character-varying16-in-django-social-auth


While logging in using Google (Using Google OAuth), there was an origin mismatch error.  I discovered that I needed to set the origin in the Google API console to the URL and not to the IP address.  This resource was helpful in pointing out the issue:

https://stackoverflow.com/questions/16850350/got-origin-mismatch-error-in-google-share-api

		

## Other useful resources/references:  

https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
https://www.postgresql.org/docs/9.6/static/index.html
http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/#configuring-apache

#
R Duran - Udacity Linux Server Configuration Project

