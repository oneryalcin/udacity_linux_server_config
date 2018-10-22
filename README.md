# Udacity Project: Linux Server Configuration

In this Full-Stack Nanodegree project in summary I did the followingto successfully deploy my Udacity Item Catalog WebApp to remote Linux server:
- Spin up an Ubuntu server on Cloud. It was Amazon AWS Lightsail in my case.
- Update Server
- Configure security
- Install applications and libraries
- Install Udacity Item Catalog App
- Configure your application and Google Oauth so webapp can be accessible from internet.

### Spinning Up Ubuntu Server
I have created an Ubuntu 16.04 server, using Amazon AWS Lightsail service. Server is in London availability zone. Server public IPv4 address is : **35.178.28.30**

### Update Server
I have updated the packages by `sudo apt-get update && sudo apt-get upgrade` command. 

### Configure Security
- Using `ufw` i configured my server to only allow TCP port `80` for `http`,  TCP port  `2200` for `ssh` and UDP port `123` for `ntp`. All other port for incoming traffic is blocked. 
 - Amazon Lightsail by default onlly allows TCP ports `22` and `80`. I configured Amazon Lightsail firewalls accorfing to the parameters explained above.
 - I have created a `grader` user and added to `sudoers` list by adding a file under `/etc/sudoers.d/` dir. 
 - I have disabled password authentication for `SSH`. Only key based authentication is allowed. In `/etc/ssh/sshd_config` I changed the default ssh port from `22` to `2200`, and restarted the  `SSH` service. 
 - I generated `SSH` key pairs for user `grader`. Grader must use private key to authenticate and login to the server. 
 - I added `grader` public  key to `/home/grader/.ssh/authorized_keys` file, set file permissions `644` for `authorized_keys` file and `700` for `/home/grader/.ssh` dir 
 
 ### Install Applications and Libraries
 I installed :
 -	`git`, to clone `Udacity Item Catalog Application`, 
 -	`apache2` as Web Server
 -	`libapache2-mod-wsgi-py3` for mod wsgi
 -	`python3-pip` for installing required libraries for my python application defined in `requirements.txt` file.

### Install and Configure Udacity Item Catalog App
I cloned my Udacity Item Catalog app from`https://github.com/oneryalcin/udacity_product_app_project` into `/var/www/html`. I had to change a few things:
 
 -	Changes app name from `app.py` to `myapp.wsgi`. 
 -	Run `setup.sh` , so it does flask migration, creates all database and tables and populates few catalog items. One thing to note that I don't need to configure `Postgresql` on the server as Flask app uses `sqlite` as backend and flask migrate does all the work. 
 -	Added following lines to `myapp.wsgi` so Apache can run my flask application.
 ```python
import sys
sys.path.insert(0,"/var/www/html")
from catalog_app import app as application	
```  

I configured Apache to use mod wsgi to pointed my application. I edited `/etc/apache2/sites-enabled/000-default.conf` and added `WSGIScriptAlias / /var/www/html/myapp.wsgi` line.  Restarted Apache2 server.

### Configure Google Oauth
My Udacity Item Catalog Application uses Google Oauth for authentication, and only `localhost` was allwed as authorized host in Google Oauth setup in Google Cloud console. I had to add few more endpoints to Google Oauth so that users can get authenticated when they visit my app page. I added http://35.178.28.30.xip.io to trusted domains. 
