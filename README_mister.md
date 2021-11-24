# compile rtl8821au driver with support for 5ghz wifi

> this has been tested on linux mint, adapted from [the official mister wiki](https://github.com/MiSTer-devel/Main_MiSTer/wiki/MISTER-CUSTOM-WIFI-DRIVER-COMPILATION-GUIDE).

0. plug your wifi dongle into MiSTer !

2. install arm toolchain on the host.

~~~bash
apt install gcc-arm-linux-gnueabihf build-essential
~~~

2. on the host, clone MiSTer kernel repo and driver repo in a work dir

~~~bash
mkdir tmpwrk && cd tmpwrk
git clone https://github.com/MiSTer-devel/Linux-Kernel_MiSTer
git clone https://github.com/valerino/8821au-20210708
rm -rf Linux_Kernel_MiSTer/.git
rm -rf 8821au-22010708/.git
~~~

3. on the host, compile mister kernel

~~~bash
cd Linux-Kernel_MiSTer
make ARCH=arm mrproper && make ARCH=arm MiSTer_defconfig && make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- EXTRAVERSION=-MiSTer modules_prepare
~~~

4. on the host, compile driver

~~~bash
cd .. && cd 8821au-20210708
MISTER=1 make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- KSRC=../Linux-Kernel_MiSTer modules
~~~

if everything went well, you should find the compiled driver in your current directory on the host.

~~~bash
➜ ~/Downloads/tmpdriver/8821au-20210708 (main) ✗ ls -l 
total 4648
-rw-rw-r-- 1 valerino valerino 2149464 ott 27 14:24 8821au.ko
~~~

5. now transfer the compiled driver to MiSTer.

~~~bash
scp ./8821au.ko root@mister.ip:/media/fat
~~~

7. ssh into MiSTer, and copy driver to /lib/modules/<kernel-version>, i.e. /lib/modules/5.15.1-MiSTer

~~~bash
cp /media/fat/8821au.ko /lib/modules/5.15.1-MiSTer/8821au.ko
~~~

7. on MiSTer, install module.

~~~bash
depmod -a
modprobe 8821au

# the following error is normal!
/lib/modules/5.15.1-MiSTer# modprobe 8821au
modprobe: ERROR: could not insert '8821au': Device or resource busy
~~~

8. on MiSTer, copy over the default module, making sure to delete the old.

~~~bash
rm /lib/modules/5.15.1-MiSTer/kernel/drivers/net/wireless/realtek/rtl8821au/*
mv /lib/modules/5.15.1-MiSTer/8821au.ko /lib/modules/5.15.1-MiSTer/kernel/drivers/net/wireless/realtek/rtl8821au/8821au.ko
~~~

9. assuming you have a working /media/linux/wpa_supplicant.conf on MiSTer, reboot and you should be done. that's it :)
  
~~~bash
# example wpa_supplicant.conf
ctrl_interface=/run/wpa_supplicant
update_config=1
country=IT

network={
	ssid="my_SSID"
	psk="my_pwd"
}

network={
	ssid="another_SSID"
	psk="another_pwd"
}
~~~
