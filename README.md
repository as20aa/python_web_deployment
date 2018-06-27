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
