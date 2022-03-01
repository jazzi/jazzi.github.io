* Brand: Mercury
* Model: MW150UH
* Chipset: Realtek 8811CU

## Chipset misery

The most tough part of linux driver install actually is find out the exactaly **chipset** it has, as even the same USB Wifi dongle brand and same model, the chipset might be different, thus although you googled out an answer, it might not work. Except for google direct answer, the best option is find out the _Vendor ID_ and _Product ID_ by command `lsusb`.

```
$> lsusb
Bus 008 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 005 Device 004: ID 0bda:c811 Realtek Semiconductor Corp. RTL8812AU 802.11a/b/g/n/ac WLAN Adapter
```

In above listing, `0bda:8812` is the Vendor id and Product id, checking the flowing database to find out what each number represent.

* [Device Hunter](https://devicehunt.com/view/type/usb/vendor/0BDA)
* [The USB ID Repository](http://www.linux-usb.org/usb-ids.html)
* [USB ID Database](https://the-sz.com/products/usbid/index.php)

After searching you will find out code **0bda* represents Realtek while code **c811** represents the real chipset model **8811CU**, so the complete chipset name is Realtek 8811CU, Mercury just purchase this chipset from Realtek company.

## Install the driver

Go to [Github](https://github.com/morrownr/8821cu-20210118), download the ZIP file and unzip the file then add the excution permission to the install-driver.sh.

```
$> cd 8821cu-20210118-main
$> chmod +x install-driver.sh
$> sudo ./install-driver.sh
```

After successfully done above steps, your wifi module should be installed and go to open your WIFI and scan available signals.

A weird thing I encounted is it can not connect to the wifi even input the right password, later I dismissed this wifi and input the password again, finally it worked as charm.

## 0bda:1a2b or 0bda:c811

The more weird thing is the result of `lsusb` is different, it has two Product ID, the first time is **1a2b**, then sometimes it shows *c811*, so which one is right? I don't know, the best gamble is try both, install the driver respectively one by one and see which works.

Here are the driver for common chipsets you might need:

1. 8188gu (Produt id = 1a2b): https://github.com/McMCCRU/rtl8188gu
2. 8192eu (Product id = 818b): https://github.com/clnhub/rtl8192eu-linux 
