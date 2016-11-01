# rtl8xxxu

These are patches to fix the mainline Linux rtl8xxxu wireless driver to work with the Cube i9 which has a Realtek rtl8723bu wireless IC.

The first commit is a patch of contents of directory
https://git.kernel.org/cgit/linux/kernel/git/jes/linux.git/tree/drivers/net/wireless/realtek/rtl8xxxu?h=rtl8xxxu-devel
as of 30 October 2016

Although the kernel is listed as a fork of linux-2.6.git, the contents of the above directory is from the Linux maintainers tree for the wireless rtl8xxxu driver.

Linux maintainers and their SCM trees are listed in https://www.kernel.org/doc/linux/MAINTAINERS

The patches have been sent to relevant Linux maintainers and kernel mailing lists.

# How to build and install the driver

You need sudo, git, linux-headers and basic devlopment tools, such as base-devel for Archlinux and Antergos or build-essential for Debian and Ubuntu.

You also need to check you have firmware at /lib/firmware/rtlwifi/rtl8723bu_nic.bin

You can get the firmware from
https://git.kernel.org/cgit/linux/kernel/git/firmware/linux-firmware.git/plain/rtlwifi/rtl8723bu_nic.bin

The firmware location is hard coded into the driver and is the same whether or not bluetooth is used as well as WLAN. For this driver the firmware size used is 32108 bytes and is v35.0

For Debian the non-free package firmware-realtek is available.

For installation purposes it is now common to boot from an ISO on a thumb USB drive and to install packages from a repository on the Internet. Without a working network driver for the Cube i9 it is necessary to use another newtork device, such as a WiFi to USB adapter, an ethernet to USB adapter or a mobile phone in tethered mode. The least troublesome method is to use an ethernet to USB adapter.

From a terminal as non root user:

```
git clone https://github.com/johnheenan/rtl8xxxu.git
cd rtl8xxxu
make install

```

The patched driver has the same system name as the mainline driver: rtl8xxxu. 

The alternative working driver (that must be installed separately if you want to use it or run tests with it) has the system name 8723bu. If you were using the 8723bu driver then check *.conf files in /etc/modprobe.d and ensure you have entries such as

```
cat /etc/modprobe.d/blacklist.conf
# blacklist rtl8xxxu
blacklist 8723bu

```
to ensure only 8723bu is blacklisted. This only affects booting. The # before blacklist prevents blacklisting

You can switch between drivers using rmmod and modprobe or by changing what is blacklisted and rebooting

You can check only one of rtl8xxxu or 8723bu has been loaded with command

```
lsmod | grep usbcore
```
The top line should have one of either 8723bu or rtl8xxxu in it. If none or both are in the line then this is incorrect.


rtl8xxxu_ops
has rtl8xxxu_start and rtl8xxxu_stop

# What the patch does

The rtl8723bu wireless IC shows evidence of a more agressive approach to power saving, powering down its RF side when there is no wireless interfacing but leaving USB interfacing intact. This makes the wireless IC more suitable for use in devices which need to keep their power use as low as practical, such as tablets and Surface Pro type devices.

In effect this means that a full initialisation must be performed whenever a wireless interface is brought up. It also means that interpretations of power status from general wireless registers should not be relied on to influence an init sequence.

The patch works by forcing a fuller initialisation and forcing it to occur more often in code paths (such as occurs during a low level authentication that initiates wireless interfacing).

The initialisation sequence is now more consistent with code based directly on vendor code. For example while the vendor derived code interprets a register as indcating a particular powered state, it does not use this information to influence its init sequence.

Only devices that use the rtl8723bu driver are affected by this patch.

With this patch wpa_supplicant reliably and consistently connects with an AP. Before a workaround such as executing rmmod and modprobe before each call to wpa_supplicant worked with some distributions.


There are only two patched files
* Makefile
* rtl8xxxu_core.c

The Makefile was edited to allow the kernel module to be built and installed outside of kernel source tree

File rtl8xxxu_8723b.c is relevant but was not patched

None of the other three .c files are relevant to the Cube i9


# Practical issues

There is evidence userspace packages may interfere with functionality so for testing it is better to start with as minimal an install as practical.

Known to work with Archlinux, wpa_supplicant and dhcpcd with no additional networking userspace packages. Archlinux which uses whatever the current Kernel release is.

It is only in recent kernels that support for Surface Pro type tablets is emerging. So it may be pointless trying anything except very recent kernels with the Cube i9.


# Kernels

Known to work with more recent series kernels as of October 2016. Due to recent kernel changes it is possible this patched driver may not work with linux 4.7 or earlier. 


# Alternatives and acknowledgements

There is an alternative working driver at https://github.com/lwfinger/rtl8723bu that is based on original manufactuer driver code by Realtek. The system name of this driver is 8723bu.

This alternative working driver was very helpful in helping developing this fix.


John Heenan
