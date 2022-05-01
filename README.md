# wifiap

So why yet another tutorial? The internet is full of blogs that show how to do this.


* These instructions are the simplest way to setup a wifi-ethernet bridge access point for the following use case:
* Already have a wired network with DHCP, router, NAT, etc?
* Just need to add another wifi access point, maybe to add coverage further out?
* Client IPs will be assigned by your already existing infrastructure.
* We won't be adding another NAT (in the AP) to your network.
* No DNS caching, no extra DHCP server. Just a simple wifi to ethernet bridge.
* To-the-point instructions to get you operational faster. I'm not explaining how to install OS, how to edit files, etc - I assume you know all of this. Just focusing on the task at hand.

Is this what you need? Then continue reading.

* I will be using a RPI3b for this. You can use pretty much anything you want - the process will be similar. Low spec hardware may limit your speed due to the USB adapter or 100Mbps LAN, etc - but you already know what hardware you are using and it's specs/limitations.
* I am using a USB-wifi adapter with external antenna for better coverage. I won't use the built in RPI wifi.


Procedure
=========

* Install the OS of your choice on the device of your choice. For this project I downloaded the raspberry pi OS from their official site and burned it to SD card per their instructions, then I used the `raspi-config` tool to tidy it up and open ssh access.
* I'm using debian style `apt` commands, modify as required if you're on a different distro.
* Prepare your device and OS and configure it to a working state where the ethernet interface is functional.
* Install these two required tools:  `apt-get install hostapd bridge-utils`
* I also installed these optional helpful utils:  `apt-get install tmux mtr-tiny iptraf-ng ncdu dstat nethogs iftop htop pv pixz fping tmux vim`

* Stop the ethernet and wifi devices from getting an IP address. Best to do the next few steps in one go without rebooting so that we have working networking after we're done (networking will be broken if you don't complete these steps). We need to make these changes because we're going to create a bridge to connect eth and wifi - and the bridge itself gets an IP address. On raspberry pi OS you edit:  `vim /etc/dhcpcd.conf`  and add the following lines at the end of the file (figure out the actual eth and wlan devices you'll be using and use them):
```
# Ken changes below. v1.0.1 , 2022-05-01

denyinterfaces eth0
denyinterfaces wlan1
# ^ wlan1 is an external usb to wifi adapter - more power than the builtin rpi wifi antenna

interface br0
static ip_address=192.168.1.4/24
static routers=192.168.1.1
```

* `brctl addbr br0`
* `brctl addif br0 eth0`

* Edit:  `vim /etc/network/interfaces`  and add the following lines at the end of the file:
```
# Additions by Ken below. v1.0.1 , 2022-05-01

auto br0
iface br0 inet manual
bridge_ports eth0
# ^ hostapd adds the wlan interface to the bridge.
```




