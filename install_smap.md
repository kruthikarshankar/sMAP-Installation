-----------------------------------------------------------------------------------------------------------------
The procedure is reasonably explained in the sMAP website, but for a few more requirements. I am just giving a list of commands to use here. Just go with this step by step and you should get sMAP installed in a breeze.

Ubuntu 13.10 and 14.04 users: I have tried to cover most of the issues from my installation, but I still seem to find a few errors because of version differences. 

As specified in the website the installation is easy and quick for Ubuntu 12.04 users. 


## sMAP Library Installation

Open a terminal and type in these commands.

Install pip

```
#!unix

sudo apt-get install python-pip
```
Install smap

```
#!unix

sudo pip install smap
```
Install dependencies likes python, twisted, zope interface, avro, configobj4, python-dateutil, pyopenssl

```
#!unix

sudo apt-get install python python-zopeinterface python-twisted python-dateutil python-beautifulsoup
sudo pip install avro
sudo apt-get install python-configobj
```
Install subversion

```
#!unix

sudo apt-get install subversion
```
Check out smap code from subversion repo

```
#!unix

svn checkout http://smap-data.googlecode.com/svn/trunk/ smap-data-read-only
cd smap-data-read-only/python
sudo python setup.py install
```
Once you type these commands in terminal, you must have got smap installed. To test this, just type twistd in the terminal. You should be able to see a variety of options to run twistd with, one of which will be smap. This confirms successful installation of smap.

#sMAP Archiver and Front End Installation

This has been tested to work in Ubuntu version 11.10 and 12.04, for other versions it might or might not work. From my experience, it works partially in Ubuntu 13.10.

Create a new directory for all the sources. 

```
#!unix

mkdir ~/sources
cd ~/sources
```
Install monit.

```
#!unix
sudo apt-get install monit
sudo gedit /etc/monit/monitrc
```
Uncomment the following lines from the document and save it.

```
#!unix

set httpd port 2812 and
    use address localhost
    allow localhost

```
Do a restart of monit

```
#!unix

sudo /etc/init.d/monit restart
```
Install readingdb from the repository.

```
#!unix

sudo add-apt-repository ppa:stevedh/smap
sudo apt-get update

```
This might not work in Ubuntu 13.10, so goto the following file (Type sudo gedit /etc/apt/sources.list in the Terminal) and manually add the following entries.

```
#!unix

deb http://ppa.launchpad.net/stevedh/smap/ubuntu precise main 
deb-src http://ppa.launchpad.net/stevedh/smap/ubuntu precise main

```
Install readingdb.

```
#!unix

sudo apt-get install readingdb readingdb-python
```
The above command should successfully install readingdb. If this worked, you can skip to *Postgres Installation*. Otherwise continue.

Install readingdb’s dependencies.

```
#!unix
sudo apt-get install libdb4.8 libdb4.8-dev libprotobuf-c0   \
     libprotobuf-c0-dev protobuf-c-compiler zlib1g zlib1g-dev \
     build-essential autoconf libtool python python-dev       \
     python-numpy swig check


```
This command might throw an error saying libdb4.8 and libdb4.8-dev are not available. In this case, use libdb4.8++ for the former one, and for the latter, use libdb5.1-dev. This should resolve the issues for now, and the above command should run successfully.

Now checkout readingdb from git.

```
#!unix

git clone git://github.com/stevedh/readingdb.git
```
Goto readingdb’s folder. Type in the following commands from readingdb folder.

```
#!unix

autoreconf --install
./configure --prefix=/
make
sudo make install
cd iface_bin
make
sudo make install

```
Now start the service. A service config file is automatically created in /etc/monit/conf.d

```
#!unix

sudo monit reload
sudo monit start readingdb

```
At this point, you can check if it got started using the following command. The data for readingdb in /var/lib/readingdb folder.

```
#!unix

ps -eaf|grep readingdb
```
Install dependencies for installing Postgres.

```
#!unix

sudo apt-get install postgresql postgresql-contrib python-psycopg2
```
# Postgres Installation
Login and create a user for the archiver to use. 

```
#!unix

root@box$ sudo su postgres
postgres@box$ psql
postgres=# CREATE USER archiver WITH PASSWORD 'password';
postgres=# CREATE DATABASE archiver WITH OWNER archiver;
postgres=# \q
postgres@box$ psql archiver
postgres=# CREATE EXTENSION hstore;
postgres=# \q
postgres@box$ exit

```
We are ready to install the web front end now.
Install powerdb’s dependencies.

```
#!unix

sudo apt-get install subversion 
sudo pip install avro python-dateutil django-piston
sudo pip install Django==1.4

```
Checkout the powerdb2 project.

```
#!unix

cd ~/sources
svn checkout http://smap-data.googlecode.com/svn/branches/powerdb2
cd powerdb2/

```
Type sudo gedit settings.py to edit the file. Add the following lines after Databases entry.

```
#!unix

DATABASE_ENGINE = 'postgresql_psycopg2' 
DATABASE_NAME = 'archiver'                                                        
DATABASE_USER = 'archiver'                                    
DATABASE_PASSWORD = 'password'                                                            
DATABASE_HOST = 'localhost'

```
Save the file. Now type in the following command.

```
#!unix

python manage.py syncdb
python manage.py runserver
```
This should run the server. You can now goto localhost:8000 in your browser and login with the user name and password, that you created a while ago. This will do for now, but for later you might want to run the site inside of apache using mod_python and mod_wsgi.

Finally we come to archiver installation.

#Archiver Installation

Type in the following commands in the Terminal.

```
#!unix

cd ~/sources
sudo apt-get install python-twisted python-scipy
sudo pip install ply
svn checkout http://smap-data.googlecode.com/svn/trunk smap-data-read-only
cd smap-data-read-only/python
sudo python setup.py install

```
If all of the above commands went well, you should be able to run twistd in the command line and see options for both smap and smap-archiver in there. Copy some files to complete the setup.

```
#!unix

sudo mkdir /etc/smap
sudo cp conf/archiver.ini /etc/smap
sudo cp monit/archiver /etc/monit/conf.d

```
You should edit the archiver.ini file that you put under etc, so that the username and password data are available. Now you can reload monit and start the server.

```
#!unix

monit reload
monit start archiver

```
This completes the sMAP installation. 

#Few smap commands to get started with

The following command will start the twisted engine for the particular driver configuration.

```
#!unix

twistd -n smap <your_driver_file>.ini
```

**Kruthika Rathinavel 
2/27/2014**
