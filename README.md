# FSDN-P6
This project is part of Full Stack Developer Nanodegree

## About
Project 6: Linux Server Configuration

## Objective
Take a baseline installation of a Linux server and configure it to host a web applications. The server should be secured from a number of attack vectors. 
It is necessary to install and configure a database server and deploy one web applications onto it.

## Steps executed:

### AWS Instance Info
OS: Ubuntu <br />
Public IP: 18.221.43.122 <br />
Public DNS: ec2-18-221-43-122.us-east-2.compute.amazonaws.com <br />
Port: 2200 <br />
Connect using : ssh -i [path_key] grader@18.221.43.122 -p 2200

### User Management

* Create grader User: <br />
```sudo adduser grader```
* Add sudo to grader User: <br />
``` sudo nano /etc/sudoers.d/grader ```<br />
Insert the text ```grader ALL=(ALL) NOPASSWD:ALL ``` into the file and save it.

* On client machine generate ssh key: <br />
```ssh-keygen -t rsa```
* Add key in server authorized_keys
```cd /home/grader/```
```sudo mkdir .ssh```
copy the PUB key into it and save it ```sudo nano /home/grader/.ssh/authorized_keys```

* Change owner and group of the files to grader: <br />
```sudo chown grader .ssh``` <br />
```sudo chown grader .ssh/authorized_keys``` <br />
```sudo chgrp grader .ssh``` <br />
```sudo chgrp grader .ssh/authorized_keys``` <br />

* Remove remote root conection and password authentication (only use ssh key pair) <br />
```sudo nano /etc/ssh/sshd_config ``` <br />
change ```PasswordAuthentication to No``` and change ```PermitRootLogin to No``` <br />
```sudo service ssh restart```

### Security

* Update repositories: <br />
```sudo apt-get update ``` <br />
```sudo apt-get upgrade```
```sudo unattended-upgrades```

* Change ssh port to 2200: <br />
```sudo nano /etc/ssh/sshd_config ``` <br />
change the number ```22 to 2200```

* Firewall rules: <br />
```sudo ufw default deny incoming ``` <br />
```sudo ufw default deny outgoing ``` <br />
```sudo ufw allow 2200/tcp ``` <br />
```sudo ufw allow ntp ``` <br />
```sudo ufw allow www ``` <br />
```sudo ufw enable ``` <br />
```sudo ufw status```


### Application Functionality

* Configuring Database Postgresql: <br />
``` sudo apt-get install postgresql ``` <br />
(set db password as catalog) <br />
```sudo -u postgres createuser -P itemcatalog ``` <br />
```sudo -u postgres createdb -O itemcatalog itemcatalog```

* Download dependencies Apache/Python: <br />
```sudo apt-get install apache2 libapache2-mod-wsgi ``` <br />
```sudo apt-get install python-psycopg2 python-flask python-sqlalchemy python-pip``` <br />
```sudo pip install oauth2client requests httplib2 flask_wtf``` <br />

* Download and configuring project and GIT: <br />
```sudo apt-get install git``` <br />
```cd /var/www/``` <br />
```sudo mkdir itemcatalog``` <br />
```cd itemcatalog/``` <br />
```sudo git clone https://github.com/walternunes/FSDN-P4.git``` <br />
```cd FSDN-P4``` <br />
```sudo mv * ../``` <br />
```cd ..``` <br />
```sudo rm FSDN-P4 -r``` <br />

* Change DB connection configuration of the project: <br />
(change connections to postgresql://itemcatalog:catalog@localhost/itemcatalog) <br />
```sudo nano database_setup.py``` <br />
```sudo nano catalog_data.py``` <br />
```sudo nano catalog.py```

* Execute Python files to create and populate the database: <br />
```sudo python database_setup.py```  <br />
```sudo python catalog_data.py ```

* Configuring app.wsgi: <br />
```sudo nano app.wsgi``` <br />
Copy the following code into it and save it <br />
```
#!/usr/bin/python
import sys
sys.path.insert(0,"/var/www/itemcatalog/")
from catalog import app as application
application.secret_key = 'super_secret_key'
``` 

* Configuring apache conf: <br />
```sudo nano /etc/apache2/sites-enabled/000-default.conf``` <br />
copy inside virtualHost brackets: 
```WSGIScriptAlias / /var/www/itemcatalog/app.wsgi```

* Restart the service: <br />
```sudo apache2ctl restart```

### Resources
* [Flask mod WSGI Apache](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)
* [Unattended Upgrades](https://help.ubuntu.com/lts/serverguide/automatic-updates.html)
* [Install Postgresql](https://help.ubuntu.com/community/PostgreSQL)
* [AWS config](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html)

