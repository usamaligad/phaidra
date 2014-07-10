Phaidra
===
Phaidra is that system that powers the Historical Language eLearning Project. It will provide distributed eLearning for historical languages such as Greek and Latin, so that more people can delve deeply into primary sources in their original language.

License
===
Phaidra and Ancient Geek, by the Open Philology Project, is licensed under the Creative Commons Attribution-ShareAlike 4.0 International license:
https://creativecommons.org/licenses/by-sa/4.0/

Installing Phaidra
===
These instructions will assume you are running Ubuntu 12.04, and that you are installing Phaidra in `/opt/phaidra`.

Push directly to OpenPhilology/Phaidra
---
*Only for authorized contributors*

		$ cd /opt
		$ git clone git@github.com:OpenPhilology/phaidra.git

Or use our https clone url.
This will create a directory at `/opt/phaidra`. 

Fork from Github
---
On this page, click "Fork" in the upper right-hand corner.

Clone the project to your local machine.

		$ cd /opt
		$ git clone https://github.com/YOUR_USERNAME/phaidra.git
		$ git remote add upstream https://github.com/YOUR_USERNAME/phaidra.git
		$ git fetch upstream

This will create a directory at `/opt/phaidra`. 

Set up Virtualenv
---

		$ sudo apt-get install python-virtualenv 
		$ cd /opt/phaidra
		# The name of our virtualenv is "env"
		$ virtualenv --no-site-packages env

You will need permission on /opt directory.

Install and Configure Postgres
---

		$ apt-get build-dep python-psycopg2
		$ apt-get install postgresql postgresql-contrib
		$ sudo -u postgres psql postgres
		$ \password  # Set as you like, our test setup will use "phaidra"
		$ \q
		$ sudo -u postgres createdb phaidra

Install Requirements
---
Presuming you are starting with a new machine, you need to install Python-related tools. Hint: You might don't wanna install or at least run the application with superuser permissions.

		$ source env/bin/activate
		$ sudo pip install -r requirements.txt
		$ deactivate

Install and configure Nginx with Uwsgi
---
Phaidra runs on nginx as its web server.

		$ sudo apt-get install nginx uwsgi uwsgi-plugin-python python2.7-dev 

Make sure you are running the latest version of Uwsgi (at time of writing, 1.9) with `uwsgi --version`. If this gives you an older version, you will need to upgrade by doing the following:

		$ pip install -U uwsgi
		$ cd /usr/bin
		$ mv uwsgi uwsgi-old
		$ ln -s /usr/local/bin/uwsgi uwsgi
		
		# Test the version again
		$ uwsgi --version
		1.9

Open `/opt/phaidra/extras/nginx/phaidra.conf` and set the variables to be appropriate for your server:

		# Can also just be an IP Address
		server_name YOUR_SERVER.com www.YOUR_SERVER.com 

Next we must create a symlink so Nginx uses our configuration file:

		$ ln -s /opt/phaidra/extras/nginx/phaidra.conf /etc/nginx/sites-enabled/phaidra.conf

And do the same to enable Uwsgi:

		$ ln -s /opt/phaidra/extras/uwsgi/phaidra.ini /etc/uwsgi/apps-enabled/phaidra.ini

Create a safe space for the socket to exist, which can be accessed by both Nginx and Uwsgi:

		$ sudo mkdir /var/uwsgi
		$ sudo chown www-data:www-data /var/uwsgi

Touch necessarily files so that Uwsgi can write to them.

		$ mkdir /opt/phaidra/logs
		$ touch /opt/phaidra/logs/master.pid
		$ mkdir /var/log/uwsgi
		$ touch /var/log/uwsgi/phaidra_daemon.log

Start up services:

		$ sudo service nginx start
		$ uwsgi /opt/phaidra/extras/uwsgi/phaidra.ini
		
* Check that everything has worked up to this point by navigating to http://localhost:8000 *

Set up initial migration with South
---
Make sure the virtual env is activated.

		$ cd /opt/phaidra
		$ source env/bin/activate

First, comment out the apps 'app' and 'tastypie' from INSTALLED_APPS as well as 'AUTH_USER_MODEL' in phaidra/settings.py. We have to do some finangling to get South to play nicely with our custom user model.

Then run:

		$ ./manage.py syncdb

Uncomment 'apps', 'tastypie', and 'AUTH_USER_MODEL' in phaidra/settings.py. Then run:

		$ ./manage.py schemamigration app --initial
		$ ./manage.py schemamigration tastypie --initial
		$ ./manage.py migrate app
		$ ./manage.py migrate tastypie

If you get errors about tables already existing, you can run these commands to fix the South history and keep the tables from trying to re-create themselves:

		$ ./manage.py migrate app --no-initial-data
		$ ./manage.py migrate tastypie --fake 0001


Install and configure the Neo4j database
---
Make sure you installed java 1.7. Do this outside your virtualenv.

		$ java -version

If you have an earlier version of Java or no Java at all, install it as you prefer (e.g. [Installing Java 7 on Ubuntu](http://www.cyberciti.biz/faq/howto-installing-oracle-java7-on-ubuntu-linux/)) or follow:
		
		$ sudo add-apt-repository ppa:webupd8team/java
		$ sudo apt-get update
		$ sudo apt-get install oracle-java7-installer
		$ sudo update-java-alternatives -s java-7-oracle

Install Neo4j (see [Neo4j Downloads](http://www.neo4j.org/download)):

		$ wget -O - http://debian.neo4j.org/neotechnology.gpg.key | apt-key add - 
		$ echo 'deb http://debian.neo4j.org/repo stable/' > /etc/apt/sources.list.d/neo4j.list
		$ sudo apt-get update
		$ sudo apt-get install neo4j
		$ service neo4j-service start

If you are developing remotely, you must uncomment the following line in `/var/lib/neo4j/conf/neo4j-server.properties` so that you can access the web interface from outside the VM:

		#allow any client to connect
		org.neo4j.server.webserver.address=0.0.0.0
