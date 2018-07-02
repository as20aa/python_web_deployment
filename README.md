# python_web_deployment
Configure environment for depoly python web application

# Environment
* CPU: 1GHz
* RAM: 1G
* OS:  Centos 7 x64
# Requirements
* Python 3.6
* Virtualenv
* Uwsgi
* Nginx
* Flask
# Prepare
* Install Python 3.6
```bash
# Install dependencies
yum install openssl openssl-devel
yum install zlib
# Download the Pyhotn 3.6 source code version
wget https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tgz
# Unzip file
tar zxvf Python-3.6.5.tgz
# Go into the directory
cd Python-3.6.5
# Change the option and install with openssl
vim Modules/Setup.dist
```
Then you will go into the vim prompt,just find the ssl and uncomment just like below
```bash
# Socket module helper for SSL support; you must comment out the other
# socket line above, and possibly edit the SSL variable:
#SSL=/usr/local/ssl
_ssl _ssl.c \
        -DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
        -L$(SSL)/lib -lssl -lcrypto
```
and then go on
```bash
# Configure the install path
# You have to on the root directory of Python-3.6
./configure --prefix=usr
make
make install
```
Then verify your installaiton
```bash
python3
>>>import ssl
>>>quit()
```
If the command above didn't informat error then you had installed python3.6 successfully
The next step is to install virtualenv
```bash
# We use python3 -m before pip to avoid confliction of python2.7 and python3.6
# This command will use python3.6's pip and will not affect the python2.7
python3 -m pip install virtualenv
# Create a virtual environment
# We use the virtualenv to make sure some operation will not affect the system's python
python3 -m virtualenv myproject
# Activate the virtual environment
source myproject/bin/activate
```
If you had activated the virtualenv sucessfully, you can see the prompt like this
```bash
(myproject) root$:
```
You can quit the virtualenv via typing
```bash
deactivate
```
Don't quit the virtual environment and install the essential python packages
```bash
# The Uwsgi framework is a python web server framework
pip install uwsgi
# The flask is a framework of python web application
pip install flask
# Install pandas for operating file of data
pip install pandas
# install pandas will auto-install numpy
# Sklearn is a highly-intergated api of machine learning
pip install sklearn
```
The last step of prepare is installing Nginx
```bash
# Create a repo file to install nginx from nginx.org
vi /etc/yum.repos.d/nginx.repo
```
Add the follow lines to nginx.repo
```bash
[nginx]
name=nginx repo
baseurl=https://nginx.org/packages/mainline/centos/7/$basearch/
gpgcheck=0
enabled=1
```
Update yum
```bash
yum update
```
Install Nginx
```bash
# After install the nginx repo, you can install nginx directly
yum install nginx
```
Now all the preparation was done, We can begin to configure nginx and uwsgi
# Configure
Before configure our environment, we create a python web application for test
just cd /myproject and activate the virtualenv myproject
```bash
vim hello.py
```
Add lines to hello.py
```python
from flask import Flask
application = Flask(__name__)
@application.route('/')

def hello_word():
        return 'Hello World!'

if __name__ == '__main__'
        application.run(host = '127.0.0.1',port = 8080)
```
Save and exit, use curl to test the code above
```bash
# Start the flask application
python hello.py
```
Open another ssh and type in
```bash
# The IP address must be the same as the paramaters in the hello.py
curl -I 127.0.0.1:8080
```
If you see the prompt below you had create a executable python web application
```bash
HTTP/1.0 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 40
Server: Werkzeug/0.14.1 Python/3.6.5
Date: Wed, 27 Jun 2018 06:17:34 GMT
```
(Press ctrl+c to stop the flask web application)
We begin to configure the uwsgi for the hello.py to use the unix socket to commucate with nginx
```bash
# Create a configure file for uwsgi, we assume you are in the same directory as hello.py
vim uwsgi.ini
```
Add lines to uwsgi.ini
```bash
[uwsgi]
# The socket pointes to the pipe file which commucate with the network
# It will auto-create this file
socket = /root/evaluation/online/hello.sock
# Set the permission for the sock file, and must be 666 not 660
chmod-socket = 666
# Set the vacuum true, the uwsgi program will delete the sock file after it terminal
vacuum = true
# Set the wsgi file to point the python web application file
wsgi-file = wsgi.py
```
And then add lines to nginx.conf
```bash
 server {
        # The port you want to open for others to visit
        listen 80;
        # The server name
        server_name localhost;
        location / {
            # Set uwsgi_params to use uwsgi
            include uwsgi_params;
            # Set the web file directory
            root /root/evaluation/online;
            # Set the commucate file you use between uwsgi and nginx, must be the same as the uwsgi.ini
            uwsgi_pass unix:/tmp/uwsgi.sock;
        }
    }
```
Start the uwsgi and reload the nginx, if you got the right web page, congraturation!
