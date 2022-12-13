---
layout: post
title: "Acurite sensors connected to ProxMox deployed HA"
---

## My setup

I recently deployed a ProxMox Home Server and decided to install Home Assistant. The configuration is build on an old Dell 3620T with 16GB memory installed. Platform runs on Debian 11 with ProxMox PVE7.3.3.
On ProxMox I have at the moment 3 lxc's and 3 VM's installed.
Virtual Machines:
- Home Assistant 9.3
- RTLSDR
- Allstar

Linux Containers:
- SysLog
- MQTT
- NodeRed

##Githubs used for this integration

On the RTLSDR Virtual Machine I followed this github until the docker section. https://github.com/mverleun/RTL433-to-mqtt
My config.py:
## Config section
## Fill in the next 2 lines if your MQTT server expected authentication
MQTT_USER=""
MQTT_PASS=""
MQTT_HOST="192.168.1.19"
MQTT_PORT=1883
MQTT_TOPIC="sensors/rtl_433"
MQTT_QOS=0
DEBUG=False # Change to True to log all MQTT messages
## End config section




