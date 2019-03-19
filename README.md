### Parts
* Apple IR Reciever (_APPLE PART NUMBER: 820-2540-A_)
* USB-A Header
* Raspberry Pi


### Wiring the APPLE IR reciever to usb

As you can see on the wiring diagram. The pin next to the "V" marked on the board is D- and then it is D+, 5V, GND:

![alt text](https://raw.githubusercontent.com/MadsLudvig/appleirreciever/master/Apple%20IR%20to%20usb%20diagram.png?token=AhbQcaTgsNl_bHviLEdJ7d5b_S_f6k18ks5cmfjdwA%3D%3D)

### Using the correct driver

To make usbhid driver use the Apple IR receiver as dev/usb/hiddev instead of dev/input/hidraw,

The quirk we need to add to usbhid, contains following:
```
0x05ac:0x8242:0x40000010
```
* With 0x05ac being the vendor ID (Apple, you shouldn't need to change this)
* With 0x8242 being the product ID (check the output of lsusb for your hardware)
* And 0x10 being "HID_QUIRK_HIDDEV_FORCE" and 0x40000000 being "HID_QUIRK_NO_IGNORE"

add the following line:
```
usbhid.quirks=0x05ac:0x8242:0x40000010
```
to this file:
```
$ sudo nano /boot/cmdline.txt
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
