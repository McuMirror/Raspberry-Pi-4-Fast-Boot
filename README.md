# Introduction
This tutorial will show you how to make your raspberry pi 4 with **Raspberry PI Os Lite** boot faster

# Check initial boot time
In order to know that our changes actually do something, we will check the boot time before we change anything
In Raspberry Pi Os you can check the boot time with this command:

```
sudo systemd-analyze time
Startup finished in 2.241s (kernel) + 17.891s (userspace) = 20.132s
graphical.target reached after 17.853s in userspace
```
In my case stock Raspberry Pi Os Lite took **~20 seconds** to boot\
Thats way to long for my use case\
Let's fix it

# Disable waiting for network
The first tweak is to disable waiting for network on boot. This can be done through raspi-config.\
`sudo raspi-config`\
This will bring up a. Follow this path to disable:\
System Options -> Network at Boot -> No -> Ok

# Disable Services 
The next tweak is disabling not needed services\
Raspberry PI Os comes with a handy command which prints a list of all services sorted by start time
```
sudo systemd-analyze blame
12.004s dhcpcd.service
7.085s hciuart.service
3.680s console-setup.service
2.767s dev-mmcblk0p2.device
1.655s raspi-config.service
923ms rpi-eeprom-update.service
566ms systemd-udev-trigger.service
530ms dphys-swapfile.service
519ms keyboard-setup.service
515ms avahi-daemon.service
481ms systemd-logind.service
447ms networking.service
399ms systemd-fsck@dev-disk-by\x2dpartuuid-edb23c1b\x2d01.service
359ms systemd-timesyncd.service
317ms rng-tools.service
...
```
**Warning:** you cant disable all services because some of them are actually needed\
Here is a list of services which are safe to disable below:
```
sudo systemctl disable ntp.service
sudo systemctl disable rsyslog
sudo systemctl disable dphys-swapfile.service
sudo systemctl disable keyboard-setup.service
sudo systemctl disable systemd-journald
sudo systemctl disable systemd-timesyncd
sudo systemctl disable systemd-udev-trigger
sudo systemctl disable apt-daily.service
sudo systemctl disable wpa_supplicant
sudo systemctl disable wifi-country.service
sudo systemctl disable hciuart.service
sudo systemctl disable raspi-config.service
sudo systemctl disable avahi-daemon.service
sudo systemctl disable triggerhappy.service
```
If you need bluetooth dont disable hciuart.service

# Config file
After that we have the config file\
Here we will overclock the pi

```
sudo nano /boot/config.txt
```

And add the following

```
# overclock
over_voltage=2
arm_freq=1850
initial_turbo=40
```
My full config.txt looks like this (i deleted all the unnecessary comments etc.) 
```
# display settings
hdmi_group=2
hdmi_mode=85
hdmi_force_hotplug=1

# general
dtoverlay=vc4-fkms-v3d,disable-bt
max_framebuffers=2
dtparam=audio=on

# overclock
over_voltage=2
arm_freq=1850
initial_turbo=40
```
# Cmdline File
The last fix is the cmdline.txt file. Here we will disable the boot messages (yes, this takes time)\
```
sudo nano /boot/cmdline.txt
```
And add "**quiet**" before rootwait
Then your file should look like this\
**Warning**: Do NOT copy&paste this because PARTUUID is always different 
```
console=serial0,115200 console=tty1 root=PARTUUID=61dc8113-02 rootfstype=ext4 elevator=deadline fsck.repair=yes quiet rootwait
```
# Result
Let's reboot the pi and check how much we could reduce the boot time
```
sudo systemd-analyze time
Startup finished in 1.127s (kernel) + 2.479s (userspace) = 3.606s 
graphical.target reached after 2.450s in userspace
```
Amazing result. We got it down to just 3.6 seconds

