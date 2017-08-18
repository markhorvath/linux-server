# Linux Server Configuration Project

## To the Reviewer:
* Public IP address: 34.227.31.137

# Steps Taken (In Order)
###Setup AWS account and Lightsail Server according to Udacity [link](https://classroom.udacity.com/nanodegrees/nd004/parts/ab002e9a-b26c-43a4-8460-dc4c4b11c379/modules/357367901175462/lessons/3573679011239847/concepts/ce268cfe-99ec-49be-9326-876375f89a22)
    *Add custom TCP 2200 port on Amazon Lightsaild Networking tab
    *Go to instance page and click "Connect using SSH" to open terminal

### Set up firewall
##### In AWS SSH Terminal
* sudo apt-get update && sudo apt-get upgrade
* sudo ufw allow 2200/tcp
* sudo ufw allow 80/tcp
* sudo ufw allow 123/udp
* sudo nano /etc/ssh/sshd_config, add ‘Port 2200’ below ‘Port 22’ for now
* sudo ufw default allow outgoing
* sudo ufw default deny incoming
* sudo ufw allow ssh
* sudo ufw enable

### Set up user 'grader' and give grader sudo access
* sudo apt-get install finger
* sudo adduser grader
* sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
* sudo nano /etc/sudoers.d/
* added grader ALL=(ALL) NOPASSWD:ALL to /etc/sudoers.d/
* chmod 700 .ssh
* Back on the main Lightsail Instance page click on the 'Account page' link toward the bottom
* Click on the "SSH Keys" tab and download the default private key

##### In a separate, local terminal shell
* ls -ltr /Users/username/Downloads/LightsailDefaultPrivateKey-us-east-1.pem (this was to check the file permissions)
* cp /Users/username/Downloads/LightsailDefaultPrivateKey-us-east-1.pem ~/.ssh/ (this is just copying the default key and moving it to /.ssh)
* chmod 600 ~/.ssh/LightsailDefaultPrivateKey-us-east-1.pem (change the permissions)
* ssh -i ~/.ssh/LightsailDefaultPrivateKey-us-east-1.pem ubuntu@34.227.31.137 -p 22 (check to make sure I can ssh into the server as ubuntu)
* sudo login grader (switch to user grader)

### Set up new keypair
##### Open a new terminal on local machine
* ssh-keygen (this creates a new keypair, place it in ~/.ssh/ and name it)
* ls -ltr ~/.ssh/ (just to make sure it's there)
* ssh -i ~/.ssh/LightsailDefaultPrivateKey-us-east-1.pem ubuntu@34.227.31.137 -p 2200 (test ssh into -p 2200)
* In the ssh terminal, enter 'sudo service ssh restart'
* Back in 'grader', enter mkdir .ssh
* touch .ssh/authorized_keys (this makes the file)
* in local terminal, enter 'cat ~/.ssh/lightsail-grader.pub' to read the file then copy contents
* Back in grader, enter 'nano .ssh/authorized_keys' to open file, then paste contents of public key into file, hit ctrl-x, Y, enter
* chmod 700 .ssh
* chmod 644 .ssh/authorized_keys
* test ssh as 'grader': ssh –i ~/.ssh/lightsail-grader grader@34.227.31.137 -p 2200
* did the same steps for user 'ubuntu' as a precaution, not sure if necessary
* sudo nano /etc/ssh/sshd_config to delete Port 22 from ports to listen to
* On Amazon Lightsail Networking page, under "Firewall" section click "Edit rules" and delete port 22

### Configure the Local Timezone to UTC
* 'date' was already showing UTC but entered the following command anyway according to this [link](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)
* sudo dpkg-reconfigure tzdata
* Enter, Enter, (aka select 'none of the above' and 'UTC')

### Install and configure Apache to serve a Python mod_wsgi application.
* sudo apt-get install apache2
* sudo apt-get install libapache2-mod-wsgi
* sudo a2enmod wsgi
* sudo nano /etc/apache2/sites-enabled/000-default.conf (per Udacity instructions)
* Right before closing tag of <VirtualHost *:80> added the following line: WSGIScriptAlias / /var/www/html/myapp.wsgi
* ctrl+X, Y, Enter, back on command line enter 'sudo apache2ctl restart'
* sudo nano /var/www/html/myapp.wsgi
* Placed the code found here [link](https://classroom.udacity.com/nanodegrees/nd004/parts/ab002e9a-b26c-43a4-8460-dc4c4b11c379/modules/357367901175461/lessons/4340119836/concepts/48018692630923) for testing
* sudo apt-get install python-pip
* sudo pip install virtualenv
* cd /var/www
* sudo mkdir Catalog
* cd into Catalog, the again ‘sudo mkdir MyApp’
* cd into new Catalog, then ‘sudo mkdir static templates’
* sudo nano __init__.py
* enter the code found [here on Digital Ocean]{https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) then ctrl-X, Y, Enter
* sudo virtualenv venv
* . venv/bin/activate (the '.' is the same as 'source', but source wasn't available in my shell)
* Inside venv, run 'sudo pip install Flask'
* Tried running 'sudo python __init__.py' but "No such file or direc


### Install and configure PostgreSQL
* sudo apt-get install postgresql
* sudo -i -u postgres
* sudo -u postgres psql
* sudo -u postgres createuser -P catalog
* Enter password twice
* sudo -u postgres createdb -O catalog catalog (the first catalog is the user, 2nd is the db name)

### Install Git
* sudo apt-get install git

### Deploy the Item Catalog Project
* cd var/www/
* sudo apt-get install python-sqlalchemy
* sudo git clone https://github.com/markhorvath/songcatalog.git
* cd songcatalog, cd vagrant, cd catalog
* sudo nano songsdb_setup.py
* go to bottom, change create_engine to 'postgresql://catalog:catalog@localhost/catalog', ctrl-X, Y, Enter
* sudo -H pip install psycopg2
* sudo python songsdb_setup.py
* sudo nano songpopulator.py
* change create_engine to 'postgresql://catalog:catalog@localhost/catalog', ctrl-X, Y, Enter
* sudo python songpopulator.py (should notify when finished)
* sudo -H pip install Flask
* in songcatalog, sudo nano .htaccess, add "RedirectMatch 404 /\.git", save & exit
* sudo nano __init__.py
* Add the following:
```
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello"
if __name__ == "__main__":
    app.run()
```
* sudo nano /etc/apache2/sites-available/songcatalog.conf
```
<VirtualHost *:80>
                ServerName 34.227.31.137
                ServerAdmin admin@34.227.31.137
                WSGIScriptAlias / /var/www/Catalog/songcatalog.wsgi
                <Directory /var/www/Catalog/Catalog/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/Catalog/Catalog/static
                <Directory /var/www/Catalog/Catalog/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
* sudo a2ensite songcatalog returned this:
    *Enabling site songcatalog.
    *To activate the new configuration, you need to run:
    *service apache2 reload
* attempted running service apache2 reload, but it asked for a password i was unaware of
##### Create the .wsgi file
* cd /var/www/Catalog
* sudo nano songcatalog.wsgi and add this code:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/Catalog/")

from Catalog import app as application
application.secret_key = 'super_secret_key'
```
* sudo service apache2 restart



