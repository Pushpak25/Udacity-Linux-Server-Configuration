# Udacity-Linux-Server-Configuration
The last project in the Udacity Full stack nanodegree is to deploy Item catalog project on the server using AWS LightSail instance. You can find the hosted link here: http://ec2-35-154-72-191.ap-south-1.compute.amazonaws.com/
<br>

## Details of Amazon LightSail Instance
<br>
<ul><li>IP address: 35.154.72.191</li>
<li>Accessible SSH port: 2200</li>
<li>Application URL: http://ec2-35-154-72-191.ap-south-1.compute.amazonaws.com/</li></ul>
<br>

## Instructions to SSH into server.
<br>
<ol><li>Download default Private Key from the SSH keys section in Amazon Lightsail (in your Account section).</li>
<li>Move the private key file into the folder ~/.ssh . You can use the below command to do this if your file is in Downloads folder
  
  ```mv ~/Downloads/LightsailDefaultPrivateKey-ap-south-1.pem ~/.ssh/```
</li>
<li>Set permissions for your key typing the following in the terminal 
  
  ```chmod 400 ~/.ssh/LightsailDefaultPrivateKey-ap-south-1.pem```
</li>
<li>After that, type in 
  
  ```ssh -i ~/.ssh/LightsailDefaultPrivateKey-ap-south-1 ubuntu@35.154.72.191```</li>
<li>After this step you should be able to successfully login as ubuntu user</li>
</ol><br> 

## Create a new user named grader
<br>

```sudo adduser grader``` (You'll be asked to give a password)

```sudo touch /etc/sudoers.d/grader```

```sudo nano /etc/sudoers.d/grader```, type in grader ALL=(ALL:ALL) NOPASSWD:ALL, save and quit (Ctrl+X , Y)
<br>

## SSH login using ssh-keygenn
<br>
Type 

```ssh-keygen```

in a command window in your local machine. Save that private key in ~/.ssh folder.<br>

On your virtual machine:
<br>

```su - grader```<br>
```mkdir .ssh```<br>
```touch .ssh/authorized_keys```<br>
```nano .ssh/authorized_keys```<br>

Copy the contents of public key (id_rsa.pub) generated on your local machine to this file and save
<br>
```chmod 700 .ssh```<br>

```chmod 644 .ssh/authorized_keys```<br>

```service ssh restart```<br>
Now verify by logging into system with newly created user<br>
```ssh -i ~/.ssh/id_rsa grader@35.154.72.191``` <br>

# Secure your server.
<br>

## Update all currently installed packages

```sudo apt-get update``` <br>

```sudo apt-get upgrade``` <br>

## Change the SSH port from 22 to 2200
<br>
Warning: When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you don't lock yourself out of the server. When you change the SSH port, the Lightsail instance will no longer be accessible through the web app 'Connect using SSH' button. The button assumes the default port is being used. There are instructions on the same page for connecting from your terminal to the instance. Connect using those instructions and then follow the rest of the steps.
First, open Port 2200 for TCP under network tab of Amazon LightSail account
<br>

```sudo nano /etc/ssh/sshd_config```

<ul><li>change port from 22 to 2200</li>
<li>Update PermitRootLogin to <em>no</em> (disable root login)</li>
<li>change PasswordAuthentication from no to yes</li>
<li>add AllowUsers grader at end of the file </li>
<li>sudo service ssh restart</li>
 </ul>
<br>
Now verify your changes by 

```ssh -i ~/.ssh/id_rsa -p 2200 grader@35.154.72.191```
<br>

## Configure the Uncomplicated Firewall (UFW) 
<br>```sudo ufw status```
```sudo ufw default deny incoming```<br>
```sudo ufw default allow outgoing```<br>
```sudo ufw allow 2200/tcp```<br>
```sudo ufw allow 80/tcp```<br>
```sudo ufw allow 123/udp```<br>
```sudo ufw show added```<br>
```sudo ufw enable```<br>
```sudo ufw status```

<br>

# Prepare to deploy your project.

<br>

## Configure the local timezone to UTC
<br>Configure timezone using 

```sudo dpkg-reconfigure tzdata``` ( select none of the above and then set timezone to UTC). Infact, it's already configured to UTC <br>

## Install and configure Apache to serve a Python mod_wsgi application
<br>Install apache using

```sudo apt-get install apache2``` 
<br>Type 35.154.72.191 (public IP address) on URL. If you see ubuntu apache default page, your installation of apache2 is success
<br>Install mod_wsgi using sudo apt-get install libapache2-mod-wsgi .
Edit /etc/apache2/sites-enabled/000-default.conf file.<br>
add the following line at the end of the <VirtualHost *:80> block, right before the closing line: 

```WSGIScriptAlias / /var/www/html/myapp.wsgi```

<br>
Restart Apache with 

```sudo service apache2 restart command```<br>
To test if you have your Apache configuration correct you can write a very basic WSGI application.Create the /var/www/html/myapp.wsgi file using the command 

```sudo nano /var/www/html/myapp.wsgi``` <br> Within this file, write the following application:


    ```def application(environ, start_response):
    
          status = '200 OK'
          
          output = 'Hello World!'

          response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
          
          start_response(status, response_headers)

          return [output]```
          
          
<br>
If you can see Hello World! upon page reload of 35.154.72.191, your installation is successful. 
<br>

## Install git, clone and setup your Catalog project

<br>

```sudo apt-get install git```<br>

```cd /var/www```<br>

```sudo mkdir catalog```<br>

```cd catalog```<br>

```git clone https://github.com/Pushpak25/Udacity-Item-Catalog.git```<br>

```sudo mv ./Udacity-Item-Catalog ./catalog```<br>

```cd catalog```<br>

```mv project.py __init__.py```<br>
Edit database_setup.py and lotsofmenus.py to change ```engine = create_engine('sqlite:///catalog.db')``` to ```engine = create_engine('postgresql://catalog:password@localhost/catalog')``` as we'll be using postgresql for this project<br> (edit this line whererver you write create_engine command.

``` 
sudo apt-get install python-pip
pip install httplib2
pip install requests
sudo pip install --upgrade oauth2client
sudo pip install sqlalchemy
pip install Flask-SQLAlchemy
sudo pip install flask-seasurf 
```

<br>
To make github repository inaccessible make a .htaccess file in /var/www/catalog.
paste the content - RedirectMatch 404 /\.git in this file and save it .
<br>

## Deploy Flask Application

<br>

```
sudo apt-get install python-dev
sudo a2enmod wsgi
sudo pip install virtualenv
sudo virtualenv venv
source venv/bin/activate
```

<br>
After this, execute this command 

```sudo nano /etc/apache2/sites-available/catalog.conf``` 
and place the below content in catalog.conf file. 

```

<VirtualHost *:80>
    ServerName 35.154.72.191
    ServerAdmin admin@35.154.72.191
    ServerAlias ec2-35-154-72-191.ap-south-1.compute.amazonaws.com
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
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
```

<br>
In the next step, make a file catalog.wsgi under /var/www/catalog folder and place the below content there

```Python
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key' #This should match your secret key in __init__.py
```

After that execute the below 2 commands

```
sudo a2ensite catalog
sudo service apache2 restart
```

# Install PostgreSQL

Install postgresql and login as user postgres

```
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres
psql
```

Do the following database operations in psql
<br>

 ```
 CREATE USER catalog WITH PASSWORD 'catalog-pw';
 ALTER USER catalog CREATEDB;
 CREATE DATABASE catalog WITH OWNER catalog;
 \c catalog
 REVOKE ALL ON SCHEMA public FROM public;
 GRANT ALL ON SCHEMA public TO catalog;
 \q
 ```
 
 <br>
 
 exit as postgres user and restart postgresql using 
 
 ```sudo service postgresql restart```
 
 # Run the Application
 
 First change the create_engine line everywhere as mentioned above. Next, change client_secrets.json file path everywhere to absolute path like /var/www/catalog/catalog/client_secrets.json
```
cd /var/www/catalog/catalog
python database_setup.py
python lotsofmenus.py
sudo service apache2 restart
python __init__.py
```

Now, we should be able to access our item catalog website in our public IP (http://35.154.72.191). One last thing to be done to get our application completely working is to modify our client_secrets json file. Add host name and public IP address to redirect uri's and javascript origins. 

# Attributions

[Udacity Course Videos](https://classroom.udacity.com/courses/ud299-nd) <br>
[Discussion Forum](https://discussions.udacity.com/c/nd004-p7-linux-based-server-configuration) <br>
[Digital Ocean Instructions for initial setup](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04) <br>
[Digital Ocean Instructions to Deploy Flask Applications](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) <br>
[Digital Ocean Instructions to Install Postgresql](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)<br>
[Amazing readme](https://github.com/anumsh/Linux-Server-Configuration)<br>
