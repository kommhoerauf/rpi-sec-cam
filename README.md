# Raspberry Pi LTE security camera

Welcome to this small project

The whole motivation behind it was to get a small LTE security camera for the garden for which we do not need to pay a lot of money.
We do not have wifi in the allotment, so the choice was using the mobile network to alert us on any movements instead. 

What you will need : 
* A Raspberry Pi single board computer (or other comparable SBCs, for this project I am using a Raspberry Pi Zero W)
* A micro SD card (which we use to install Raspbian (Debian 11) 32 bit OS, I have gone with 32GB to have some space for media files)
* A micro USB cable to provide power to our SBC (I chose a 1A USB wall charger)
* A LTE modem compatiable with your SBC (for this project I have chosen the SIM7600X-4G-HAT (would also have been possible to the the variant for the Pi Zero))
* A USB cable to connect the LTE Modem to our SBC (Had to use this as I was not being able to get the LTE connection working without it, only got GSM working with IO pins)
* A SIM card to be used with the LTE modem (I have chosen to go the cheapest german mobile phone provider plan (~200 MB free per month but getting personalized advertisment))
* A Raspberry Pi camera (I used the Raspberry Pi Zero camera without IR filter)

Software we will make use of : 
* motioneye (used to operate our security camera) - https://github.com/motioneye-project/motioneye
* telegram (we will setup a telegram bot for our Raspberry Pi)
* telegram-send (we will configure motioneye to sent us text/video using telegram-send) - https://pypi.org/project/telegram-send/
* teleterm (to be able to send commands to our security camera once its running and not) - https://github.com/alfiankan/teleterm

Optional additions : 
* fail2ban
* ddns