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

After some research I ended up choosing Mumble-Django, because lately I've 
been learning a little python and got interested in setting up the Django web-
framework, so there's my chance. The only problem was to set it up with Nginx, 
as I found no tutorials, only for setting it with Apache or Lighthttp.

Without further ado, here's how I set it up on my VPS.
<!-- More -->

### Some disclaimer first

I installed Mumble-Django on my server somewhat following the _manual 
installation [guide][1]_ from the official homepage, with some sparse 
information helping to choose and learn how to use Gunicorn (Green Unicorn) 
WSGI HTTP server built with python.
Other detail is that I'm doing all the installation and configuration in my 
VPS with Debian 6.0 (Squeeze), some steps or details might change in other 
linux distributions.

Preparing the Environment
-------------------------

First of all, I wanted to setup a secure and easy to handle Python environment.
The best solution I found was to use [virtualenv][2] tool and run it from source
. So, let's get our hands with the initial setup.

First, download the source from the [official page][2] on pypi wiki and 
initialize the Env.
```
$ cd virtualenv
$ python virtualenv.py MD
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
locally the libraries that I installed globally. So I link the necessary libs 
to the _lib_ folder in my Env.
``` 
$ ln -s /usr/share/pyshared/PIL lib/python2.6/PIL
$ ln -s /usr/share/pyshared/simplejson lib/python2.6/simplejson
$ ln -s /usr/share/pyshared/Ice* lib/python2.6/
$ ln -s /usr/lib/pyshared/python2.6/IcePy.so lib/python2.6/
```

Installing Mumble-Django
------------------------

The next step is to download the lastest version of the Mumble-Django from 
their [repository][3]. Go to "Tags" tab and download the version you want. I 
recommend to download from the __Tag__ _stable_. Then we download and 
uncompress it to our Env folder and do the initial setup.
```
$ wget https://bitbucket.org/Svedrin/mumble-django/get/stable.tar.gz
$ tar xvfz stable.tar.gz # rename the folder extracted to what you want
$ cd mumble-django/pyweb # I renamed mine to mumble-django
$ python manage.py syncdb
```
__Note:__ Don't forget to do this after using the ___activate___ script or 
using the python provided in the ___env's bin___ folder.

Configuring Nginx and Gunicorn
------------------------------
After you configured your mumble-django accordingly, it's time to deploy 
gunicorn in Nginx. Let's start with Nginx config file.
``` nginx mumble-django.conf
server {
     listen      80;
     server_name mumble.edrago.me;
 
     location / {
         proxy_pass http://127.0.0.1:9999;
     }
 
     location /static {
         alias /home/eddie/virtualenv-1.9.1/mumble-django/app/htdocs;
     }
 
     location /static/admin {
         alias /home/eddie/virtualenv-1.9.1/mumble-django/lib/python2.6/site-packages/django/contrib/admin/static/admin/;
     }
 
     location /mumble/media {
         alias /home/eddie/virtualenv-1.9.1/mumble-django/app/pyweb/mumble/media/;
     }
}
```

<!-- Link References -->
  [1]: http://docs.mumble-django.org/en/installation.html#manual-installation
  [2]: https://pypi.python.org/pypi/virtualenv
  [3]: https://bitbucket.org/Svedrin/mumble-django/downloads
  [D-Bus]: http://docs.mumble-django.org/en/connecting_murmur_to_dbus.html#en-connecting-dbus "Connecting Murmur to DBus"
  [Ice]: http://docs.mumble-django.org/en/connecting_murmur_to_ice.html#en-connecting-ice "Making Murmur available via Ice"
