---
layout: post
title: "Oldschool multiplayer goodness with DOSBox"
date: 2013-03-08 13:55
comments: true
categories: [Games, Oldschool, DOSBox]
---

  So, yesterday me and some friends were talking about how cool some of our
childhood games were, and trying to remember the name of some of those titles
or googling in search of these. Problem is, fond memories like these tends to
distort the reality, and trying to find a title that you even wasn't capable of
understanding a half-dozen words and by a few equivocated details...
You see, it doesn't work all that well.

  I digress.

  Anyway... At one point we were talking about a game that most of us happened to
had played, and agreed that it could bring lots of fun to play some multiplayer\
matchs, specially because of the fact that we had played only single-player at
the time.

  The game in question is __Constructor__.

  ![Cover art](http://upload.wikimedia.org/wikipedia/en/9/9a/Constructor-pc.jpg "Constructor Cover")

But as this is a DOS/Win95 game, it generally don’t run in modern systems. This problem is solved by utilizing a x86/DOS emulator program, like [DOSBox](http://www.dosbox.com/). Other setback is that it uses an old network protocol for multiplayer, called IPX. So let’s get this old gem going.
<!-- more -->

###FYI

First, I just want to make clear that these steps propably work with various old games as long as they use the IPX network protocol (like Doom, for example), but I don’t tested with other titles to know for sure. Maybe with for some just the installation process will work.

Installing and running old games with DOSBox
--------------------------------------------

I will use windows path style to explaim the process, but you just need to adapt it according to your OS.

Fist we need to mount the location that we want to install the game, for simplicity sake I chose C:
```
Z:\>mount c "c:\"
```

I am using an ISO image because I find it to be more convenient and created a ‘constructor’ folder beforehand, but you could mount your CD drive too. To mount an CD image:
```
Z:\>imgmount d "path\to\image.iso" -t iso
```

Or your physical drive:
```
Z:\>mount d "D:\" -t cdrom
```

We need to change our drive. Then, proceed to install the game from the disc.
```
Z:\>D:
D:\>Install.exe
# Then we can open the folder that you installed the game and test if worked
C:\>Game.exe
```

__NOTE:__ Constructor needs the CD to execute and searchs for it in the ‘D:’ drive. So you need to mount it using this letter everytime you want to start the game.

Great, the game is up and running. Now we just need to setup the IPX emulation. The best part? DOSBox ships with that feature already.

Getting multiplayer to work
---------------------------

Now that we have our game set up, we are going to enable the IPX emulation in DOSBox, that comes disabled by default.

First, close DOSBox if it’s running and open the configuration file, that can be found in the following directory in Windows 7 (For other OS information look [here](http://www.dosbox.com/wiki/Dosbox.conf)):

_C:\Users\\__\<Your Username\>__\AppData\Local\DOSBox\dosbox.conf_

Now search for a line that says _“ipx=false”_ at the end of the file and change the value to true.

Finally, open DOSBox again and type the following command:
```
# Port is optional, default is 213
Z:\>ipxnet startserver [port]
```

You should receive a _“IPX Tunneling Server started”_ message. Now your friends can connect to your computer through your IP address, via LAN or Internet:
```
# Again, port is optional, if you started with a custom one, inform your friend
Z:\>ipxnet connect <ip> [port]
```

If you have any problems with connection, make sure that you don’t have any firewall blocking DOSBox or that the correct port is [being forwarded](http://portforward.com/) by your router.

Open the game again and enjoy your afternoon lost robbing your firend’s HQ, sending a Hippie to protest in front of his factory and Tenants that never were happy once in their lives.

###_Fim_
