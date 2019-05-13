Disclaimer: I got this device from the Manufacturer to test Linux on it. The below findings are my own and express my own opinion.

Specs and short review:
=======================

* Intel Atom x5-Z8350 CPU (4x1,92 GHz)
* Intel Integrated GPU
* 1920x1200 10.1" IPS Screen
* 4GB RAM
* 64GB Internal Storage
* 1 x USB C
* 1 x USB Micro B
* 1 x Mini HDMI
* Bluetooth 4.2
* 2.4GHz WiFi
* Rotation Sensor
* Front and Back Camera
* Mico SD Card slot up to 400GB (Website claims 64GB but I sucessfully tested it with an 128GB Card that uses the SDXC Standard)
* Headphone Jack
* Stereo Speakers
* Keyboard Dock Support
* Touchpen support

The Chuwi Hi10 Air is a compact 10.1" Tablet running a Windows 10. It has a sturdy Aluminium Body and the overall build quality is astonishing.

The speakers are surpisingly good for a tablet in that price segment. Paired with Bluetooth, a headphone jack and the 1920x1200 WUXGA display it's perfect for all kind of Media Playback. The battery is powerful enough to easily survive a few Movies. The optional detachable keyboard is a welcome addition for when you need to write some text.

The keyboard is nice to type on, but I had cases where it would send keys out of nowhere. I can't guarantee that's not caused by the few drops of water I accidentally spilled on the connection to the tablet at some point. I'm not a big fan of the trackpad, mainly because it's way to small to be usable for me but that might be because I am used to the trackpad on my 15" MacBook Pro.

The Touchpen is very accurate thanks to the digitizer, however the touchscreen does not send different signals for Pen and Finger to the System. That means trying to take notes while resting your hand on the screen doesn't really work.

Sadly, the full Windows 10 install user experience is really impacted by the entry level Intel Atom Z8350. It's just not powerful enough to accomplish any advanced tasks.

However, Chuwi sent me the device to bring linux to it, so that's what I did. This allows us to use lighter Window Managers and therefore more resources for actual tasks, resulting in a better performance.

What follows is a guide on how to setup on how to install Arch Linux on your Chuwi Hi10 Air.

Install Linux
=============

First of all you need to install Linux on the tablet. That means you will erase your internal Windows installation, so make sure to either do a backup of it, or at least make yourself familiar with how to reinstall Windows in case it's needed.

Please note that you can NOT install Linux on an SDCard because the BIOS can not boot from the SDCard.

Backup
------

You can find the Hi10 Air Windows drivers here:
https://forum.chuwi.com/forum.php?mod=viewthread&tid=6861
And a guide here (applies for the Hi10 Air too):
https://forum.chuwi.com/forum.php?mod=viewthread&tid=15&page=7#pid29091
https://forum.chuwi.com/thread-15-1-1.html

Install
-------

Next up, it's time to choose your Linux distro.
I recommend anything that is based on Arch Linux and that uses either LightDM or lxdm, because that is what my guide will be using to setup certain features. Of cause you're free to choosewhatever you want.

That means you could for example use Antergos, an ArchLinux based distro that includes Desktop managers (for Antergos I recommend XFCE), or go the hard way and install ArchLinux from scratch and use lxdm with LXQt. LXQt proved to me to be the best lightweight and yet High-DPI friendly Desktop Environment, so that's what I used.

I trust you will be able to find the guides on how to install either with a quick Google search, so I will not go much further into detail here.
To get into the boot menu, you need to press F7.

After your installation is done most things are already working.

Fixing Remaining Issues
=======================

Out of the box, there are a few issues and things that do not work because they are missing drivers or similar.

Automatic Rotation
------------------

We're getting started with the most important which is rotation.

As you'll need to enter a couple commands to get automatic rotation working, start off with manually rotating the screen by opening a terminal and run the command ```xrandr -o left``` Note: xrandr is part of xorg-xrandr, and you will need that package for autoration to work.

There are a couple solution on how to rotate the screen out there, however I decided to go for the one with least overhead that looked the cleanest to me.

First of all, compile the 2in1screen binary. The sourcecode can be found in this Repository. Download, compile it and push it to /usr/local/bin.

Note: Make sure to install xorg-xrandr and xorg-xinput for this tool to work.

```
wget https://raw.githubusercontent.com/Myself5/Chuwi_Hi10_Air_Linux/master/2in1screen.c
gcc -O2 -o 2in1screen 2in1screen.c
sudo mv 2in1screen /usr/local/bin/
sudo chmod +x /usr/local/bin/2in1screen
```

Now configure the Desktop Manager to start it after login.

LXDE: add ```/usr/local/bin/2in1screen &``` to ```/etc/lxdm/PostLogin```

LightDM: Place the screenrotate.sh in /etc/lightdm/screenrotate.sh and set ```display-setup-script``` in ```/etc/lightdm/lightdm.conf``` to ```display-setup-script /etc/lightdm/screenrotate.sh```

screenrotate.sh
```
#!/bin/bash
pkill -9 2in1screen
/usr/bin/2in1screen &
```

Reboot and make sure everything works as desired.

Touchscreen
-----------

The Hi10 Airs Silead touchscreen requires drivers that are not bundled with a Linux install. You can download them here: https://github.com/onitake/gsl-firmware/tree/master/firmware/linux
See the Readme on how to install them.

These drivers get loaded by the Linux kernel. I submitted a commit to add support for the Hi10 Air to the Linux Kernel and it has been approved. The commit is merged in the 5.1 (and newer) Kernel. That means you need to update your kernel for a proper touch support.

If you previously used my Kernel you can go back to the official kernel by running:
```
sudo pacman -R linux-chewbacca linux-chewbacca-headers
sudo pacman -Syu linux linux-headers
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

Now reboot and enjoy your touchscreen.

Audio
-----

If your Audio isn't working, make sure to update the alsa-lib to Version 1.1.7 or above. On Arch that package is in the stable repos so a ```sudo pacman -Syu``` will be enough.

Bluetooth
---------

Install blueman and follow the firmware instructions mentioned here:
https://github.com/lwfinger/rtl8723bs_bt/issues/28#issuecomment-432806835

Which means you need to clone https://github.com/lwfinger/rtl8723bs_bt
and run
```
sudo cp rtlbt_fw /usr/lib/firmware/rtl_bt/rtl8723bs_fw.bin
sudo cp rtlbt_config /usr/lib/firmware/rtl_bt/rtl8723bs_config.bin
cd /usr/lib/firmware/rtl_bt
sudo ln -s rtl8723bs_config.bin rtl8723bs_config-OBDA8723.bin
```

General UI
----------

Now, you probably already noticed, everything is a bit small to use your Fingers.

As a browser I recommend Google Chrome as that has a great touchscreen UI already.

For File browsing and reading I suggest using nautilus and evince because of their touch friendlyness.

### System DPI ###

For the whole system, there is a few things you can improve:
First of all, increase the general DPI.
You can do so by adding ```Xft.dpi: 150``` to ```~/.Xresources```. If the file doesn't exist, create it.

### LXQt Tweaks ###

Next up a few additional tweaks for LXQt:
First of all increase the Panel bar
Rightclick on the Panel and Press "Configure Panel". Then set the Size to 50px and the Icon Size to 30px.

Next up, Go to the Menu -> Preferences -> LXQt settings -> Appearance -> Font and set the DPI to 96
and finally go to Menu -> Preferences -> LXQt settings -> OpenBox Settings -> Font and increase the individual fonts to get a window title bar to drag/drop as well as hit the navigation with the finger. I set mine to Cantarell 16 and Cantarell 14.

### Install OnBoard ###

Another Handy feature I found myself in need of was an onscreen keyboard. For that I installed OnBoard, and found a very neat feature here: https://bugs.launchpad.net/onboard/+bug/1232107

You can map a button to open/close the keyboard.
I set mine to Super L, which equals the left Windows button and the touch button on the tablet itself.

In the LXQt Settings Menu you can configure Shortcut Keys.
Create or edit the existing shortcut and either map the following DBus Call.

DBus Call:
Service:
```org.onboard.Onboard```
Path:
```/org/onboard/Onboard/Keyboard```
Interface:
```org.onboard.Onboard.Keyboard```
Method:
```ToggleVisible```

If your Desktop environment doesn't support DBus calls, use the following command:
```
dbus-send --type=method_call --dest=org.onboard.Onboard /org/onboard/Onboard/Keyboard org.onboard.Onboard.Keyboard.ToggleVisible
```
