---
title: Unreliable homeplug adaptors suck
date:   2017-09-17 
layout: post
comments: true
---

What's the issue?
=================

I've been struggling with my HomePlug adaptors pretty much ever since I first got some.  They seem like a great solution for getting connectivity in your house but they never seem to deliver.  For me the problem is simply their reliability.  From time to time one of them will simply stop working and needs to be powered off and back on again, after which things are good for a while.

Mostly I can handle this but more recently I've set up a VPN server at home so that I can do sensitive things like internet banking while I'm away from home.  The VPN is running on a raspberry pi and relies on a HomePlug to get connected to the router.  Guess what.  It keeps falling off the network, around once every week or two, and the offending HomePlug requires a restart before I can connect to the VPN again.

This simply isn't an acceptable situation when you're away from home.

Is it just me?
==============

I don't think so.  I asked my friend Google and there seem to be many many people suffering the same issue.  The advice is varied and only consistent in one thing - that none of it helped me:
* Make sure your HomePlug is plugged directly into a socket, not an extension or multisocket
* Make sure you're running the latest firmware
* Make sure you don't live in a house with old wiring
* Make sure the two sockets are on the same circuit
* Cut the front off your HomePlug to stop it overheating (yes, really, I found this advice and perhaps it works for some people)
* It's caused by powersaving so set up a regular ping that travels between the HomePlugs

In trying to solve this I figured that all I needed was some way to remotely reset a misbehaving HomePlug device.  It was surprising how hard it turned out to be to find something to do this. I was close to giving up and physically moving my VPN box to be directly by my router when I eventually found something that looked a bit promising.  There's an API out there that lets you communicate with these devices. It's called the [Open PLC Utils](https://github.com/qca/open-plc-utils) and one of the things it lets you do is remotely reset a HomePlug device.  It appears to be designed for Qualcomm Atheros based devices but it seems to work OK on all of mine (I have two TPLink and two DLink devices).

How do I use it?
================

The readme tells you exactly what you need to do once you have the toolkit downloaded.  I chose to set it up on a Pi Zero that happened to be lying around on the network.  The basic steps were
```
sudo yum install git
git clone https://github.com/qca/open-plc-utils.git
cd open-plc-utils
make
sudo make install
sudo make manuals
```
At this point you've got the toolkit installed on your box and available to do things with.  The main program I used has the slightly odd name _int6k_.

First I needed to locate all my HomePlug devices which was easy enough, although it does depend on you knowing which network interface your machine uses.  My Pi is using a wireless network and so I needed wlan0.  If you have a wired network perhaps you want eth0.

```
int6k -miwlan0

source address = C0:4A:00:69:AE:DE
  network->NID = E0:E0:AF:62:1F:D6:0F
  network->SNID = 12
  network->TEI = 1
  network->ROLE = 0x00 (STA)
  network->CCO_DA = B8:A3:86:08:34:12
  network->CCO_TEI = 15
  network->STATIONS = 3

    station->MAC = B8:A3:86:08:34:11
    station->TEI = 6
    station->BDA = B8:27:EB:25:34:5F
    station->AvgPHYDR_TX = 088 mbps
    station->AvgPHYDR_RX = 122 mbps

    station->MAC = B8:A3:86:08:34:12
    station->TEI = 15
    station->BDA = 00:08:9B:8B:F6:82
    station->AvgPHYDR_TX = 092 mbps
    station->AvgPHYDR_RX = 131 mbps

    station->MAC = C0:4A:00:69:23:D4
    station->TEI = 21
    station->BDA = E0:CB:4E:D8:E9:3C
    station->AvgPHYDR_TX = 166 mbps
    station->AvgPHYDR_RX = 179 mbps
```
From this listing we can see three entries for `station->MAC` and these correspond to three of the four devices in my network.  The fourth, well that's a little odd but it appears as the `source address` in the report.  So, from this report I now know the MAC addreses of my four devices.

```
C0:4A:00:69:AE:DE
B8:A3:86:08:34:11
B8:A3:86:08:34:12
C0:4A:00:69:23:D4
```

From here it turns out we can issue a very simple command to reset one of the devices
```
int6k -i wlan0 -R C0:4A:00:69:23:D4
```

Further, if you want to restart the world you can use multiple MAC addresses on one line like this (it's a bit hard to read but there's a space between each MAC address on the command line)
```
int6k -i wlan0 -R C0:4A:00:69:AE:DE B8:A3:86:08:34:11 B8:A3:86:08:34:12 C0:4A:00:69:23:D4
```

The only issue I had with this was that when I restarted multiple at the same time, they didn't always all receive the command I guess once one starts the restart operation the network breaks and the message doesn't reach the other ones.

The last thing I did was to set this up to run on a regular schedule, the idea being that if I restart them regularly they won't ever get overloaded enough to fail.  To run on a schedule on the Pi is easy, just use `cron`.

I used the command `crontab -e` to edit my cron table and added the following lines, which run a script at 02:00 every morning
```
# Homeplug restart
0 02 * * * /home/pi/code/picode/homeplugAV/restartAllHomePlug.sh
```

The script this calls was very simple too, just enough to restart all the devices
```
#!/bin/sh
(
date
echo Restarting network
/usr/local/bin/int6k -i wlan0 -R C0:4A:00:69:AE:DE 
sleep 30
/usr/local/bin/int6k -i wlan0 -R B8:A3:86:08:34:11 
sleep 30
/usr/local/bin/int6k -i wlan0 -R CB8:A3:86:08:34:12
sleep 30
/usr/local/bin/int6k -i wlan0 -R CC0:4A:00:69:23:D4
sleep 30
echo finished
) > /tmp/homeplugRestart.log 2>&1
```

It would be nice, of course, to have a more dynamic script that gets the list of devices and resets them one at a time, but for now this will do.  Does it solve the problem - I don't know yet.  So far things are good but it's early days.