---
title: "Pianoteq in a Raspberry Pi (headless)"
date: 2020-07-17
---
My Yamaha DXG660 digital piano has a great set of features for a very reasonable price.
The sound synthesizer is quite basic though, so after buying a [Pianoteq Stage](https://www.modartt.com/pianoteq) 
license, I usually pracise using the output of my computer audio interface connected to the line input of my keyboard while running Pianoteq.
I really don't want to either boot my computer every single time I want to play or have it always on, and Pianoteq has a 
native Linux/ARM version, so I decided to install the softare in a new [Raspberry Pi 4B](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/). 
In this document I describ the steps that I followed in order to have everything setup in a way that makes it quick and easy for me to practise with the much nicer sound that Pianoteq provides, without having to rely on my personal computer being on at all times.

## Materials
* Raspberry PI 4B (I used the 4GB model)
* SD Card (at least 32GB)
* USB Hi-res DAC with a male 3.5 jack adapter
* USB-C power cable and adapter
* USB-B to USB-A cable to connect the keyboard to the Pi

## Steps

### Preparing the SD Card
In this step we will install a minimal operating system to run our software.

* Install [Raspberry Pi Imager](https://www.raspberrypi.org/downloads/) 
* Click on **Choose OS** and from the **Raspberry Pi OS (Other)** section, select **Raspberry Pi OS Lite (32-bit)**
* Select you SD Card and finally click on **Write**. It shouldn't take more than a few minutes.
* We should have a new partition called `boot` (remove and reinsert the SD card if not)

## Setting up the SD Card
We will setup the Raspberry Pi to enable the SSH service at boot, and optionally connect to a WiFi access point
(alternatively, we can just connect a Ethernet cable if so desired). This way we won't need an external monitor or a 
keyboard.

* We will create two new files in the **boot** partition root directory: `ssh` and `wpa_supplicant.conf`
* The `ssh` file can contain anything or just be empty, if you have a UNIX terminal, you can simple type `touch ssh` in the `/boot` directory.
* The `wpa_supplicant.conf` file must contain the following:
```country=<you two-letter country code e.g. US>
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
scan_ssid=1
ssid="<your access point SSID>"
psk="<your password>"
}
```
* Safely extract you SD Card from the computer

## Installing Pianoteq on to Pi
* Insert the SD Card in the Raspberry Pi.
* Find the IP address our DHCP server assigned to the Pi (you can just try running `ping raspberry` in your local terminal).
* SSH into the Pi by typing `ssh pi@raspberry` in your terminal (the default password is `raspberry`).
* Login in the [Pianoteq User Area](https://www.modartt.com/user_area).
* Copy the **serial number** and download the Pianoteq Linux binary from the **Downloads** section (we will assume it's called `pianoteq_stage_linux.7z`).
* Use SCP to copy the Linux binary file over to the Pi: `scp pianoteq_stage_linux.7z pi@raspberry:`
* Install P7Zip in the Pi: `apt-get install p7zip`
* Extract the files from the archive: `p7zip -d pianoteq_stage_linux.7z`
* At this point, you should have in your `/home/pi` directory a new directory called `Pianoteq 6 STAGE`. 

## Creating a boot service for Pianoteq
With this service, Piano will run as soon as we boot up the device.

* Create a new system service: `sudo systemctl --full --force edit pianoteq.service`
* Copy and paste the following in the new file:
```
[Unit]
Description=Pianoteq

[Service]
Type=simple
User=pi
Group=audio
ExecStart="/home/pi/Pianoteq 6 STAGE/arm/Pianoteq 6 STAGE" --headless
LimitRTPRIO=90
LimitMEMLOCK=500000
LimitNICE=-10

[Install]
WantedBy=multi-user.target
```
* Enable the service: `sudo systemctl enable pianoteq.service`

## Creating a boot service for the CPU governor
In order to avoid pops and clicks, we will force the first CPU to run always at maximum speed
* Create a new system service: `sudo systemctl --full --force edit governor@.service`
* Copy and paste the following in the new file:
```
[Unit]
Description=Set CPU0 governor to %i
Before=pianoteq.service

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/bin/sh -c 'echo %i > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor'

[Install]
WantedBy=pianoteq.service
```
* Enable the service: `sudo systemctl enable governor@performance.service`

## Shuting down the Pi when powering off the keyboard
We don't want for the Pi to be always running, and powering it off without shuting it down first will eventually result in a corrupted filesystem. 
We will create a `udev` rule to shutdown the Pi when we power off the keyboard. Alternatively, we can just just start and stop Pianoteq when attaching and removing a keyboard. Please note that Pianoteq launches on boot, regardless of the digital piano being connected.

* Figure out the USB vendor/model id: `lsusb`
* In our case is `ID 0499:161d Yamaha`
* Create a new set of rules: `sudo nano /lib/udev/rules.d/80-pianoteq.rules`
* Copy and paste the lines below:
```
ACTION=="add", SUBSYSTEM=="usb", ENV{ID_VENDOR_ID}=="0499", ENV{ID_MODEL_ID}=="161d", RUN+="/bin/systemctl start pianoteq"
ACTION=="remove", SUBSYSTEM=="sound", ENV{ID_VENDOR_ID}=="0499", ENV{ID_MODEL_ID}=="161d", RUN+="/sbin/poweroff"
#ACTION=="remove", SUBSYSTEM=="sound", ENV{ID_VENDOR_ID}=="0499", ENV{ID_MODEL_ID}=="161d", RUN+="/bin/systemctl stop pianoteq"
```
* Reload the rules: `sudo udevadm control --reload-rules`

## Setting up Pianoteq
In order to setup and activate the license for Pianoteq we can't use the headless version. 
We don't really need a desktop manager to do this, but we can install one optionally if desired. 

* Install a vnc server: `sudo apt-get install tightvncserver`
* Optionally install a desktop manager such as lxde: `sudo apt-get instal lxde`
* Start it: `vncserver :1`
* In your computer download and install a [VNC client](https://www.realvnc.com/en/connect/download/viewer/)
* Connect to the Pi VNC server at `raspberry:1`
* Stop Pianoteq headless: `sudo systemctl stop pianoteq` and start the GUI version: `DISPLAY=:1 /home/pi/Pianoteq\ 6\ STAGE/arm/Pianoteq\ 6\ STAGE`
* Proceed with activation process and regular setup.
