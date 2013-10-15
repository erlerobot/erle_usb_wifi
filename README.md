This tutorial explains how to use a USB WiFi dongle with chipset rtl8188CUS with Erle robot. The default drivers don't work so it's necessary to compile them manually.

##1) Build the RTL8192 Driver

* Download the Unix/Linux driver from the Realtek [website](http://www.realtek.com.tw/downloads/downloadsView.aspx?Langid=1&PNid=21&PFid=48&Level=5&Conn=4&DownTypeID=3&GetDown=false&Downloads=true) - It will be called something like RTL8192xC_USB_linux_v3.4.4_4749.20121105.zip
* Unzip the file and then unpack the driver source which is  driver/tl8188C_8192C_usb_linux_v3.4.4_4749.20121105.tar.gz in the zip file

```
cp rtl8188C_8192C_usb_linux_v3.4.4_4749.20121105.tar.gz ~/
tar xvfz rtl8188C_8192C_usb_linux_v3.4.4_4749.20121105.tar.gz
cd rtl8188C_8192C_usb_linux_v3.4.4_4749.20121105
```

* Download this patch to add BB config and to fix building on linux 3.8 into the driver directory and apply it

```
wget https://raw.github.com/cmicali/rtl8192cu_beaglebone/master/util/rtl-8192-beaglebone-linux-3.8.patch
patch -p1 < rtl-8192-beaglebone-linux-3.8.patch
```

* Build the driver - Replace your_kernel_dir with the root of your BeagleBone 3.8 kernel source (mine is ~/projects/beaglebone_kernel/kernel) (If you wish to compile the kernel natively you'll have to modify the Makefile).

```
make KSRC=your_kernel_dir
```

##2) Install the RTL8192 Driver

* Copy the driver (8192cu.ko) to your beaglebone and then log into it: 

```
scp 8192cu.ko root@erlerobot:~/
ssh rooterlerobot@
```

* Install the driver

```
mv 8192cu.ko /lib/modules/$(uname -r)
depmod -a
(these steps are recommended but not necessary)
cd /etc/modules-load.d
echo "8192cu" >rtl8192cu-vendor.conf
```

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
