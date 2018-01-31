# cups-openwrt
If you want to install cups on your openwrt/openwrt-LEDE router to share a usb printer or to enable airprint on your usb/network printer follow these steps.

### Step 1
Find the "Platform" of your router on the openwrt. https://wiki.openwrt.org/toh/start

### Step 2
Follow those steps if your router is **not on a LEDE firmware**. https://wiki.openwrt.org/doc/howto/cups.server

If your router run an **LEDE** firmware follow these steps to compile the cups package. For these steps you must have a Linux computer. Do the following steps on the computer.

git clone https://github.com/lede-project/source

cd source

echo "src-git cups https://github.com/Gr4ffy/lede-cups.git" >> feeds.conf.default

cd scripts

./feeds update -a

**make sure to have all the dependencies install to compile the package**

./feeds install -a

make menuconfig (set Network->Printing->cups as "M" and set the target to your router "Platform")

make

copy the following packages on the router with the scp command: 


scp Package root@192.168.0.1:/DESTINATION

**if you got an usb device mount on your router, upload the package on the usb device**

### Step 3
install the packages with:
opkg install **package.ipk**

You can install multiple packages at the same time.

