# cups-openwrt
If you want to install cups on your openwrt/openwrt-LEDE router to share a usb printer or to enable airprint on your usb/network printer follow these steps. If you follow these step you will be able to print via an ios device, android, macos, linux and windows.

### Step 1
Find the "Platform" of your router on the openwrt. https://wiki.openwrt.org/toh/start

You will also need to know if your openwrt router run an LEDE version or not.

### Step 2
Follow those steps if your router is **not on a LEDE firmware** or go to step 3. https://wiki.openwrt.org/doc/howto/cups.server

If your router run an **LEDE** firmware follow these steps to compile the cups package. For these steps you must have a Linux computer. 

If you're lucky maybe the cups package is already build for your platform here: https://wiki.openwrt.org/doc/packages.old

Do the following steps on the computer to compile the packages.

*git clone https://github.com/lede-project/source*

*cd source*

*echo "src-git cups https://github.com/Gr4ffy/lede-cups.git" >> feeds.conf.default*

*cd scripts*

*./feeds update -a*

**make sure to have all the dependencies install to compile the package**

*./feeds install -a*

*make menuconfig* **(set Network->Printing->cups as "M" and set the target to your router "Platform")**

*make*

copy the following packages on the router with the scp command: 

you will find these packages there:


you will find this package here:


*scp Package root@192.168.0.1:/DESTINATION/PACKAGENAME*

**If your router's ip is not 192.168.0.1 change it!**
**if you got an usb device mount on your router, upload the package on the usb device**

### Step 3
install the packages with:

if your router doen't run an LEDE version just run: 
*opkg install cups*

If you got the compiled version of cups (LEDE) run this:
*opkg install* **package.ipk**

You can install multiple packages at the same time.

### Step 4
Configure cups.

*chmod 700 /usr/lib/cups/backend/usb*

Change the config of cups to be able to modify things over your network!

*vi /etc/cups/cupsd.conf*

### Step 5
Configure the printer in cups.

Go to http://192.168.0.1:631

Add your printer. If your printer is plug via USB you should see it on the screen. 
If your printer is a network printer you will have to configure it manualy. You will choose the prococol compatible with your printer. Personaly I have configure mine with the socket protocol.

**Note the printer name and description of the printer because you will need it for configuring airprint**

### Step 6
Configure airprint!

