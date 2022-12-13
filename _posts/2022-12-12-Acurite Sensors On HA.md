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

