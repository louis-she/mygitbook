## 树莓派使用 TP-Link WN725N USB 无线网卡

刚入树莓派，以为只要是无线USB网卡就都可以自动识别，大不了编一个驱动，结果就买了京东爆款`TP-Link WN725N`。

在SD卡上拷贝了操作系统，插上电源，接上网线，都一切正常，然后就在准备接无线网卡的时候，问题来了。

直接插上去肯定是无法识别，包装盒子里都带了一张windows驱动的安装光盘。既然连windows都要装驱动，linux更是不用说了，上了TPLINK官网准备搜linux驱动程序，结果被官方的问答机器人告知该型号不支持linux。这时已经以为几十块钱打了水漂。不过后来一番google拯救了我。

google一通发现，该网卡内部使用芯片实际上是有linux驱动的，所以就开始按照各种方法安装了，大概是下面这个步骤：

```shell
wget https://dl.dropboxusercontent.com/u/80256631/8188eu-20140307.tar.gz
tar -zxvf 8188eu-20140307.tar.gz
sudo cp rtl8188eufw.bin /lib/firmware/rtlwifi //not needed anymore if your operating system is above 3.10.33+
sudo install -p -m 644 8188eu.ko /lib/modules/3.10.33+/kernel/drivers/net/wireless
sudo insmod /lib/modules/3.10.33+/kernel/drivers/net/wireless/8188eu.ko
sudo depmod -a
sudo reboot
```

不过试了之后，依然不行，最后`lsusb`发现，网卡根本就没被识别嘛！

`lsusb`输出：
```shell
Bus 001 Device 002: ID 0424:9514 Standard Microsystems Corp.
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp.
```

随后仔细的google下来，发现so上的一篇文章，解决了问题：

> From the dmesg output, you see that the driver r8188eu recognizes the WiFi dongle, but cannot load firmware for it, since the firmware file cannot be found.

> The Raspberry Pi forum has two threads about getting these WiFi devices to work under Raspbian.

> Normally USB devices can be seen in the lsusb output even if a suitable driver is not available. I believe the dongle did not show up, because it is disconnected when the firmware is not found, as the last two dmesg lines show.

> I found that the regular Debian package firmware-realtek contains the required file. This would be a clean and convenient way to get it, but on my Raspbian I don't have this file, even though the package is installed.

大意就是“**固件都没有还玩个蛋**”，顺着大神给出的[链接](https://packages.debian.org/sid/firmware-realtek)去下固件，发现是一个，deb文件，满心欢喜，拷贝到pi上后：

```shell
dpkg -i xxxx.deb
```
就行了，然后修改 `/etc/network/interfaces`，加上

```shell
iface wlan0 inet static
wpa-ssid  ziroom    //无线网名称
wpa-psk  xxxxxxxxx  //无线网密码
address 192.168.1.148
netmask 255.255.255.0
gateway 192.168.1.1
network 192.168.1.1
```

再重启network `/etc/init.d/networking restart` 就行了，真是感谢这位大神，发现so的提问者也是对大神顶礼膜拜

> Yeah, i followed the link you posted above and it led me to the firmware download for the 8188eu chipset. As soon as I installed that, Raspbian identified the dongle succesfully. Many thanks for your time and help on this. – jonnyknowsbest Feb 9 at 16:02










