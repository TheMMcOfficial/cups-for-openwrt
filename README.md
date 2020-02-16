# CUPS for Openwrt/LEDE
If you want to install cups on your openwrt/openwrt-LEDE router to share a usb printer over network or to enable airprint on your usb/network printer follow these steps. If you follow these step you will be able to print via an iOS device, Android, macOS, Linux and Windows.

### Step 1
Find the "Platform" of your router on the openwrt. https://wiki.openwrt.org/toh/start

You will also need to know if your openwrt router run an LEDE version or not.

### Step 2
Follow those steps if your router is **not on a LEDE firmware (Openwrt 15 or less)** or go to step 3. https://wiki.openwrt.org/doc/howto/cups.server

If your router run an **LEDE** firmware follow these steps to compile the cups package. For these steps you must have a Linux computer. 

If you're lucky maybe the cups package is already build for your platform here: https://wiki.openwrt.org/doc/packages.old

Do the following steps on the computer to compile the packages.

```
git clone https://github.com/lede-project/source

cd source

echo "src-git cups https://github.com/TheMMcOfficial/lede-cups.git" >> feeds.conf.default
```

**make sure to have all the dependencies install to compile the package**
```
./scripts/feeds update -a

./scripts/feeds install -a

make menuconfig
```

**(set the target system to your router "Platform" and set Network->Printing->cups as "M")**

```
make
```

**If you try to compile with the root user you will get errors. To be able to compile you have to type this command:**
```
export FORCE_UNSAFE_CONFIGURE=1
```
copy the following packages on the router with the scp command: 

you will find these packages there:

/source/bin/packages/PLATFORM/cups/

cups_VERSION_PLATFORM.ipk 

libcupscgi_VERSION-PLATFORM.ipk

libcupsmime_VERSION_PLATFORM.ipk

libcupsppdc_VERSION_PLATFORM.ipk


you will find this package here:

/source/bin/packages/PLATFORM/packages

libpng_VERSION_PLATFORM.ipk

libcups_VERSION_PLATFORM.ipk 

you can also update the libjpg if you want

/source/bin/packages/PLATFORM/base

libusb-1.0_VERSION_PLATFORM.ipk

zlib_VERSION_PLATFORM.ipk

```scp Package root@192.168.0.1:/DESTINATION/PACKAGENAME```

**If your router's ip is not 192.168.0.1 change it!**
**if you got an usb device mount on your router, upload the package on the usb device**

### Step 3
install the packages with:

if your router doen't run an LEDE version just run: 

```opkg install cups```


If you got the compiled version of cups (LEDE) run this:

```opkg install **package.ipk**```

You can install multiple packages at the same time.

You will maybe have to install avahi-demon too.

opkg install avahi-daemon

### Step 4
Configure cups.

```chmod 700 /usr/lib/cups/backend/usb```

Change the config of cups to be able to modify the config of cups over your network!

```vi /etc/cups/cupsd.conf```

```
#######################################################################
#                                                                      #
# This is the CUPS configuration file.  If you are familiar with       #
# Apache or any of the other popular web servers, we've followed the   #
# same format.  Any configuration variable used here has the same      #
# semantics as the corresponding variable in Apache.  If we need       #
# different functionality then a different name is used to avoid       #
# confusion...                                                         #
#                                                                      #
########################################################################


AccessLog syslog
ErrorLog syslog
LogLevel info
PageLog syslog
PreserveJobHistory No
PreserveJobFiles No
AutoPurgeJobs Yes
MaxJobs 25
MaxPrinterHistory 10
#Printcap /etc/printcap
#PrintcapFormat BSD
RequestRoot /var/cups
#RemoteRoot remroot
#User nobody
#Group nobody
# root permissions required to make cups work with the usb backend
User root
Group root
RIPCache auto
TempDir /var/cups
Port 631
HostNameLookups On
KeepAlive On
# No: "BrowseOrder" "BrowseAllow" "BrowseRemoteProtocols"
Browsing Yes
BrowsingWebIF Yes
BrowseLocalProtocols DNSSD
DefaultShared Yes
WebInterface Yes
Listen /var/run/cups/cups.sock

ServerAlias *

<Location />
Order Allow,Deny
Allow From 127.0.0.1
Allow From 192.168.1.0/24
</Location>

<Location /admin>
AuthType Basic
AuthClass System
Order Allow,Deny
Allow From 127.0.0.1
Allow From 192.168.1.0/24
</Location>

```

restart cups

```
/etc/init.d/cupsd restart
```

### Step 5
Configure the printer in cups.

Go to http://192.168.0.1:631

Add your printer. If your printer is plug via USB you should see it on the screen. 
If your printer is a network printer you will have to configure it manualy. You will choose the prococol compatible with your printer. Personaly I have configure mine with the socket protocol.

You can search for the ppd of your printer but airprint may have trouble.

**Note the printer name and description of the printer because you will need it for configuring airprint**

### Step 6
Configure airprint!

create the following file

```vi /etc/avahi/services/AirPrint-YOUR_PRINTER.service```

And paste this. Replace all the "YOUR_PRINTER" by the name gave in the cups configuration. 

```
<?xml version="1.0" encoding='UTF-8'?>
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
<name replace-wildcards="yes">AirPrint YOUR_PRINTER @ %h</name>
<service>
       <type>_ipp._tcp</type>
       <!-- This is key for AirPrint -->
       <subtype>_universal._sub._ipp._tcp</subtype>
       <port>631</port>
       <!-- These are copied from the txt portion of the above output -->
       <txt-record>txtvers=1</txt-record>
       <txt-record>qtotal=1</txt-record>
       <txt-record>rp=printers/YOUR_PRINTER</txt-record>
       <txt-record>ty=I make the printer Go</txt-record>
       <!-- This is visible in iPhone UI -->
       <txt-record>note=I am an AirPrint Printer</txt-record>
       <txt-record>product=(GPL Ghostscript)</txt-record>
       <txt-record>printer-state=3</txt-record>
       <txt-record>printer-type=0x3006</txt-record>
       <txt-record>Binary=T</txt-record>
       <txt-record>Transparent=T</txt-record>
       <txt-record>URF=DM3</txt-record>
       <txt-record>pdl=application/octet-stream,application/pdf,application/postscript,application/vnd.cups-raster,image/gif,image/jpeg,image/png,image/tiff,image/urf,text/html,text/plain,application/vnd.adobe-reader-postscript,application/vnd.cups-pdf</txt-record>
</service>
</service-group>

```

restart avahi

```
/etc/init.d/avahi-daemon restart
```

I don't know if the last two file are required but I have create them.

```vi /usr/share/cups/mime/airprint.convs ```

```
more airprint.convs
#
# "$Id: $"
#
# AirPrint
# Updated list with minimal set 25 Sept
image/urf application/pdf 100 pdftoraster
#
# End of "$Id: $".
#
```

```/usr/share/cups/mime/airprint.types```

```
airprint.types
#
# "$Id: $"
#
# AirPrint type
image/urf urf string(0,UNIRAST<00>)
#
# End of "$Id: $".
#
```

To make cups and avahi-demon start at boot

```/etc/init.d/cupsd enable```

```/etc/init.d/avahi-daemon enable```

### Optional step

To put the spool on the usb 

```mkdir /mnt/shares/cups_spool```

Change the spool config in /etc/cups/cupsd.conf

```
RequestRoot /mnt/shares/cups_spool

TempDir /mnt/shares/cups_spool
```

restart cups

```/etc/init.d/cupsd restart```

To debug avahi/airprint

```opkg install avahi-utils```

```avahi-browse --terminate --resolve _ipp._tcp```

```avahi-browse -rt _universal._sub._ipp._tcp```

### Sources:

https://wiki.openwrt.org/doc/howto/cups.server

http://mattie47.com/getting-cups-working-on-openwrt/

https://github.com/Gr4ffy/lede-cups

http://openrouter.info/forum/viewtopic.php?f=22&t=1856

https://forum.lede-project.org/t/compile-lede-with-error/2195/3

https://forums.linuxmint.com/viewtopic.php?t=114904


