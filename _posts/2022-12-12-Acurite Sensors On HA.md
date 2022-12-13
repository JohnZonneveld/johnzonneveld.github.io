---
layout: post
title: "Acurite sensors - MQTT - HA on ProxMox"
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

Acurite Sensor --> RTLSDR --> MQTT <--> HomeAssistant

## RTLSDR VM

To be able to attach the USB dongle to the RTLSDR server it has to be a virtual machine, USB forwarding is not possible in a LXC. For this machine I installed Debian 11 without any desktop environment.

On the RTLSDR Virtual Machine I followed this github until the docker section. https://github.com/mverleun/RTL433-to-mqtt
My config.py:

##Config section   
##Fill in the next 2 lines if your MQTT server expected authentication  
MQTT_USER=""  
MQTT_PASS=""  
MQTT_HOST="192.168.1.19"  
MQTT_PORT=1883  
MQTT_TOPIC="sensors/rtl_433"  
MQTT_QOS=0  
DEBUG=False # Change to True to log all MQTT messages  
##End config section  

To run rtl_433 I configured a service in /etc/systemd/system/rtl433.service

[Unit]  
Description=rtl_433 SDR Receiver Daemon  
StartLimitIntervalSec=5  
After=syslog.target network.target  
  
[Service]  
type=exec  
ExecStart=/usr/bin/python3 rtl2mqtt.py  
Restart=always  
RestartSec=30s  
User=root  
  
[Install]  
WantedBy=multi-user.target   

After this you have to run 'systemctl daemon-reload' and to start the service 'systemctl start rtl433'. To
start it up on boot run 'systemctl enable rtl433'.

With this configuration the messages coming from the sensors are being send to MQTT.

## MQTT

Because my machine is not exposed to the outside world I removed the authorization for MQTT.

Configuration is found in '/etc/mosquitto/conf.d/default.conv

persistence true  
listener 1883  
allow_anonymous true  

No other configuration is needed for the MQTT machine.
With MQTT Explorer you can monitor the message arriving at the server.

## Home Assistant

This section was the most confusing, there is plenty of information available but it might not be completely up to date anymore.

the RTLSDR server is sending the messages from all received sensors to the same MQTT_TOPIC that is configured in config.py in my case that is 'sensors/rtl_433'.

So we have to separate the messages per sensor in HA. For this we configure an Automation.
automations.yaml
- id: '1670900354958'  
  alias: rtl_433 demultiplexer  
  trigger:  
    platform: mqtt  
    topic: sensors/rtl_433  
  action:   
  - service: mqtt.publish  
    data:   
      topic: "mqttsensor{{ trigger.payload_json.id }}"  
      payload: "{{ trigger.payload }}"   
      retain: true  
      
Because I created the automation in HA it is automatically assigned an unique ID.
This automation is monitoring the mqtt messages with the name sensors/rtl_433, the json attached to these messages is the payload.

sensors/rtl_433/Acurite-606TX/162/time 2022-12-12 22:56:02  
sensors/rtl_433/Acurite-606TX/162/id 162  
sensors/rtl_433/Acurite-606TX/162/battery_ok 1  
sensors/rtl_433/Acurite-606TX/162/temperature_C 23.5  
sensors/rtl_433/Acurite-606TX/162/mic CHECKSUM   
sensors/rtl_433 {"time" : "2022-12-12 22:56:33", "model" : "Acurite-606TX", "id" : 162, "battery_ok" : 1, "temperature_C" : 23.500, "mic" : "CHECKSUM"}  

The topic is set as mqttsensor concatenated with the id of the sensor. So the above sensor will have a topic of mqttsensor162. And it will be accompanied with its payload.  

When this automation is saved and the HA restarted, you can see the trigger activity when you open the automation. Every time a message comes in a trigger message will be displayed.  

The automation also writes the messages back to MQTT.  

Last thing to configure is the configuration.yaml
mqtt:  
  sensor:  
    - state_topic: "mqttsensor162"  
      value_template: "{{ ((value_json.temperature_C * 9/5)+32)| round(1) }}"  
      name: "Sensor Livingroom"  
      unit_of_measurement: '°F'  
    - state_topic: "mqttsensor9217"  
      value_template: "{{((value_json.temperature_C * 9/5)+32)| round(1)}}"  
      name: "Sensor Outside Temp"  
      unit_of_measurement: '°F'  
    - state_topic: "mqttsensor9217"  
      value_template: "{{value_json.humidity}}"  
      name: "Sensor Outside Humidity"  
      unit_of_measurement: '%'  
      
As shown in this config you see two sensors, one of my neighbors has a Acurite-Tower on his/her balcony. and it is beaconing it messages on the same frequency. So we just listen in. As you can see that sensor 9217 also has a humidity sensor.
For both sensors we convert the temperature from Celsius to Fahrenheit and add the unit symbol.


