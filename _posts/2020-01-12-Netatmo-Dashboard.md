---
title: "Netatmo Dashboard"
categories:
  - Home Automation
tags:
  - Home Automation
  - Netatmo
toc: true
---

# Netatmo Database and Graph Visualization

This is a tutorial for setting up a Raspberry Pi to fetch and store Netatmo weather station data in a InfluxDB database. Information is also included for displaying the database information with Grafana on a kiosk style screen.

<img src="/assets/images/NetatmoDash.PNG">

Using:  
- Raspberry Pi
- Netatmo API
- InfluxDB
- Grafana  

## 1. Install Rasbian and Configure Pi

There are thousands of tutorials out there on how to do this. Google it and find one with the level of detail your experience requires. Here are the basic steps (mostly so I can remember):

1. Use balenaEtcher to prepare SD card with latest full version of Rasbian
2. Plug in networking, monitor, keyboard and power
3. Log in with user: pi password: raspberry
4. Change password
5. Expand file system
6. Turn on SSH  
7. ```sudo reboot```  
8. ```sudo apt update```  
9. ```sudo apt upgrade```  


## 2. Sign up for Netatmo App

You will need a ID and Key from Netatmo in order to use their API.

1. Go to [https://dev.netatmo.com/apps/createanapp](https://dev.netatmo.com/apps/createanapp) and follow the steps to sign up
2. Find your client ID and your client secret
3. Look through the API [documentation](https://dev.netatmo.com/apidocumentation/weather).

## 3. Install InfluxDB

1. Follow most of this [this](https://pimylifeup.com/raspberry-pi-influxdb/) tutorial
2. In the **Using InfluxDB on your Raspberry Pi** section only do steps 1 and 2
  1. Create a database using step 2 and name it whatever you want, just remember this name
3. Go to the **Adding Authentication to InfluxDB** section and do all steps
4. Now you should have an influx database up and running that will start on reboot

## 4. Write a script to handle the data

1. See [fetchAndWriteData.py](https://github.com/ScottEgan/NetatmoDataGather). This file was was adapted from github user [arnesund](https://gist.github.com/arnesund/29ffa1cdabacabe323d3bc45bc7db3fb)
2. I think netatmo stores data every 10min so set code to run every 8
   - Use [cron](https://www.raspberrypi.org/documentation/linux/usage/cron.md)
   - Example:  
   ```*/8 \* \* \* \* /home/pi/fetchAndWriteData.py >> /home/pi/netatmoLogs.txt```  
     The >> means that the script output will be appended to the .txt file that follows
  
## 5. Install Grafana

1. Follow [this](https://pimylifeup.com/raspberry-pi-grafana/) tutorial by pimylifeup  
2. Connect InfluxDB using default settings and your username and password from when you installed influx

## 6. Configure Grafana Dashboard

1. Play around with the queries and visualizations until you get something you are happy with
2. If you download a plugin you will need to restart the server with:

   - ```systemctl restart grafana-server```

1. Set the dashboard you just created as the home dashboard in Configuration->Preferences->Home Dashboard

   <img src="/assets/images/setAsDefault.PNG" align="middle">

## 7. Set Grafana to start in kiosk mode at startup

1. Use the instructions from [here](https://github.com/grafana/grafana-kiosk)
2. Use wget to grab the tar file from releases section  
   - Find URL of latest release gz  
   - ```wget URL```  
   - Extract the tar file with ```tar xvzf <path/to/tar.gz>```  
2. Pick a way to do automatic startup
   - I used the [Systemd](https://github.com/grafana/grafana-kiosk#systemd-startup) startup option
   - Use the first two commands to create the file and edit permissions:
   ```BASH
   $ sudo touch /etc/systemd/system/grafana-kiosk.service
   $ sudo chmod 664 /etc/systemd/system/grafana-kiosk.service
   ```
   - Use the following to open the file in a text editor:  
   ```
   $ sudo nano grafana-kiosk.service
   ```
   and then paste:  
   ```INI
   [Unit]
   Description=Grafana Kiosk
   Documentation=https://github.com/grafana/grafana-kiosk
   Documentation=https://grafana.com/blog/2019/05/02/grafana-tutorial-how-to-create-kiosks-to-display-dashboards-on-a-tv
   After=network.target

   [Service]
   User=pi
   Environment="DISPLAY=:0"
   Environment="XAUTHORITY=/home/pi/.Xauthority"
   ExecStart=/usr/bin/grafana-kiosk --URL <url>  -login-method local -username <username> -password <password> --kiosk-mode full --lxde

   [Install]
   WantedBy=graphical.targe
   ```
   - Make sure to personalize the ```ExecStart=``` section based on what you want to see and which options you want to use
   - Follow the rest of the instructions in that section

That should be everything

### References:  
[Netatmo API docs](https://dev.netatmo.com/apidocumentation/weather)   
[Pimylifeup tutorial on Influxdb](https://pimylifeup.com/raspberry-pi-influxdb/)  
[Pimylifeup tutorial on Grafana](https://pimylifeup.com/raspberry-pi-grafana/)  
[arnesund's original python file](https://arnesund.com/2016/07/10/visualize-your-netatmo-data-with-grafana/)  
[Grafana docs for plugins](https://grafana.com/docs/grafana/latest/plugins/installation/)  
[How to set up Grafana as a kioskk](https://github.com/grafana/grafana-kiosk)
