# wifiap

So why yet another tutorial? The internet is full of blogs that show how to do this.


These instructions are the simplest way to setup a wifi-ethernet bridge access point for the following use case:
* Already have a wired network with DHCP, router, NAT, etc?
* Just need to add another wifi access point, maybe to add coverage further out?
* Client IPs will be assigned by your already-existing infrastructure.
* We won't be adding another NAT (in the AP) to your network.
* No DNS caching, no extra DHCP server. Just a simple wifi to ethernet bridge.
* Focused instructions to get you operational faster. I'm not explaining how to install OS, how to edit files, etc - I assume you know all of this. Just focusing on the task at hand.

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

* Stop the ethernet and wifi devices from getting an IP address. Best to do the next few steps in one go without rebooting so that we have working networking after we're done (networking will be broken if you don't complete these steps). We need to make these changes because we're going to create a bridge to connect eth and wifi - and the bridge itself gets an IP address. On raspberry pi OS edit:  `vim /etc/dhcpcd.conf`  and add the following lines at the end of the file (figure out the actual eth and wlan devices you'll be using and use them):
```
# Ken changes below. v1.0.1 , 2022-05-01

denyinterfaces eth0
denyinterfaces wlan1
# ^ wlan1 is an external usb to wifi adapter - more power than the builtin rpi wifi antenna

interface br0
static ip_address=192.168.1.4/24
static routers=192.168.1.1
# ^ Modify these per your own network.
```

* Add a new bridge:  `brctl addbr br0`
* Add your eth interface to the bridge:  `brctl addif br0 eth0`

* Edit:  `vim /etc/network/interfaces`  and add the following lines at the end of the file (making changes to whatever you need):
```
# Additions by Ken below. v1.0.1 , 2022-05-01

auto br0
iface br0 inet manual
bridge_ports eth0
# ^ hostapd adds the wlan interface to the bridge.
```

* Configure the access point itself:  `vim /etc/hostapd/hostapd.conf` and insert this content:
```
# Config file v1.0.2 , 2022-05-01  by Kenneth Aaron

interface=wlan1
# ^ The name of the wifi interface we are configuring in this file

bridge=br0
# ^ The bridge interface behind the wifi interface.
#   hostapd will automatically add the wlan interface to this bridge.
#   Also look at how I setup:  /etc/network/interfaces

driver=nl80211
# ^ Use the nl80211 driver with the brcmfmac driver

ssid=NameOfAP
# ^ The name of this AP as seen by the clients

hw_mode=g
# ^ Use the 2.4Ghz band (g)  or the 5Ghz band (a)

channel=5
# ^ wifi channel number
#   Set this to 0 to let the AP search for the best channel.
#   Note - setting this to:  0  didnt work for me and I have to specify
#       a value manually.
#   Also note - some adapters like tplink dont support all the channels -
#       I couldn't configure 12 or 13.

max_num_sta=32
# ^ Maximum number of stations allowed to connect to this AP.

ieee80211d=0
# ^ Advertises country code + channels + power levels.
#   The default value is 0.

ieee80211n=1
# ^ Enable 802.11n

wmm_enabled=1
# Enable WMM
# QoS support, also required for full speed on 802.11n/ac/ax

macaddr_acl=0
# ^ Accept all MAC addresses

auth_algs=1
# ^ Use WPA authentication
#   1=wpa, 2=wep, 3=both

ignore_broadcast_ssid=0
# Dont require clients to know the network name

wpa=2
# ^ Use WPA2

wpa_key_mgmt=WPA-PSK
# ^ Use a pre-sharedkey

wpa_passphrase=YourPasswordGoesHere
# ^ The network password

rsn_pairwise=CCMP
# ^ Use AES instead of TKIP

logger_syslog=-1
logger_syslog_level=2
# ^ Increase logging level

```

* Tell hostapd to use our newly created file:  `vim /etc/default/hostapd`  and edit/create this config line:  `DAEMON_CONF="/etc/hostapd/hostapd.conf"`

* You may need to unmask the hostapd service to allow it to run:  `systemctl unmask hostapd`

* Reboot and test that it works.


Troubleshooting
===============

* If it doesn't load - try this:  `systemctl status hostapd.service`  and if you get:  `hostapd.service    Loaded: masked (Reason: Unit hostapd.service is masked.)    Active: inactive (dead)`  then do the unmask command above.


