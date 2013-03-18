---
layout: post
title: "Installing Mumble-Django under virtualenv and behind Nginx"
date: 2013-03-11 10:20
comments: true
categories: [Python, Mumble, Nginx]
---

So, I've been using murmur (the mumble server) on my VPS to host a server where 
my friends can gather and talk freely for some time now. But none of then has
much expertise using technical computer programs, and I've been looking into a
front-end interface that would offer an easy way to some of then manage the
server, as I can't be online all the time.

After some research I ended up choosing Mumble-Django, because lately I've been
learning a little python and got interested in setting up the Django web-
framework, so there's my chance. The only problem was to set it up with Nginx,
as I found no tutorials, only for setting it with Apache or Lighthttp.

Without further ado, here's how I set it up on my VPS.
<!-- More -->

### Some disclaimer first

I installed Mumble-Django on my server somewhat following the _manual
installation [guide][1]_ from the official homepage, with some sparse
information helping to choose and learn how to use Gunicorn (Green Unicorn) WSGI
HTTP server built with python.  
Other detail is that I'm doing all the installation and configuration in my VPS
with Debian 6.0 (Squeeze), some steps or details may change in other linux
distributions.
It's presumed that the reader has some basic knowledge of Linux and Nginx.

Preparing the Environment
-------------------------

First of all, I wanted to setup a secure and easy to handle Python environment.
The best solution I found was to use [virtualenv][2] tool and run it from source
. So, let's get our hands with the initial setup.

First, download the source from the [official page][2] on pypi wiki, uncompress
it and initialize the Env. Install the Mumble server (_murmur_) if you don't
have it already.
```
$ cd virtualenv
$ python virtualenv.py MD
$ aptitude install mumble-server # For Debian
```

After that, use the _activate_ script that's present on the bin folder of the
'MD' env so you can use the contents of env's _bin_ as functions.  
This isn't necessary, is just for convenience. Then install some dependencies.
```
$ source MD/bin/activate
  # Pil needs to be compiled in the process and might protest due to some 
  # dependencies, I installed my distro package, but you could try to install
  # it with pip
  # i.e. pip install pil
$ aptitude install python-imaging # PIL
$ aptitude install python-simplejson # Same as PIL, 'pip install simplejson'
$ pip install django==1.2.3
$ pip install django-registration
$ pip install gunicorn
```

Now, Mumble-Django needs some way to communicate with the murmur process and we 
have two options. Using [D-Bus][] or [Ice][]. I chose Ice, but you can pick
whatever method that you think it's best.

If you chose Ice you should install the _zeroc_ library. In debian it comes in
the package _"python-zeroc-ice"_.

Because I am installing everything under virtualenv, it's necessary to provide
locally the libraries that I installed globally. So I link the necessary libs to
the ___lib___ folder in my Env.
``` 
$ ln -s /usr/share/pyshared/PIL lib/python2.6/PIL
$ ln -s /usr/share/pyshared/simplejson lib/python2.6/simplejson
$ ln -s /usr/share/pyshared/Ice* lib/python2.6/
$ ln -s /usr/lib/pyshared/python2.6/IcePy.so lib/python2.6/
```

Installing Mumble-Django
------------------------

The next step is to download the lastest version of the Mumble-Django from their
[repository][3]. Go to "Tags" tab and download the version you want. I recommendto download from the __Tag__ _stable_.  
Then we download and uncompress it to our Env folder and do the initial setup.
```
$ wget https://bitbucket.org/Svedrin/mumble-django/get/stable.tar.gz
$ tar xvfz stable.tar.gz # rename the folder extracted to what you want
$ cd mumble-django/pyweb # I renamed mine to mumble-django
  # If you are using Ice, make sure that the path to Slice configured in settings.py
  # (variables SLICE and SLICEDIR) points to the correct location of Murmur.ice
  # file. If using Murmur 1.2.3 or higher you can just use getslice as below
$ python manage.py getslice
$ python manage.py syncdb
$ python manage.py checkenv # To check if everything is configured correctly
```
__Note:__ Don't forget to do this after using the ___activate___ script or using
the python provided in the ___env's bin___ folder.  
__Note2:__ Create an admin user when asked or run _python manage.py
createsuperuser_ later. This is used to login to the admin system.  
__Note3:__ If you are doing this on your machine, you can see the page running
by executing the command _"python manage.py runserver 0.0.0.0:8000"_ and access
it in your browser with _localhost:8000_.
__Note4:__ To disable the __activate__ script, just run the command __deactivate__.

Configuring Nginx and Gunicorn
------------------------------
After you configured your mumble-django accordingly, it's time to deploy
gunicorn through Nginx. Let's start with Nginx config file. The location of the
file depends on your distro.
``` nginx mumble-django.conf
server {
     listen      80;
     server_name your.domain.com;
     location / {
         proxy_pass http://127.0.0.1:8888;
     }
     location /static {
         alias /home/user/virtualenv-1.9.1/MD/mumble-django/htdocs;
     }
     location /static/admin {
         alias /home/user/virtualenv-1.9.1/MD/lib/python2.6/site-packages/django/contrib/admin/media/;
     }
     location /mumble/media {
         alias /home/user/virtualenv-1.9.1/MD/mumble-django/pyweb/mumble/media/;
     }
}
```
__Note:__ You can insert this directly in _nginx.conf_ file or source it from
another file, like I did using _mumble-django.conf_ file.

Now we'll start the server using gunicorn. You need to go to the folder that
contains the _mumble-django.wsgi_ file (At the time, in the root of the Mumble
Django project folder.
```
$ cd virtualenv/MD/mumble-django
$ ../bin/gunicorn_django -w 3 -b 127.0.0.1:8888
```
Now your server should be running correctly and you should be able access it
from your browser.  
The _-w_ parameter defines the number of worker processes that will serve the
requests.

<!-- Link References -->
  [1]: http://docs.mumble-django.org/en/installation.html#manual-installation
  [2]: https://pypi.python.org/pypi/virtualenv
  [3]: https://bitbucket.org/Svedrin/mumble-django/downloads
  [D-Bus]: http://docs.mumble-django.org/en/connecting_murmur_to_dbus.html#en-connecting-dbus "Connecting Murmur to DBus"
  [Ice]: http://docs.mumble-django.org/en/connecting_murmur_to_ice.html#en-connecting-ice "Making Murmur available via Ice"
