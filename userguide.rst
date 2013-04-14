.. highlight:: bash

User guide
==========


Demo
----

http://xxx.com/

Design goals
------------

* simple is better than complicated
* inline documentation
* convention over configuration
* can act as view interface only (optional configuration functionality)


Installation
------------

Fetchopia will install everything inside one directory, which must be empty. Application is meant to be self-contained,
meaning no additional administrator is needed besides upgrading to a newer version. 

Prerequisities
**************

* Scrapy 0.16 installed
* installed python2.6 or python2.7
* using postgresql: make sure `postgresql.conf` includes a line `client_encoding = utf8`


Installer (interactive)
***********************


Install prerequesities (Debian based)::

    $ sudo apt-get install git python-distribute gcc python-dev wget 

.. note::

    Installer will ask you few questions about SQL database and configuration for bconsole.

Install fetchopia (recommended: under same user as xxx)::

    $ cd /some/empty/directory/to/install/xxx/
    $ sh -xec "$(wget -O - https://raw.github.com/xxx/xxx/master/install_production.sh)"

You can continue with configuring :ref:`nginx`.


Manual (not recommended)
************************

For security and \*unix freaks, here is a step by step description what interactive install does behind the scene. 
Taking manual steps to install will ensure you are missing lovely time with your beloved one this weekend and replacing that with mild headache (specially if you are not familiar with python deployment quirks).

Interactive install also handles upgrades transparently. Fetchopia is developed with agile workflow with small incremental versions of new changes. 
You will have to dig in yourself how to upgrade environment upon a new release. Still stubborn? Let's go!

- Download latests release from github (as tarball or git clone) from *latests* branch (install_production.sh takes care of that otherwise)

- unpack into empty directory

- Make sure you use python2.7 or python2.6, since other versions are not supported

- Populate *production.ini* file for daemon configuration, taken from *buildout.d/templates/production.ini.in* (*xxx/scripts/configure_deploy.py* takes care of that interactively, then runs buildout to output configuration file from the template)

- Install all python dependencies from *setup.py* file, preferably within virtualenv (buildout takes care of that and pins them down to known workable set of versions, found in *buildout.d/versions.cfg*)

- Once you have installed dependencies (with `python setup.py install`) inside virtualenv or system python (REALLY not recommended), you should have *pserve* binary installed in bin/ directory

- run it like so: `bin/pserve production.ini`

- make sure daemon runs at reboot, configure log rotation

Happy? Let's see until first upgrade.

.. _nginx:

Configuring Nginx as a frontend
*******************************

It is wise to use frontend HTTP server and proxy HTTP requests to python web server. Following is an example for nginx, you could also use papache2 or lighthttpd. 

You would normally put this in /etc/nginx/sites-enabled/xxx.mywebsite.com.conf

.. code-block:: nginx 

    server {
        listen          80;
        server_name     xxx.mywebsite.com;

        location / {
                proxy_pass http://localhost:2500;

                proxy_set_header  X-Real-IP  $remote_addr;
                proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header  Host $http_host;
                proxy_redirect off;

                # optional authentication - recommended
                auth_basic "Restricted";
                # how to correctly write htpasswd: http://wiki.nginx.org/HttpAuthBasicModule#auth_basic_user_file
                auth_basic_user_file /some/directory/to/install/xxx/.htpasswd;

        }
    }

Then run::

    $ /etc/init.d/nginx reload

Now try to access http://xxx.mywebsite.com/ (if you have an error, follow instructions at :ref:`reporting-bugs`)


Configuring Apache2 as a frontend
*********************************

.. code-block:: apache

    <VirtualHost *:80>

    ServerName xxx.mydomain.com
    DocumentRoot "/var/www/xxx.mydomain.com"

    ProxyPreserveHost On

    <Location />
    ProxyPass  http://xxx.mydomain.com:2500/
    ProxyPassReverse http://xxx.mydomain.com:2500/ 
    </Location>

    ErrorLog /var/log/httpd/xxx.mydomain.com-error.log
    CustomLog /var/log/httpd/xxx.mydomain.com-access.log combined

    </VirtualHost>

Do not forget to restrict access to xxx, either by IP or by username/password.

Upgrading to a newer release
----------------------------

Run::

    $ cd xxx_install_directory
    $ git pull
    $ python bootstrap.py
    $ bin/buildout
    $ bin/supervisorctl shutdown
    $ bin/supervisord

You can also use that in crontab to auto upgrade on new releases, if you are crazy enough. You would probably extra check if upgrade is needed, something like running following and checking for any output::

    $ git log latests..origin/latests

.. _reporting-bugs:

Reporting bugs
--------------

Check if an issue already exists at https://github.com/xxx/issues,
otherwise add new one with following information: 

* bacula-director version, operating system and browser version
* include screenshot if it provides any useful information
* pastebin (http://paste2.org) output of $ cat XXX_ROOT/var/logs/xxx-stderr*
* pastebin XXX_ROOT/buildout.cfg, but be careful to *remove any sensitive data*


Filesystem structure
--------------------

TODO ;)
