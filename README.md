# Raspberry Pi LTE security camera

Welcome to this small project.

The whole motivation behind it was to get a small LTE security camera for the garden for which we do not need to pay a lot of money.
We do not have wifi in the allotment, so the choice was using the mobile network to alert us on any movements instead. 

![Finished Raspberry Pi LTE securiy camera in a wood box](pictures/finished.png?raw=true "RPI LTE security camera")

What you will need : 
* A Raspberry Pi single board computer (or other comparable SBCs, for this project I am using a Raspberry Pi Zero W) ~ 20€ (if you get a good deal nowadays)
* A micro SD card (which we use to install Raspbian (Debian 11) 32 bit OS, I have gone with 32GB to have some space for media files) ~ 10€
* A micro USB cable to provide power to our SBC (I chose a 1A USB wall charger) ~ 10€
* A LTE modem compatible with your SBC (for this project I have chosen the SIM7600X-4G-HAT (would also have been possible to choose the specific variant for the Pi Zero (SIM7600G-H))) ~ 90€
* A USB cable to connect the LTE Modem to our SBC (Had to use this as I was not being able to get the LTE connection working without it, only got GSM working with IO pins) ~ 0€, its included for the SIM7600X, but depending on your container for the camera you might need angled connectors (as I did)
* A SIM card to be used with the LTE modem (I have chosen to go the cheapest german mobile phone provider plan (~200 MB free per month but getting ads)) ~ 0€ (depending on your mobile network provider you have monthly costs)
* A Raspberry Pi camera (I used the Raspberry Pi Zero camera without IR filter) ~ 10€
* A container for everything (I chose a small and tightly fitting wood box for this) ~ 5€

Software we will make use of : 
* motioneye (used to operate our security camera) - https://github.com/motioneye-project/motioneye
* telegram (we will setup a telegram bot for our Raspberry Pi)
* telegram-send (we will configure motioneye to sent us text/video using telegram-send) - https://pypi.org/project/telegram-send/
* teleterm (to be able to send commands to our security camera once its running and connected to the mobile network, a reverse SSH tunnel might also work but I wanted to save as much mobile traffic as possible) - https://github.com/alfiankan/teleterm

Optional additions : 
* fail2ban (might be obsolete as no ports are directly reachable through mobile network)
* dyndns (might be obsolete, it works to store the mobile IP but as no ports are open you gain nothing)
* ufw (might be obsolete, ports are not reachable from the mobile network)

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

Lets also disable daily updates to be sure not to use too much mobile data once deployed : 

```
sudo systemctl stop apt-daily.timer
sudo systemctl disable apt-daily.timer
sudo systemctl mask apt-daily.timer
sudo systemctl stop apt-daily-upgrade.timer
sudo systemctl disable apt-daily-upgrade.timer
sudo systemctl mask apt-daily-upgrade.timer
```

#### Setup LTE modem

Second step is to configure our LTE modem :

Assuming you have already prepared the following : 
* SIM card with PIN disabled
* Connected the SIM7600X-4G-HAT using the IO pins
* Set the correct jumper positions (to configure the module for Raspberry Pi and boot it up at the same time)
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
``` 

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

In rndis mode I can see the following : 

```
root@picam:/var/log# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: usb0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 1000
    link/ether 86:ba:c7:f9:d2:b5 brd ff:ff:ff:ff:ff:ff
    inet 192.168.225.24/24 brd 192.168.225.255 scope global dynamic noprefixroute usb0
       valid_lft 39064sec preferred_lft 33664sec
    inet6 2a02:3032:208:e82:ade7:b6a5:fa4a:9de2/64 scope global mngtmpaddr noprefixroute
       valid_lft forever preferred_lft forever
    inet6 fe80::e664:5ce:3cba:7c8a/64 scope link
       valid_lft forever preferred_lft forever
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether b8:27:eb:f9:c7:0f brd ff:ff:ff:ff:ff:ff
    inet 192.168.178.75/24 brd 192.168.178.255 scope global dynamic noprefixroute wlan0
       valid_lft 861261sec preferred_lft 753261sec
    inet6 fe80::9545:b9b3:e84c:4b28/64 scope link
       valid_lft forever preferred_lft forever

root@picam:/var/log# ip ro
default via 192.168.225.1 dev usb0 proto dhcp src 192.168.225.24 metric 202 mtu 1500
default via 192.168.178.1 dev wlan0 proto dhcp src 192.168.178.75 metric 303
192.168.178.0/24 dev wlan0 proto dhcp scope link src 192.168.178.75 metric 303
192.168.225.0/24 dev usb0 proto dhcp scope link src 192.168.225.24 metric 202 mtu 1500

root@picam:/var/log# ping -I usb0 8.8.8.8
PING 8.8.8.8 (8.8.8.8) from 192.168.225.24 usb0: 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=51.8 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=40.1 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=113 time=38.4 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=113 time=36.4 ms
^C
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 36.382/41.660/51.776/5.988 ms
```

#### Motioneye

The software heart of this project. There are preinstalled images with this software but we will set it up manually.

We are following the wiki installation instructions : https://github.com/motioneye-project/motioneye/wiki for a Debian 11 installation (as we chose a Raspbian 11 (bullseye) OS in the beginning).
Its straight forward, just needs python2 pip but works like a charm afterwards.

The camera integration is also easily done, the configuration options of the software are huge!

![motioneye screen with configured camera](pictures/motioneye.png?raw=true "motioneye UI in webbrowser with configured camera")

I have saved myself a few variants of the camera configuration from /etc/motioneye directory (versions with and without motion detection and working schedule) so I can turn things on or off from the distance without needing an UI.
Depending on what SBC you use you might want to lower your camera resolution, for the Raspberry Pi Zero I have gone with the lowest (320x200 & 2 fps).

#### Telegram integration

* Setting up telegram bot
TBD

* Installation and configuration of telegram-send
Its very well explained on their package page : https://pypi.org/project/telegram-send/
All we will need is the bot token and a device to talk to the bot (e.g. your Smartphone with Telegram App installed)
Motioneye will run beneath the root user, so be sure to be root when configuring telegram-send

```
# Depending if you want to send messages to your personal Telegram or a group :
# You might also have configured a different user
sudo -H -u motion bash -c "telegram-send --configure"
sudo -H -u motion bash -c "telegram-send --configure-group"
```

If you by accident already have teleterm running you might need to stop it for the duration until you have configured telegram-send.

* Installation and configuration of teleterm
Installation instructions are clear on : https://github.com/alfiankan/teleterm
Be sure to have go version (at least) 1.18 installed and compile the code directly on your Raspberry Pi (took a few minutes for the Pi Zero ;-))
We are kind of installing a backdoor here, be aware there might be better solutions, its just a personal preference which came in quite handy.


We further need to whitelist your Telegram contact (you will need your telegram user id here) to be allowed to send commands. You do not want anybody to have root access to your remote Pi

```
ubuntu@picam:~ $ cat ~/.teleterm/config.yaml
teleterm:
  telegram_token: "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  shell_executor: "/bin/bash"
whitelist:
  - 123456789
```

#### Performance

In the end the CPU/Memory usage is ok when idling, but if motioneye captures videos/pictures the Raspberry Pi noticeable slows down.

![htop terminal output of Raspberry Pi Zero when idling](pictures/htop.png?raw=true "htop output when idling")

When there is movement in front of the camera cpu usage spikes up to 80% for me, afterwards if a video is being recorded + saved we already hit 100% for a short duration.