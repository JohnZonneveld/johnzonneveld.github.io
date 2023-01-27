---
layout: post
title: "Autogrid - WSJT-X - gpsd"
---

Great work has be done by Brian Moran in the past to create a working python script to automatically update the gridsquare in WSJT-X in Windows (https://github.com/bmo/py-wsjtx). As far as I understand after plugging in a USB GPS dongle on a windows based system it resembles a serial port. In his script he added a todo for Linux, but for some reason it never happened. Looking at the date of his work, it was mainly done a little over 4 years ago in 2018.

Most of my operations are mobile so my grid is changing almost every time I head out to do another POTA activation. As my windows laptop has some kind of battery issue that wasn't resolved by replacing the battery I was looking for other ways. My other option was using my MacBook, but problem arises there that after a while I would have to charge that too.
As I recently did acquire a generator both option shouldn't be a real issue anymore.

I also looked at both a RaspberryPi 3B that I have in my posession and a recently bough Inovato Quadra. Both run their own Linux distro but both are based on Debian.

Until now when I went to a park I was using my phone and the Android app HamGPS to determine my GridSquare.

Back to the python script (I did install python3 on my Quadra), there is a slight difference between the way Brian followed and how gpsd works in linux. In his script a connection to the serial port is opened and the script is reading the $GPGLL NMEA0183 messages. With gpsd it is possible to open a stream with gpspipe to send the stream of NMEA messages to for example stdout. still have to dig a little bit further to look at this to see if and how it can be send somewhere else and treat it like the serial stream Brian is using.
Gpsd however standard sends out the position in decimal notation, so a lot of conversion Brian is doing in his script is not needed. Part of his script is already able to translate from decimal position notation to a gridsquare.

I modified the NMEAlocation class and especially the function handleSerial. I renamed the function to convert and modified it. I did remove some error handling for now. Main goal at first was to get things working.

Full NMEAlocation class:
```python
class NMEALocation(object):
    # As we are using gpsd there is no need to parse the NMEA message
    # we read the GPS value with gpsp and convert the decimal position to a GRID

    def __init__(self, grid_changed_callback = None):
        self.valid = False
        self.grid = ""                  # this is an attribute, so allegedly doesn't need locking when used with multiple threads.
        self.last_fix_at = None
        self.grid_changed_callback = grid_changed_callback

    def convert(self,lat,lon):
        # should be a single line.
        if (lat != None and lon != None):
            grid = pywsjtx.extra.latlong_to_grid_square.LatLongToGridSquare.to_grid(lat,lon)
            if grid != "":
                self.valid = True
                self.last_fix_at = datetime.utcnow()
            else:
                self.valid = False

            if grid != "" and self.grid != grid:
                logging.debug("NMEALocation - grid mismatch old: {} new: {}".format(self.grid,grid))
                self.grid = grid
                if (self.grid_changed_callback): # if the  calculated grid is different callback will be called to set the new grid
                    c_thr = threading.Thread(target=self.grid_changed_callback, args=(grid,), kwargs={})
                    c_thr.start()
```

Only changed part in this is the function convert which was originally called serial_handler. As we don't handle the serial anymore I renamed it and simplified it to adapt for gpsd that is already sending the decimal position:
```python
def convert(self,lat,lon):
    # should be a single line.
    if (lat != None and lon != None):
        grid = pywsjtx.extra.latlong_to_grid_square.LatLongToGridSquare.to_grid(lat,lon)
        if grid != "":
            self.valid = True
            self.last_fix_at = datetime.utcnow()
        else:
            self.valid = False

        if grid != "" and self.grid != grid:
            logging.debug("NMEALocation - grid mismatch old: {} new: {}".format(self.grid,grid))
            self.grid = grid
            if (self.grid_changed_callback): # if the  calculated grid is different callback will be called to set the new grid
                c_thr = threading.Thread(target=self.grid_changed_callback, args=(grid,), kwargs={})
                c_thr.start()
```

I also added Gpspoller to the code, this is as the name already mentions polling the GPS. This will be used to read the gpsd data. Code was found both on GitHub and StackOverflow.

```python
class GpsPoller(threading.Thread):

    def __init__(self):
        threading.Thread.__init__(self)
        self.session = gps(mode=WATCH_ENABLE)
        self.current_value = None
        self.running = True

    def get_current_value(self):
        return self.current_value

    def run(self):
        try:
            while self.running:
                self.current_value = self.session.next()
        except StopIteration:
            pass

```

For GpsPoller we have to create a thread and start it. That we do with the following code.
```python
gpsp=GpsPoller() # Create the thread
gpsp.start() # start it up
```

The main program looks like followed. It is mainly Brian's script modified for gpsd
```python
while True:

    (pkt, addr_port) = s.rx_packet()
    if (pkt != None):
        the_packet = pywsjtx.WSJTXPacketClassFactory.from_udp_packet(addr_port, pkt)
        if wsjtx_id is None and (type(the_packet) == pywsjtx.HeartBeatPacket):
            # we have an instance of WSJTX
            print("wsjtx detected, id is {}".format(the_packet.wsjtx_id))
            print("starting gps monitoring")
            wsjtx_id = the_packet.wsjtx_id
            nmea_p = NMEALocation(example_callback)
            report = gpsp.get_current_value()
            if len(list(report.keys())) > 5:
                lat = report.get('lat')
                lon = report.get('lon')
                print('lat: ',lat,' lon: ', lon)
                nmea_p.convert(lat,lon)
        if type(the_packet) == pywsjtx.StatusPacket:
            print(' wsjtx grid: ',the_packet.de_grid)
            if gps_grid != "" and the_packet.de_grid.lower() != gps_grid.lower():
                print("Sending Grid Change to wsjtx-x, old grid:{} new grid: {}".format(the_packet.de_grid, gps_grid))
                grid_change_packet = pywsjtx.LocationChangePacket.Builder(wsjtx_id, "GRID:"+gps_grid)
                logging.debug(pywsjtx.PacketUtil.hexdump(grid_change_packet))
                s.send_packet(the_packet.addr_port, grid_change_packet)

signal.signal(signal.SIGINT, sigint_handler)
```

As long as the GPS doesn't have a fix gpsp only report 5 key,value components that don't carry a lot of useful information for us. For that I added the comparision to see if the length of the report is greater than 5.
```python 
if len(list(report.keys())) > 5:
```
