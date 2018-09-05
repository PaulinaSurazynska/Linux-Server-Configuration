# Linux Server Configration Project

This project is about a baseline installation of Ubuntu Linux on a virtual machine to host a Flask web application. This includes the installation of updates, securing the system from a number of attack vectors and installing/configuring web and database servers.

# Technical details:
- Server IP Address: 35.178.169.235
- Application URL: http://35.178.169.235.xip.io
- SSH login username: grader

# Step by step 
###  1 Get the server

This linux server is created on [Amazon Lightsail](https://aws.amazon.com/) that provides all the details on setting up Lightsail instance (e.i instructions to SSH into your server)
### 2 Secure the server 
First logged in into the server ( as ubuntu user - default user) by running this command from local machine 
```
ssh ubuntu@35.178.169.235 -p 2200 -i~/.ssh/privatekeyfile.pem
```
(side note:  `.ssh/privatekeyfile.pem` - it's a file where private key for this user is stored)
within successful login:

![screen shot 2018-09-05 at 16 08 57](https://user-images.githubusercontent.com/24628087/45118147-7e0b4280-b14f-11e8-99d7-4a7b047440f3.png)

Once logged in it's time to make this server secure:
#### 2.1 Update all currently installed packages by running this command:
```
apt update && apt upgrade
```
#### 2.2 Change the SSH port from 22 to 2200
 Open the `/etc/ssh/sshd_config` (using `nano` in the command will open file in nano text editor) :
``` 
   # nano /etc/ssh/sshd_config
```
#### 2.3.Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123) using commands:
```
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
```
enable firewall by:
```
sudo ufw enable
```
and check the status ( just in case;) with command:
```
sudo ufw status
```
as a result:
```
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
123/udp                    ALLOW       Anywhere                  
2200/tcp (v6)              ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
123/udp (v6)               ALLOW       Anywhere (v6) 
```
#### 2.4 Configure Timezone to Use UTC
 In order to do it run `sudo dpkg-reconfigure tzdata`, choose `None of the Above`
![screen shot 2018-09-02 at 21 48 54](https://user-images.githubusercontent.com/24628087/45118352-2f11dd00-b150-11e8-9d56-201fcbdb40fd.png)

and then `UTC`:

![screen shot 2018-09-02 at 21 49 03](https://user-images.githubusercontent.com/24628087/45118392-4b157e80-b150-11e8-9b34-cd4fe8628db9.png)

### 3 Create `grader` user and give him `sudo` access

To create a user run:

``` 
sudo adduser grander
```

with following output:

``` 
Adding user `grader' ...
  Adding new group `grader' (1000) ...
  Adding new user `grader' (1000) with group `grader' ...
  Creating home directory `/home/grader' ...
  Copying files from `/etc/skel' ...
  Enter new UNIX password:
  Retype new UNIX password:
  passwd: password updated successfully
  Changing the user information for grader
  Enter the new value, or press ENTER for the default
	  Full Name []: Grader
	  Room Number []:
	  Work Phone []:
	  Home Phone []:
	  Other []:
  Is the information correct? [Y/n]
  ```
  now it's time to give him some rights in this server ;), so run:
  
  ```
  sudo usermod -aG sudo grader
  ```
  create ssh key for grader:
  - first swith the accounts by running:
  ```
  su - grader
  ```
  - create files where key will be stored:
  ```
  mkdir .ssh
  cd .ssh/
  touch authorized_keys
 ```
-  and secure them so only this user will have access to them:
```
chmod 700 .ssh
chmod 644 authorized_keys
```
- on local machine run `ssh-keygen` command:

![sdsd](https://user-images.githubusercontent.com/24628087/45118466-7bf5b380-b150-11e8-9eed-522bc93a50a9.png)

- after that copy and paste the public key to the server's `authorized_keys` file using nano or any other text editor, and save.

## 4 Prepare server to host your application

#### 4.1 Install and configure Apache to serve a Python mod_wsgi application.
``` 
sudo apt update
sudo apt install apache2
```
with a successfull instalation, if you go to `http://35.178.169.235` you should see:

![apache](https://user-images.githubusercontent.com/24628087/45118546-ae071580-b150-11e8-83e8-bf9761a96511.png)

as application that will be installed on the server is written in Python version 2 install pip package:

```
sudo apt install python-pip
```

you also need  `mod_wsgi` for serving Python apps from Apache, so run:

```
sudo apt install libapache2-mod-wsgi
```

after successful instalation restart apache with this command:
```
sudo service apache2 restart
```

#### 4.2 install PostgreSQL

(more inf about this step you can find [here](http://terokarvinen.com/2016/deploy-flask-python3-on-apache2-ubuntu))
##### 4.2.1 you need to create the file:

`nano /etc/apt/sources.list.d/pgdg.list`

##### 4.2.1 add the following line to it:

`deb http://apt.postgresql.org/pub/repos/apt/ bionic-pgdg main`

##### 4.2.3 Import the repository signing key, and update the package lists:

```
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt update
```

##### 4.2.4 install PostgreSQL:

```
sudo apt install postgresql-10
```
##### 4.2.5 and configure it:

- First log in as the user postgres (it was automatically created during the installation):

```
sudo su - postgres
```

- Open the psql shell:

```
psql
```

- In the psql shell type as follows:

```
postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;
postgres=# ALTER ROLE catalog WITH PASSWORD 'yourpassword';
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
Then exit from the terminal by running \q followed by exit.
```

- and finally exit psql terminal by typing `\q`

#### 4.3 Install `mod_wsgi` by running:

```
sudo apt install libapache2-mod-wsgi
```

and again restar apache (`sudo service apache2 restart`)

### 5 Install and Configure Git
(general information about git installation you can find [here](https://help.github.com/articles/set-up-git/#platform-linux))
what you need is:
#### 5.1 Install Git:
`sudo apt-get install git`
#### 5.2 Set your name, e.g. for the commits:
`git config --global user.name "YOUR NAME"`
#### 5.3 Set up your email address to connect your commits to your account:
`git config --global user.email "YOUR EMAIL ADDRESS"`
#### 6 Clone git repo and with the Catalog Flask application
### 6.1 First change directory to `/var/www/`:
```
cd /var/www/
```
### 6.2 Create a directory called FlaskApp and change the working directory to it:
```
sudo mkdir FlaskApp
cd FlaskApp/
```
### 6.3 clone GitHub repository of Catalog application project (Flask project) as the directory FlaskApp.
```
sudo git clone https://github.com/PaulinaSurazynska/catalog-project b develop FlaskApp
```
### 6.4 change the current working directory to the newly created directory:
```
cd FlaskApp/
```
### 6.5 If you want to check file structure run `tree` command in the terminal (*if you don't have it just run `sudo apt-get install tree` to install )
you should get this result in the terminal:
```ubuntu@ip-172-26-9-202:/var/www/FlaskApp$ tree
.
└── FlaskApp
    ├── catalog.db
    ├── catalog.py
    ├── database_setup.py
    ├── database_setup.pyc
    ├── flaskapp.wsgi
    ├── google_client_secret.json
    ├── README.md
    ├── static
    │   └── style.css
    └── templates
        ├── 404.html
        ├── countries.html
        ├── countries-public.html
        ├── country-cities.html
        ├── country-cities-public.html
        ├── delete-city.html
        ├── delete-country.html
        ├── edit-city.html
        ├── edit-country.html
        ├── header.html
        ├── login.html
        ├── main.html
        ├── new-city.html
        └── new-country.html

3 directories, 22 files
```

but not so fast .. remember when you installed `pip` package? .. now ( in order to properly run freshly cloned app you still need few things on your server), run this command to install all required packages:
```
sudo pip install --upgrade Flask SQLAlchemy httplib2 oauth2client requests psycopg2 psycopg2-binary`
```
### 7 Set Up the VirtualHost Configuration
7.1 in order to set up a file called FlaskApp.conf to configure the virtual hosts run this command in the terminal:
```
sudo nano /etc/apache2/sites-available/FlaskApp.conf
```
7.2 Add the following lines to it:
```
<VirtualHost *:80>
   ServerName YOUR SERVER IP
   ServerAlias YOUR SEVER IP.xip.io
   ServerAdmin youremail@domain.com
   WSGIScriptAlias / /var/www/FlaskApp/FlaskApp/flaskapp.wsgi
   <Directory /var/www/FlaskApp/FlaskApp/>
       Require all granted
   </Directory>
   Alias /static /var/www/FlaskApp/FlaskApp/static
   <Directory /var/www/FlaskApp/FlaskApp/static/>
       Require all granted
   </Directory>
   ErrorLog ${APACHE_LOG_DIR}/error.log
   LogLevel warn
   CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

7.3 Enable the virtual host:
```
sudo a2ensite FlaskApp
```
7.4 Restart Apache server:
```
sudo service apache2 restart
```

### 8 Now if in your browser you pass application URL, ( in this case is ) `http://35.178.169.235.xip.io`
TADAM!! :D
![alt text](https://lh3.googleusercontent.com/F8AEUvZJhPNQZhCpnrnfQVEihCPL9suzz1w_5_CQTbu5cOmt3RwwFRMUg6WhJXRFQlQxnS8-__z-JNVWGc231-I53wdtXtbJwwBfXVwwkDp53oELPYlwRhqJtRjed_Kq0LzhJxW_IioNoQxd1mwSwkRJjkJfs2Y3Xc0MWIqgzR8XV1Una_9cHuJ0H4W4eQ6FBhOUeqbG0XjQlATN0Ui9Ydoe7uo2FIH7yEjImHf3-xybDyAZ2SFFBBEzojgKfj1demCKAypA54Jy3S05n3JAqM0ymVc_6cc62gTYd51-tFaxlZgt_GZ4psHH939WyZR_5mZDG2PbYK8uR14yDoLuwfUJzx5KA0eKIK09Tmd0b6iigySNdT4DWrS_JrIzWjuh4tcs3e9LUdcPmef898bH0TpAkfnT3gEcd5Mih5VfuSELFI90JAcinBSJHZTuv-OzAO6eUxzgcjuzAIHJX9fLH-oW6jeLqnKko-RXScRJX5YvdGw2-tnhH14WMuYC-q3dYhLxL1dyQ3AviOkT4W8ykN7KjL82m3gqG0-SiMiOpeWzueagscyTIRMzejVqmrc118viU1LqW8URNCdOlzq3esPERYumnzZ-vFxKV8dO6XjnT33RacaK-w-2xfuq1kzr=w1258-h803-no)

something went wrong?!
run this command in the terminal:
`sudo cat /var/log/apache2/error.log`

# 9 Thank you and enjoy!
