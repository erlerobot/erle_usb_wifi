This tutorial explains how to use a USB WiFi dongle with chipset rtl8188CUS with Erle robot. The default drivers don't work so it's necessary to compile them manually.

##1) Build the RTL8192 Driver

* Download the Unix/Linux driver from the Realtek [website](http://www.realtek.com.tw/downloads/downloadsView.aspx?Langid=1&PNid=21&PFid=48&Level=5&Conn=4&DownTypeID=3&GetDown=false&Downloads=true) - It will be called something like RTL8192xC_USB_linux_v3.4.4_4749.20121105.zip
* Unzip the file and then unpack the driver source which is  driver/tl8188C_8192C_usb_linux_v3.4.4_4749.20121105.tar.gz in the zip file

* Download this patch to add BB config and to fix building on linux 3.8 into the driver directory and apply it

```
wget https://raw.github.com/cmicali/rtl8192cu_beaglebone/master/util/rtl-8192-beaglebone-linux-3.8.patch
patch -p1 < rtl-8192-beaglebone-linux-3.8.patch
```

-------

*The cross-compilation was done without the patch and the driver seems to work fine.*

-------

* Build the driver - Modify the `Makefile` according to your host system (i cross-compiled it in a Ubuntu 12.04, 64 bits machine). The configuration I used in the `Makefile`:

```
ifeq ($(CONFIG_PLATFORM_ERLE), y)
EXTRA_CFLAGS += -DCONFIG_LITTLE_ENDIAN
ARCH=arm
CROSS_COMPILE := arm-linux-gnueabihf-
#ROSS_COMPILE := 
KVER  := 3.8.13-bone35.2
KSRC ?= /home/victor/Desktop/linux-dev/KERNEL
MODDESTDIR := /lib/modules/$(KVER)/kernel/drivers/net/wireless/
INSTALL_PREFIX :=
endif
```

##2) Install the RTL8192 Driver

* Copy the driver (8192cu.ko) to the robot and then log into it: 

```
scp 8192cu.ko root@erlerobot:~/
ssh root@erlerobot
```

* Install the driver

```
mv 8192cu.ko /lib/modules/$(uname -r)
depmod -a
cd /etc/modules-load.d
echo "8192cu" >rtl8192cu-vendor.conf
```

-----

*If you previously installed the rtlwifi drivers, it'll be wise to blacklist them*

----

* Blacklist the old rtlwifi drivers 
```
cd /etc/modprobe.d
echo "install rtl8192cu /bin/false" >wifi_blacklist.conf
echo "install rtl8192c_common /bin/false" >>wifi_blacklist.conf
echo "install rtlwifi /bin/false" >>wifi_blacklist.conf
```

##3) Configure wpa_supplicant:

* Edit wpa_supplicant.conf:

```
ctrl_interface=/var/run/wpa_supplicant

# Unsecure Network
network={
    ssid="<ssid>"
    key_mgmt=NONE
    priority=<unsecure_priority>
}

# WEP Network
network={
    ssid="<ssid>"
    key_mgmt=NONE
    wep_key0="<key>"
    priority=<wep_priority>
}

# WPA Network
network={
    ssid="<ssid>"
    psk="<passphrase>"
    priority=<wpa_priority>
}
```

(tip: use wpa_passphrase)

* Use the `wifi-connect.sh` script.

----

###Sources:

* [http://embeddedprogrammer.blogspot.com.es/2013/01/beaglebone-using-usb-wifi-dongle-to.html](http://embeddedprogrammer.blogspot.com.es/2013/01/beaglebone-using-usb-wifi-dongle-to.html)
* [http://bonenotes.tumblr.com/](http://bonenotes.tumblr.com/)


[![Visit our IRC channel](https://kiwiirc.com/buttons/chat.freenode.net/erlerobot.png)](https://kiwiirc.com/client/chat.freenode.net/?nick=erlecoderf|?#erlerobot)
