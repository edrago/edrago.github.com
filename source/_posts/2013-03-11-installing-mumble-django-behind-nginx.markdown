---
layout: post
title: "Installing Mumble-Django behind Nginx"
date: 2013-03-11 10:20
comments: true
categories: Python, Mumble, Nginx
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
$ python virtualenv.py mumble-django
```

After that move to the bin folder of the 'mumble-django' env and install some 
dependencies.
```
$ cd mumble-django/bin
  # Pil needs to be compiled in the process and might protest due to some 
  # dependencies, so a package for your distro could be installed.
  # i.e. 'aptitude install python-imaging' (for debian)
$ ./pip install pil 
$ ./pip install simplejson # Same as pil, package 'python-simplejson' in debian
$ ./pip install django
$ ./pip install django-registration
$ ./pip install gunicorn
```

Now, Mumble-Django needs some way to communicate with the murmur process and we 
have two options. Using [D-Bus][] or [Ice][]. I chose Ice, but you can pick 
whatever method that you think it's best.

After you configured your mumble server according to one of the two previous 
links

<!-- Link References -->
  [1]: http://docs.mumble-django.org/en/installation.html#manual-installation
  [2]: https://pypi.python.org/packages/source/v/virtualenv/virtualenv-1.9.1.tar.gz
  [D-Bus]: http://docs.mumble-django.org/en/connecting_murmur_to_dbus.html#en-connecting-dbus "Connecting Murmur to DBus"
  [Ice]: http://docs.mumble-django.org/en/connecting_murmur_to_ice.html#en-connecting-ice "Making Murmur available via Ice"
