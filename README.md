# Raspberry Pi LTE security camera

Welcome to this small project

The whole motivation behind it was to get a small LTE security camera for the garden for which we do not need to pay a lot of money.
We do not have wifi in the allotment, so the choice was using the mobile network to alert us on any movements instead. 

![Finished Raspberry Pi LTE securiy camera in a wood box](pictures/finished.png?raw=true "RPI LTE security camera")

What you will need : 
* A Raspberry Pi single board computer (or other comparable SBCs, for this project I am using a Raspberry Pi Zero W)
* A micro SD card (which we use to install Raspbian (Debian 11) 32 bit OS, I have gone with 32GB to have some space for media files)
* A micro USB cable to provide power to our SBC (I chose a 1A USB wall charger)
* A LTE modem compatiable with your SBC (for this project I have chosen the SIM7600X-4G-HAT (would also have been possible to choose the specific variant for the Pi Zero (SIM7600G-H)))
* A USB cable to connect the LTE Modem to our SBC (Had to use this as I was not being able to get the LTE connection working without it, only got GSM working with IO pins)
* A SIM card to be used with the LTE modem (I have chosen to go the cheapest german mobile phone provider plan (~200 MB free per month but getting personalized advertisment))
* A Raspberry Pi camera (I used the Raspberry Pi Zero camera without IR filter)
* A container for everything (I chose a small and tightly fitting wood box for this)

Software we will make use of : 
* motioneye (used to operate our security camera) - https://github.com/motioneye-project/motioneye
* telegram (we will setup a telegram bot for our Raspberry Pi)
* telegram-send (we will configure motioneye to sent us text/video using telegram-send) - https://pypi.org/project/telegram-send/
* teleterm (to be able to send commands to our security camera once its running and not) - https://github.com/alfiankan/teleterm

Optional additions : 
* fail2ban
* ddns

If you have the space I would recommend to use a Raspberry Pi 3 or 4 together with a SSD for storage instead of a SD card.



## The manual setup

The first step is to copy an operating system image on our SD card. For convenience reasons I decided for : Raspbian 11 (bullseye) 32 bit Desktop (Server instead of Desktop would also have worked out easily)
Connect your screen/keyboard/mouse to your Raspberry Pi and boot it up and go through the initial configuration.
(For the username I chose "ubuntu", I also configured the wifi connection for installing our required updates and software)

Using the raspi-config I further configured follow settings :
* Enable serial interface ports but disable serial console
* Enable ssh interface
* Enable camera interface
* Change boot to console instead of UI

The next step is to do an initial upgrade (might take some time on the Pi Zero, might be worth to run it in a termux or screen session)

```
apt-get update
apt-get upgrade -y
``` 

Second step is to configure our LTE modem :

Assuming you have already prepared the following : 
* SIM card with PIN disabled
* Connected the SIM7600X-4G-HAT using the IO pins
* Connected a USB cable from Raspberry Pi towards the LTE modem.

![Not yet finished Raspberry Pi LTE securiy camera in a wood box](pictures/finished.png?raw=true "Not yet ready RPI LTE security camera")

Also see https://www.waveshare.com/wiki/SIM7600E-H_4G_HAT

