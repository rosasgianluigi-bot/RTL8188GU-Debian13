RTL8188GU / RTL8710BU USB Wi-Fi on Debian 13

This repository documents a working configuration of a Realtek RTL8188GU / RTL8710BU USB Wi-Fi adapter on Debian 13 with the Linux 6.12 kernel series.

The tested USB device has the following identification:

VID:PID = 0BDA:B711

The configuration described here was developed and tested on Debian GNU/Linux 13 with KDE Plasma.

Important: this repository documents a tested configuration. It is not an official Realtek driver distribution.

Hardware

The tested adapter is identified as:

Realtek RTL8188GU / RTL8710B
VID:PID = 0BDA:B711

Typical USB description:

RTL8188GU 802.11n WLAN Adapter

Main characteristics:

Realtek RTL8188GU / RTL8710B family
USB 2.0
802.11n
2.4 GHz
1T1R
USB Vendor ID: 0BDA
USB Product ID: B711

Some adapters may initially appear with another USB ID before USB mode switching:

0BDA:1A2B

and subsequently switch to:

0BDA:B711

This behavior may involve usb-modeswitch.

Tested System

The configuration documented here was tested on:

Operating System: Debian GNU/Linux 13 (trixie)
Kernel: 6.12.107+deb13-amd64
Architecture: x86_64
Desktop: KDE Plasma
GCC: 14.2.0

The kernel headers used for compilation matched the running kernel.

The resulting driver module is:

8188gu.ko
Original Driver Project

The starting point for this work is:

https://github.com/McMCCRU/rtl8188gu

The original project identifies the adapter as:

RTL8188GU (RTL8710B)
VID:PID = 0x0BDA:0xB711

The original project is not an official Realtek driver distribution. It is based on source code derived from Realtek driver sources.

The original source files and their copyright/license notices are retained in this repository.

Debian 13 / Kernel 6.12 Compatibility Fixes

The original source required modifications to compile against the Debian 13 Linux 6.12 kernel headers.

os_dep/linux/ioctl_cfg80211.c
cfg80211_rtw_change_beacon

The function parameter was updated from:

struct cfg80211_beacon_data *info

to:

struct cfg80211_ap_update *info

The corresponding beacon data references were updated from:

info->head
info->head_length
info->tail
info->tail_length

to:

info->beacon.head
info->beacon.head_len
info->beacon.tail
info->beacon.tail_len
cfg80211_rtw_set_monitor_channel

The current kernel API requires the network device parameter:

struct net_device *ndev

The function declaration was updated accordingly.

os_dep/linux/usb_intf.c

The USB driver shutdown callback was changed from:

.usbdrv.drvwrap.driver.shutdown = rtw_dev_shutdown,

to:

.usbdrv.driver.shutdown = rtw_dev_shutdown,

This was required because the drvwrap member is no longer available in the kernel API used by Debian 13 / Linux 6.12.

Build and Installation

Install the required build tools and kernel headers:

sudo apt install build-essential linux-headers-$(uname -r)

Clone the repository:

git clone https://github.com/rosasgianluigi-bot/RTL8188GU-RTL8710BU-Debian13.git

Enter the source directory:

cd RTL8188GU-RTL8710BU-Debian13

Compile:

make

Install:

sudo make install

Update the module dependency database:

sudo depmod -a

After installation, reconnect the USB adapter or reload the driver according to the current system state.

USB Device Verification

Check whether the USB adapter is detected:

lsusb

The expected device is:

0bda:b711

If the adapter initially appears as:

0bda:1a2b

wait briefly and check again:

lsusb

The expected final wireless-device identification is:

0bda:b711

If the adapter does not appear, check USB enumeration and kernel messages before changing the driver configuration.

Useful commands:

lsusb

and:

dmesg | tail -n 50
USB Driver Association

To inspect the USB device and its associated driver:

lsusb -t

The wireless adapter should appear together with its associated kernel driver.

The system may contain both:

rtl8xxxu

and:

8188gu

as possible drivers for this device.

Driver selection depends on kernel configuration, aliases and driver priority.

Therefore, the presence of the 8188gu module alone does not automatically prove that it is the driver currently managing the adapter.

Wireless Interface

List wireless interfaces:

iw dev

or:

ip link

After successful initialization, a wireless interface should be present.

The interface name may look like:

wlxXXXXXXXXXXXX

The actual name depends on the USB adapter MAC address.

For the tested adapter, an example interface was:

wlx90de806b12e1
Basic Driver Verification

A useful diagnostic sequence is:

lsusb
lsusb -t
iw dev
ip link
nmcli device status

These commands verify, independently:

USB enumeration
USB driver association
wireless interface creation
network interface state
NetworkManager device state

Network connectivity should be tested independently of the physical LED.

LED Behavior
Important

The RTL8188GU / RTL8710BU adapter can operate correctly even when its physical LED is not illuminated.

Therefore:

The absence of LED activity must not be considered, by itself, evidence of a driver or hardware failure.

During analysis of the driver source, the following behavior was verified:

CONFIG_RTW_SW_LED is enabled in the driver configuration;
the LED framework is initialized;
the driver calls SwLedOn_8710BU();
the driver calls SwLedOff_8710BU();
in the current RTL8710B USB implementation, these functions update the internal LED state but do not necessarily perform a direct hardware LED/GPIO operation.

Consequently, the adapter may operate normally with:

no permanently illuminated LED;
no blinking LED;
no visible LED activity during network traffic.

The adapter can therefore be:

correctly detected by USB;
associated with a kernel driver;
represented by a wireless interface;
detected by NetworkManager;
connected to a Wi-Fi network;
used normally for network traffic;

even when the physical LED remains off.

Always verify actual operation using:

lsusb
iw dev
ip link
nmcli device status

and, when necessary, an actual network connection or Wi-Fi scan.

Firmware

The RTL8710B / RTL8188GU platform uses firmware associated with the RTL8710B device family.

The Linux rtl8xxxu driver uses firmware files including:

/lib/firmware/rtlwifi/rtl8710bufw_SMIC.bin
/lib/firmware/rtlwifi/rtl8710bufw_UMC.bin

During testing on Debian 13, the firmware loaded by rtl8xxxu was:

rtlwifi/rtl8710bufw_SMIC.bin

The firmware was successfully loaded during adapter initialization.

This repository may also contain RTL8710B firmware data embedded in the driver source.

Firmware licensing

Firmware licensing is separate from the Linux driver source-code license.

This repository does not claim that firmware binaries are released under the GPL.

Firmware redistribution should therefore be considered separately from redistribution of the driver source code.

Users should verify the applicable license and redistribution terms before redistributing firmware binaries.

USB Mode Switching

Some versions of this hardware may initially present themselves as a USB storage/CD-ROM device.

The initial identification may be:

0BDA:1A2B

After USB mode switching, the adapter may appear as:

0BDA:B711

Depending on the device and system configuration, this can be handled by:

usb-modeswitch

Check the current USB identification with:

lsusb

If the adapter initially appears as 0BDA:1A2B, wait briefly and check again:

lsusb

If the wireless device still does not appear, inspect USB enumeration and kernel messages before making driver changes.

NetworkManager

If NetworkManager is installed, check the device state with:

nmcli device status

List available Wi-Fi networks:

nmcli device wifi list

A connection can be created interactively with:

nmcli --ask device wifi connect "YOUR_WIFI_NAME" ifname YOUR_WIFI_INTERFACE

The --ask option allows NetworkManager to request the Wi-Fi password interactively instead of putting the password directly into the command line.

NetworkManager, iwd and wpa-supplicant
Important Debian 13 configuration note

During testing of this adapter on the documented Debian 13 system, Wi-Fi management was investigated in relation to:

NetworkManager
iwd
wpa-supplicant

The working configuration uses iwd rather than wpa_supplicant as the Wi-Fi supplicant.

On the tested system:

iwd

is the active Wi-Fi supplicant, while:

wpa_supplicant

is masked.

This distinction is important.

The USB driver itself does not require wpa_supplicant to be used directly. The driver provides the wireless interface to the Linux networking stack; NetworkManager and the configured Wi-Fi backend handle the network connection.

Therefore:

Do not unmask or enable wpa_supplicant merely because the RTL8188GU adapter is present.

Changing the Wi-Fi supplicant without verifying the existing NetworkManager configuration can introduce a second competing network-management path.

Verify the Wi-Fi backend

Check the state of iwd:

systemctl status iwd --no-pager

Check the state of wpa_supplicant:

systemctl status wpa_supplicant --no-pager

On the tested configuration, iwd is the active component and wpa_supplicant is not used as the active supplicant.

To verify whether wpa_supplicant is masked:

systemctl is-enabled wpa_supplicant

A result such as:

masked

means that the service has intentionally been prevented from starting.

This should not be changed unless there is a specific reason to switch the Wi-Fi management configuration.

Wi-Fi Initialization After Reboot

A specific initialization problem was encountered during testing.

After reboot, the USB adapter could be correctly detected by:

lsusb

and show:

0bda:b711

while the wireless interface was temporarily absent from:

ip link

The problem was related to initialization of the USB wireless device during the boot process.

The important point is that USB enumeration and wireless-interface creation are separate stages.

For example:

USB device detected
        |
        v
0BDA:B711 visible in lsusb
        |
        v
kernel driver initialization
        |
        v
wireless interface created
        |
        v
NetworkManager / iwd

Therefore, seeing the adapter in lsusb does not by itself prove that the wireless interface is already ready for use.

iwd Reinitialization

During the investigation, restarting iwd was found to be an effective way to reinitialize the Wi-Fi management path when the adapter was detected by USB but the wireless interface was not immediately available to NetworkManager.

Check the current state first:

systemctl status iwd --no-pager

If the interface is present but Wi-Fi management is not yet available, the tested recovery procedure was:

sudo systemctl restart iwd

Then verify:

iw dev
nmcli device status

and, if required:

nmcli device wifi list
Important

Restarting iwd should be considered a recovery/diagnostic step for the tested configuration.

It is not a requirement of the RTL8188GU driver itself.

The driver and the Wi-Fi supplicant are separate components.

wpa-supplicant

wpa_supplicant is a separate Wi-Fi authentication/supplicant component.

It is not necessary to run both:

iwd

and:

wpa_supplicant

as competing supplicants for the same NetworkManager-managed Wi-Fi interface.

On the tested Debian 13 system, wpa_supplicant was masked and iwd was used instead.

Therefore, when troubleshooting this adapter, first determine which supplicant is configured and active rather than automatically enabling wpa_supplicant.

Useful checks are:

systemctl is-enabled iwd
systemctl is-active iwd
systemctl is-enabled wpa_supplicant
systemctl is-active wpa_supplicant

The exact NetworkManager configuration should be preserved unless there is a specific reason to change it.

USB Authorization

If the adapter appears in:

lsusb

but no wireless interface is created, check whether the USB device is authorized.

First identify the actual USB device path.

For example:

/sys/bus/usb/devices/2-4/

The path is system-dependent and must not be assumed to be the same on another computer.

Check:

cat /sys/bus/usb/devices/<device>/authorized

If the result is:

0

the USB device has been detected but is not authorized.

It can be authorized manually with:

echo 1 | sudo tee /sys/bus/usb/devices/<device>/authorized

Then check:

ip link

and:

iw dev

On the tested system, USB authorization was part of the investigation of the boot-time initialization problem.

USBGuard

USBGuard was also investigated during troubleshooting.

To check its state:

systemctl is-enabled usbguard

and:

systemctl is-active usbguard

If USBGuard is not intentionally used, it should not automatically be assumed to be the cause of an adapter initialization problem.

Its state should be verified before changing any configuration.

udev USB Authorization Rule

A udev rule was tested during the investigation:

/etc/udev/rules.d/01-usb-allow-all.rules

with:

SUBSYSTEM=="usb", ACTION=="add", ENV{DEVTYPE}=="usb_device", ATTR{authorized}="1"

Inspect the rule with:

cat /etc/udev/rules.d/01-usb-allow-all.rules

After changing udev rules, reload them with:

sudo udevadm control --reload-rules

and:

sudo udevadm trigger

Then reconnect the adapter or reboot and verify:

lsusb
ip link
iw dev

udevadm test operates in test mode and does not itself write the resulting value to the device's authorized attribute.

Diagnostic Sequence

When the adapter does not work after boot, do not immediately reinstall the driver.

Use the following sequence:

lsusb
lsusb -t
ip link
iw dev
nmcli device status

Then inspect the USB device paths:

ls /sys/bus/usb/devices/

If the adapter is visible in lsusb but the wireless interface is missing, investigate USB authorization and driver initialization before changing NetworkManager or the driver.

Important: USB Detection vs Wi-Fi Availability

The following states are not equivalent:

USB device detected
kernel driver loaded
wireless interface created
NetworkManager reports Wi-Fi available
connected to a Wi-Fi network

A failure at one stage does not necessarily indicate failure of the preceding stage.

For example:

lsusb

may show:

0bda:b711

while:

ip link

does not yet show the wireless interface.

This situation should be diagnosed as an initialization problem rather than immediately treated as a defective adapter.

Kernel Compatibility

This repository specifically documents the modifications required for:

Debian 13
Linux kernel 6.12.107+deb13-amd64

Other kernel versions may require additional changes.

The original driver was not written specifically for the Debian 13 Linux 6.12 kernel API.

Verification Summary

The documented configuration was verified through the following stages:

USB enumeration of the adapter.
USB mode switching to 0BDA:B711.
RTL8710B firmware initialization.
Kernel driver initialization.
Creation of the wireless interface.
Detection of available Wi-Fi networks.
NetworkManager detection of the Wi-Fi interface.
Successful Wi-Fi operation on Debian 13.
Recovery of the Wi-Fi management path through iwd when required.
Successful initialization after reboot without requiring a physical unplug/replug operation.

The available test results do not establish that every successful network connection is necessarily managed exclusively by the custom 8188gu module.

Depending on kernel configuration and driver priority, the adapter may instead be handled by the in-tree:

rtl8xxxu

driver.

Backup

A complete local backup of the working environment was created separately from this public Git repository.

The backup may contain:

driver source code;
Git history;
firmware files used during testing;
installed kernel module;
recompiled kernel module;
system information;
checksums;
configuration files;
documentation.

The backup is intentionally kept separate from the public repository.

Troubleshooting Philosophy

When troubleshooting this adapter:

Start with non-destructive checks.
Verify USB enumeration before changing the driver.
Verify driver association before reinstalling anything.
Verify the wireless interface before changing NetworkManager.
Verify the Wi-Fi supplicant before changing iwd or wpa_supplicant.
Do not use the physical LED as the sole diagnostic indicator.
Avoid changing multiple components simultaneously.
Keep a working alternative network connection available when possible.

Useful first-level checks are:

lsusb
lsusb -t
iw dev
ip link
nmcli device status
Disclaimer

This repository is provided as technical documentation of a working configuration.

Hardware revisions, firmware revisions, kernel versions, USB controllers, NetworkManager configurations and distribution configurations may vary from system to system.

There is no guarantee that the driver will work without modification on all RTL8188GU / RTL8710BU devices.

When testing a wireless driver or changing network configuration, make sure that an alternative working network connection is available whenever possible.

Credits and Acknowledgements

This project is based on the original rtl8188gu project by:

McMCCRU

Original repository:

https://github.com/McMCCRU/rtl8188gu

The original project provided the source code and initial RTL8188GU / RTL8710B support.

Further kernel compatibility work, testing and documentation for Debian 13 / Linux 6.12:

Gianluigi Rosas
