# wifiap

So why yet another tutorial? The internet is full of blogs that show how to do this.


* These instructions are the simplest way to setup a wifi-ethernet bridge access point for one particular use case.
* Already have a wired network with DHCP, router, NAT, etc?
* Just need to add another wifi access point, maybe to add coverage further out?
* Client IPs will be assigned by your already existing infrastructure.
* We won't be adding another NAT (in the AP) to your network.
* No DNS caching, no extra DHCP server. Just a simple wifi to ethernet bridge.

Is this what you need? Then continue reading.

* I will be using a RPI3b for this. You can use pretty much anything you want - the process will be similar.
* I am using a USB-wifi adapter with external antenna for better coverage. I won't use the built in RPI wifi.




