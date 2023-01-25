---
layout: post
title: "Autogrid - WSJT-X - gpsd"
---

Great work has be done by Brian Moran in the past to create a working python script to automatically update the gridsquare in WSJT-X in Windows (https://github.com/bmo/py-wsjtx). As far as I understand after plugging in a USB GPS dongle it resembles a serial port. In his script he added a todo for Linux, but for some reason it never happened. Looking at the date of his work, it was mainly done a little over 4 years ago in 2018.

Most of my operations are mobile so my grid is changing almost every time I head out to do another POTA activation. As my windows laptop has some kind of battery issue that wasn't resolved by replacing the battery I was looking for other ways. My other option was using my MacBook, but problem arises there that after a while I would have to charge that too.
As I recently did acquire a generator both option shouldn't be a real issue anymore.

I also looked at both a RaspberryPi 3B that I have in my posession and a recently bough Invato Quadra. Both run their own Linux distro but both are based on Debian.

Until now when I went to a park I was using my phone and the Android app HamGPS to determine my GridSquare.

Back to the python script (I did install python3 on my Quadra), there is a slight difference between the way Brian followed and how gpsd works in linux. In his script a connection to the serial port is opened and the script is reading the $GPGLL NMEA0183 messages. With gpsd it is possible to open a stream with gpspipe to send the stream of NMEA messages to for example stdout. still have to dig a little bit further to look at this to see if and how it can be send somewhere else and treat it like the serial stream Brian is using.
Gpsd however standard sends out the position in decimal notation, so a lot of conversion Brian is doing in his script is not needed. Part of his script is already able to translate from decimal position notation to a gridsquare.
