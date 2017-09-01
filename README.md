# Linux Server Configuration Project

## To the Reviewer:
* Public IP address: 54.83.148.219
* SSH Port: 2200
* URL: http://ec2-54-83-148-219.compute-1.amazonaws.com/

# Steps Taken (In Order)
### Start a new Ubuntu Linux server instance on Amazon Lightsail. [link](https://classroom.udacity.com/nanodegrees/nd004/parts/ab002e9a-b26c-43a4-8460-dc4c4b11c379/modules/357367901175462/lessons/3573679011239847/concepts/ce268cfe-99ec-49be-9326-876375f89a22)
*Add custom TCP 2200 port on Amazon Lightsaild Networking tab
*Go to instance page and click "Connect using SSH" to open terminal

### Connect SSH

### Update all currently installed packages
* `sudo apt-get install finger`
* `sudo apt-get update` then `sudo apt-get upgrade`


### Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.
* In Networking tab, add Custom TCP port 2200
* `sudo nano /etc/ssh/sshd_config` add ‘Port 2200’ below ‘Port 22’ for now
* download default ssh key, then `cp Users/username/Downloads/lightsailkey.pem ~/.ssh/`
* rename it as you see fit, eg grader-key
* `chmod 600 grader-key`
* `ssh -i ~/.ssh/grader-key ubuntu@54.83.148.219`

### Configure the Uncomplicated Firewall (UFW) for SSH (port 2200), HTTP (port 80), and NTP (port 123).
* `sudo ufw allow 2200/tcp && sudo ufw allow 80/tcp && sudo ufw allow 123/udp`
* `sudo ufw default allow outgoing`
* `sudo ufw default deny incoming`
* `sudo ufw allow ssh`
* `sudo ufw enable`

### Create a new user account named grader.
* `sudo adduser grader` pw 'grader'

### Give grader the permission to sudo.
* Most of this part was based on this Digital Ocean [article](https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file-on-ubuntu-and-centos)
* `sudo visudo`
* In #User privelege specification section add the line `grader ALL=(ALL:ALL) ALL` under the root line
* `sudo visudo -f /etc/sudoers.d/grader` then add `grader ALL=(ALL:ALL) ALL`

### Create an SSH key pair for grader using the ssh-keygen tool.
* In separate terminal, navigate to /.ssh/ and enter `ssh-keygen`
* Name the file, udacity-grader in this case
* Back in ssh connection, enter `sudo login grader` to switch to user grader
* `mkdir .ssh` then add file `touch /.ssh/authorized_keys`
* copy contents of .ssh/udacity-grader.pub on local machine to /.ssh/authorized_keys, save and exit
* set permissions via `chmod 700 .ssh && chmod 644 .ssh/authorized_keys`
* Can now log in as grader `ssh -i ~/.ssh/udacity-grader grader@54.83.148.219 -p 2200`
* Disable ssh into Port 22 with `sudo nano /etc/ssh/sshd_config' then delete 'Port 22'
* While here also change root 'PermitRootLogin prohibit-password' to 'PermitRootLogin no'
* `sudo service ssh reload`

### Configure the local timezone to UTC
* 'date' was already showing UTC but entered the following command anyway according to this [link](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)
* `sudo dpkg-reconfigure tzdata`
* Enter, Enter, (aka select 'none of the above' and 'UTC')

### Install and configure Apache to serve a Python mod_wsgi application
* Some of this was from this Digital Ocean [article](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps#setup)
* `sudo apt-get install apache2`
* `sudo apt-get install libapache2-mod-wsgi python-dev`
* `sudo a2enmod` to enable the mod_wsgi
* Test to see it works `sudo service apache2 restart` then open a browser and go to your IP address
* `cd /var/www/` then `sudo mkdir catalog`, cd into catalog then `sudo mkdir catalog` again
* in /var/www/catalog, made a catalog.wsgi file with `sudo nano catalog.wsgi` with the following code:
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
```
* Ctrl+X, Y, Enter

### Install and configure PostgreSQL
##### Do Not Allow Remote Connections
* Most of this is from this DigitalOcean [article](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps) and postgres documentation
* `sudo apt-get install postgresql postgresql-contrib`
* `sudo su - postgres`
* `psql`
* `CREATE USER catalog WITH PASSWORD 'catalog';` [documentation](https://www.postgresql.org/docs/8.0/static/sql-createuser.html)
* `ALTER USER catalog CREATEDB;` [documentation](https://www.postgresql.org/docs/9.5/static/sql-alteruser.html)
* verify with `\du`
* `CREATE DATABASE catalog WITH OWNER catalog;`
* `/c catalog` to connect to catalog db
* `REVOKE ALL ON SCHEMA public FROM public;` [documentation](https://www.postgresql.org/docs/9.0/static/sql-revoke.html)
* `GRANT ALL ON SCHEMA public TO catalog;`
* `\q` to exit catalog db
* exit psql
* In all application files that reference `sqlite:///songs.db` from the previous project, replace with the following: `postgresql://catalog:catalog@localhost/catalog`
* While the virtual environment is running, run songsdb_setup.py and then songpopulator.py.  Go into sequel and connect to catalog then `\dt` to confirm the tables were created.  Try some select statements

##### Create a new database named catalog that has limited permissions to your catalog application database.

### Install git.
* `sudo apt-get install git`

### Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.
* `git clone https://github.com/markhorvath/songcatalog.git`
* Have to tweak the directory structure by moving files up to /var/www/catalog/catalog/ with `mv * .[^.]* ..` and removing the vagrant directory and Vagrantfile with `rmdir` or `rm`

### Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your .git directory is not publicly accessible via a browser!
##### Install depencies and venv (much of this from Digital Ocean [Step 3](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps#setup))
* `sudo apt-get install python-pip`
* `sudo pip install virtualenv`
* In /var/www/catalog, `sudo virtualenv venv`
* `source venv/bin/activate`
* Change permissions `sudo chmod -R 777 venv`
* 'sudo pip install Flask'
* `pip install sqlalchemy oauth2client httplib2 requests psycopg2`
* `sudo nano /etc/apache2/sites-available/catalog.conf` and paste in the following code:
```
<VirtualHost *:80>
    ServerName example.com
    ServerAlias ec2-54-83-148-219.compute-1.amazonaws.com/
    ServerAdmin admin@54.83.148.219
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
</VirtualHost>
```
`sudo service apache2 reload`
* Check sites enabled: `cd /etc/apache2/sites-available`
* Check what sites are enabled: `ls -alh /etc/apache2/sites-available`
* `sudo a2dissite 000-default.conf` to disable default
* `sudo serve apache2 reload`
### TROUBLESHOOTING TIME
* `cd /var/log/` and then `sudo tail apache2/error.log` to view errors
* No module named sqlalchemy
* with venv active, `pip install Flask-SQLAlchemy`
* ran `python` then `import sqlalchemy`, no errors
* both `pip freeze` and `ls venv/lib/python2.7/site-packages/` showed sqlalchemy
* added `WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages` and `WSGIScriptReloading On` to catalog.conf based on this [article](http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/#working-with-virtual-environments)
* added `activate_this = '/var/www/catalog/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))` to top of .wsgi file
* 'No module named sqlalchemy' is now gone, but now it's for requests instead, SOLVED: did pip install 'request' instead of 'requests' initially
* Getting error: `column "category.id" must appear in the GROUP BY clause or be used in an aggregate function`
* Run `sudo tail -50 /var/log/apache2/error.log` to view up to 50 error logs back in /var/log/
* Changed line 221 `categories = session.query(Category).group_by(Category.name).all()` in __init__.py to `categories = session.query(Category).all()`
* Site is now live at IP address
* ssh as grader, `sudo ufw deny 22` (forgot to deny this first time around)
* ran `sudo apt-get update && sudo apt-get upgrade`
* `/usr/lib/update-notifier/apt-check --human-readable` to ensure packages are up to date [link](https://askubuntu.com/questions/49958/how-to-find-the-number-of-packages-needing-update-from-the-command-line)

### Set up Google OAuth
* Go to google developers Song Catalog Project
* Click 'Edit' button for 'OAuth 2.0 client IDs'
* Added 'http://ec2-54-83-148-219.compute-1.amazonaws.com/' along with '/login, /gconnect, /categories' appended to redirect URIs and the IP address and alias to Authrozied Javascript origins
* `sudo tail -50 /var/log/apache2/error.log`
* Error with client_secrets.json in line 78, `sudo nano __init__.py` and add full path at line 78: `/var/www/catalog/catalog/client_secrets.json`
* `sudo service apache2 reload`
* IT WORKS!
