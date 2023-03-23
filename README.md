# Raspberry Pi LTE security camera

Welcome to this small project.

The whole motivation behind it was to get a small LTE security camera for the garden for which we do not need to pay a lot of money.
We do not have wifi in the allotment, so the choice was using the mobile network to alert us on any movements instead. 

![Finished Raspberry Pi LTE securiy camera in a wood box](pictures/finished.png?raw=true "RPI LTE security camera")

What you will need : 
* A Raspberry Pi single board computer (or other comparable SBCs, for this project I am using a Raspberry Pi Zero W)
* A micro SD card (which we use to install Raspbian (Debian 11) 32 bit OS, I have gone with 32GB to have some space for media files)
* A micro USB cable to provide power to our SBC (I chose a 1A USB wall charger)
* A LTE modem compatible with your SBC (for this project I have chosen the SIM7600X-4G-HAT (would also have been possible to choose the specific variant for the Pi Zero (SIM7600G-H)))
* A USB cable to connect the LTE Modem to our SBC (Had to use this as I was not being able to get the LTE connection working without it, only got GSM working with IO pins)
* A SIM card to be used with the LTE modem (I have chosen to go the cheapest german mobile phone provider plan (~200 MB free per month but getting ads))
* A Raspberry Pi camera (I used the Raspberry Pi Zero camera without IR filter)
* A container for everything (I chose a small and tightly fitting wood box for this)

Software we will make use of : 
* motioneye (used to operate our security camera) - https://github.com/motioneye-project/motioneye
* telegram (we will setup a telegram bot for our Raspberry Pi)
* telegram-send (we will configure motioneye to sent us text/video using telegram-send) - https://pypi.org/project/telegram-send/
* teleterm (to be able to send commands to our security camera once its running and connected to the mobile network, a reverse SSH tunnel might also work but I wanted to save as much mobile traffic as possible) - https://github.com/alfiankan/teleterm

Optional additions : 
* fail2ban (might be obsolete as no ports are directly reachable through mobile network)
* dyndns (might be obsolete, it works to store the mobile IP but as no ports are open you gain nothing)

If you have the space I would recommend to use a Raspberry Pi 3 or 4 together with a SSD for storage instead of a SD card.
A lot of writes on the SD card will render it useless quite fast. You may also think about storing the media files of motioneye on an online storage instead.

## The manual setup

#### Operating system
The first step is to copy an operating system image on our SD card. For convenience reasons I decided for : Raspbian 11 (bullseye) 32 bit Desktop (Server instead of Desktop would also have worked out easily)
Connect your screen/keyboard/mouse to your Raspberry Pi and boot it up and go through the initial configuration.
(For the username I chose "ubuntu", I also configured the wifi connection for installing our required updates and software)

Using the raspi-config I further configured follow settings :
* Enable serial interface ports but disable serial console
* Enable ssh interface
* Enable camera interface
* Change boot to console instead of UI

Confirm to have "enable_uart=1" in your /boot/config.txt (or included files)

The next step is to do an initial upgrade (might take some time on the Pi Zero W, might be worth to run it in a termux or screen session).

```
apt-get update
apt-get upgrade -y
```

#### Setup LTE modem

Second step is to configure our LTE modem :

Assuming you have already prepared the following : 
* SIM card with PIN disabled
* Connected the SIM7600X-4G-HAT using the IO pins
* Set the correct jumper positions (to configure the module for raspberry pi and boot it up at the same time)
* Connected an USB cable from Raspberry Pi towards the LTE modem.

![Not yet finished Raspberry Pi LTE securiy camera in a wood box](pictures/unfinished.png?raw=true "Not yet ready RPI LTE security camera")

Also see https://www.waveshare.com/wiki/SIM7600E-H_4G_HAT

We download the sample code for the LTE modem from waveshare : https://www.waveshare.com/wiki/File:SIM7600X-4G-HAT-Demo.7z
(If the website is already down in the meantime hit me up, I might still have the necessary file lying around)

Afterwards we actually follow the tutorial to install the LTE modem beneath a Raspberry Pi, in summary : 

```
# install 7zip
apt-get install p7zip-full

# download demo files
https://www.waveshare.com/wiki/File:SIM7600X-4G-HAT-Demo.7z

# unpack the example code
7z x SIM7600X-4G-HAT-Demo.7z -r -o/home/ubuntu

# After decompression, rename the c folder under the Raspberry folder to SIM7600X and copy it to your home folder
chmod 777 sim7600_4G_hat_init

# add following line to /etc/rc.local : sh /home/ubuntu/SIM7600X/sim7600_4G_hat_init
# in the end the script sets a few IO pins to be able to communicate with the modem.
```

Optional : If you intend to play around with the scripts the manufacturer provides (e.g. sending SMS, Phone call or TCP tests) you can also compile the example code.
These scripts work by directly sending commands to the module. They are a good starting point if you want to setup more comples use-cases, scenarios.

```
# Go to the bcm2835 directory, compile and install it
chmod +x configure && ./configure && sudo make && sudo make install

# I had issues the last time, aclocal-1.13 apparently was not found
# had to use autoreconf --force --install to be able to compile the code

# If there are compilation problems you might need to clear the original executable file
make clean
# and to recompile
make
'''

Coming back to the non obsolete parts : 

There are different ways to for the SIM7600X module (qmi vs rndis).
Using the rndis mode (which I chose for this project) the module more behaves like an USB LTE modem and your connection to the mobile network is being established automatically.
The default qmi mode gives more manual control, but you also have to do more scripting yourself (e.g. getting an IP from mobile network using dhcp, setting operating mode of the module).

You can switch the operating mode of the module using minicom and directly sending commands to the module.

```
AT+CUSBPIDSWITCH=9011,1,1
```

A full list of possible commands for the module can be checked on their official documentation : https://www.waveshare.com/wiki/SIM7600E-H_4G_HAT
Depending on your usecase you might still need to disable network or modem manager in your operating system.


```
# Further installed libqmi utils and udhcpc :
# apt install libqmi-utils && udhcpc
```

Before you can really use the LTE modem, it might also be necessary to configure the APN settings to match your mobile network provider (for me it worked out of the box).
These modules can do a lot of stuff and we are really just barely scratching the surface with this project.
