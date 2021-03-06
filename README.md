# Raspberry pi with IR transmitter and Apple IR reciever

### Foreword

This project started with me thinking of how i could make my apple silver remote work with my B&O sound system.

So i decided to use a raspberry pi with an IR reciever and transmitter, to recieve the signals from my apple remote and send corresponding signals to my B&O sound system.

At the time i only had an old tv remote and an old imac ir reciever, so i decided to use those parts. 

The main reason i'm making this guide, is to help those wanting to connect their apple ir reciever to their linux machine.

### Parts
* Apple IR Reciever (_APPLE PART NUMBER: 820-2540-A_)
* USB-A Header
* Raspberry Pi


### Wiring the APPLE IR reciever to usb

As you can see on the wiring diagram. On the back of the board where the connector is located, the pin next to the "V" marked on the board is D- and then it is D+, 5V, GND:

![IR-USB WIRING](https://raw.githubusercontent.com/MadsLudvig/appleirreciever/master/Apple%20IR%20to%20USB%20diagram.png?token=AhbQcdgM5Jng2qnsoFgfHB31-FPMCoi9ks5cm1xUwA%3D%3D)

### Using the correct driver

To make usbhid driver use the Apple IR receiver as dev/usb/hiddev instead of dev/input/hidraw,

The quirk we need to add to usbhid, does the following:
```
0x05ac:0x8242:0x40000010
```
* 0x05ac is the vendor ID (In this case it is Apple)
* 0x8242 is the product ID (found by running lsusb command)
* 0x10 is "HID_QUIRK_HIDDEV_FORCE" and 0x40000000 is "HID_QUIRK_NO_IGNORE"

add the following line to `/boot/cmdline.txt`
```
usbhid.quirks=0x05ac:0x8242:0x40000010
```
Then reboot
```
$ sudo shutdown -r now
```
to check, that it is registred as hiddev, run the following commands, with only the IR reciever plugged into the usb ports
```
$ lsusb

$ ls -l /dev/usb/
```
output should be something like:
```
crw-rw---- 1 root root 180, 96 2008-07-13 07:48 hiddev0
```

### Installing LIRC

To install use this command:
```
$ sudo apt-get install lirc
```
### Configuring LIRC

We will need to configure two files:

1. lirc_options.conf (contains device, driver, module options etc.)
2. lircd.conf (the main lircd configuration file, contains remote definition in terms of key scan codes)

**Configuring lirc_options.conf**

We will need to change the values in lirc_options.conf:
```
driver          = macmini
device          = /dev/usb/hiddev0
```
bear in mind, that the device location, is what you found earlier with the ls -l /dev/usb/ command

**Configuring lircd.conf**

When configuring the lircd.conf file you can either download config files for a specific remote, or make your own.
We are going to make our own

To make your own config files, you use the irrecord command, like so
```
$ sudo irrecord -H macmini -d /dev/usb/hiddev0 /etc/lirc/lircd.conf
```
After configuring the file, we will need to restart LIRC, like this:
```
$ sudo service lirc restart
```
Then we can test if our config is working by pressing buttons on the remote, while running:
```
$ irw
```

### Executing specific command when key is pressed
With this project, we want to execute a specific command, when a specific keycode is pressed. This can be done with irexec

First we will need to config the irexec.lircrc

Below is an example of one of the sections in the irexec.lircrc file
```
begin
    prog   = irexec
    button = KEY_1
    config = echo "KEY_1"
end
```
We just need to change config to a command we would like to execute everytime, the specific key is recieved
```
config = [command of your choice]
```

Sources:
* [IREXEC](http://www.lirc.org/html/irexec.html)
* [IRSEND](http://www.lirc.org/html/irsend.html)
* [Infrared in Linux: LIRC](https://idebian.wordpress.com/2008/07/15/infrared-in-linux-lirc/)
* [IR HTPC Macmini](https://forum.kodi.tv/showthread.php?tid=260292)
* [Getting Apple Remote, Macbook and LIRC work together](https://cweiske.de/tagebuch/Getting%20Apple%20Remote,%20Macbook%20and%20LIRC%20work%20together.htm)
* [Disabling APPLEIR kernel module](https://lwn.net/Articles/407938/)
