# compile rtl8821au driver with support for 5ghz wifi

> this has been tested on linux mint, adapted from [the official mister wiki](https://github.com/MiSTer-devel/Main_MiSTer/wiki/MISTER-CUSTOM-WIFI-DRIVER-COMPILATION-GUIDE).

1. install arm toolchain

~~~bash
apt install gcc-arm-linux-gnueabihf build-essential
~~~

2. clone this and mister kernel repo and this repo in a work dir

~~~bash
mkdir tmpwrk && cd tmpwrk
git clone https://github.com/MiSTer-devel/Linux-Kernel_MiSTer
git clone https://github.com/valerino/8821au-20210708
rm -rf Linux_Kernel_MiSTer/.git
rm -rf 8821au-22010708/.git
~~~

3. compile mister kernel

~~~bash
cd Linux-Kernel_MiSTer
make ARCH=arm mrproper && make ARCH=arm MiSTer_defconfig && make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- EXTRAVERSION=-MiSTer modules_prepare
~~~

4. compile driver

~~~bash
cd .. && cd 8821au-20210708
MISTER=1 make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- KSRC=../Linux-Kernel_MiSTer modules
~~~

if everything went well, you should find the compiled driver in your current directory.

~~~bash
➜ ~/Downloads/tmpdriver/8821au-20210708 (main) ✗ ls -l 
total 4648
-rw-rw-r-- 1 valerino valerino 2149464 ott 27 14:24 8821au.ko
~~~

5. now transfer this to MiSTer in /media/fat/8821au.ko and **unplug the wifi dongle and reboot MiSTer**.

6. **from now on, you proceed with a keyboard, logging into MiSTer as root with F9**.

7. copy file to /lib/modules/<kernel-version>, i.e. /lib/modules/5.14.5-MiSTer

~~~bash
cp /media/fat/8821au.ko /lib/modules/5.14.5-MiSTer/8821au.ko
~~~

7. install module

~~~bash
depmod -a
modprobe 8821au
~~~

you should get no error.

8. copy over the default module, making sure to delete the old.

~~~bash
rm /lib/modules/5.14.5-MiSTer/kernel/drivers/net/wireless/realtek/rtl8821au/*
mv /lib/modules/5.14.5-MiSTer/8821au.ko /lib/modules/5.14.5-MiSTer/kernel/drivers/net/wireless/realtek/rtl8821au/8821au.ko
~~~

9. assuming you have a working /media/linux/wpa_supplicant.conf, reboot replug the wifi dongle and you should be done. that's it.