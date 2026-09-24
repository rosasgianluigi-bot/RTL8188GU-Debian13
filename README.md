RTL8188GU / RTL8710BU USB Wi-Fi on Debian 13

Linux driver work for the Realtek RTL8188GU / RTL8710BU USB Wi-Fi adapter with USB ID:

0BDA:B711

This repository documents a working setup tested on Debian 13 with the Debian 6.12 kernel series.

Important: this is not an original Realtek driver. The project is based on source code derived from other Realtek driver sources. Original copyright and license notices contained in the source files have been preserved.

Hardware
Chip family: Realtek RTL8710B / RTL8188GU
USB Vendor ID: 0BDA
USB Product ID: B711
USB description: 802.11n WLAN Adapter
Bus: USB 2.0
Wi-Fi: 802.11n, 2.4 GHz
Architecture: 1T1R

The adapter may initially enumerate as:

0BDA:1A2B

and then switch to:

0BDA:B711

through usb-modeswitch.

Verified System

The configuration documented here was tested on:

OS: Debian GNU/Linux 13.7 (trixie)
Kernel: 6.12.107+deb13-amd64
GCC: 14.2.0
Architecture: x86_64
Desktop: KDE Plasma

The kernel headers used for compilation matched the running kernel.

Result

The driver source was successfully adapted and compiled for the Debian 13 kernel 6.12 series.

The resulting kernel module is:

8188gu.ko

The adapter can be detected as an RTL8710B/RTL8188GU device and can create a wireless network interface.

A working wireless connection was verified with NetworkManager.

Example interface:

wlxXXXXXXXXXXXX

The actual interface name depends on the USB adapter's MAC address.

LED behaviour

The adapter may operate correctly even when its physical blue LED is not illuminated.

Therefore, LED activity must not be used as the only indication that the Wi-Fi adapter is working.

Use lsusb, iw dev, ip link and NetworkManager to verify operation.

Original Project

The starting point for this work is:

McMCCRU/rtl8188gu

The original project identifies the device as:

RTL8188GU (RTL8710B) — VID:PID 0x0BDA:0xB711

The original repository did not contain a separate LICENSE file. Its source files contain original copyright and GPLv2 notices where applicable.

This repository therefore preserves the original source files and their existing copyright/license headers.

Debian 13 / Kernel 6.12 Fixes

The original source required changes to build correctly against the Debian 13 kernel 6.12 headers.

The following files were modified.

os_dep/linux/ioctl_cfg80211.c
1. cfg80211_rtw_change_beacon

The function parameter was updated from:

struct cfg80211_beacon_data *info

to:

struct cfg80211_ap_update *info

The beacon data references were consequently updated from:

info->head
info->head_len
info->tail
info->tail_len

to:

info->beacon.head
info->beacon.head_len
info->beacon.tail
info->beacon.tail_len
2. cfg80211_rtw_set_monitor_channel

The current kernel API requires the network device parameter:

struct net_device *ndev

before:

struct cfg80211_chan_def *chandef

The function declaration was updated accordingly.

os_dep/linux/usb_intf.c

The USB driver shutdown callback was updated from:

.usbdrv.drvwrap.driver.shutdown = rtw_dev_shutdown,

to:

.usbdrv.driver.shutdown = rtw_dev_shutdown,

These changes are included in commit:

220e7561cb0690ffc2352cdc0bfd80112ea04e6d

Commit message:

Fix build for Debian 13 kernel 6.12

Building

Install the required build tools and kernel headers.

For a Debian system using the currently running kernel:

sudo apt install build-essential linux-headers-$(uname -r)

Clone this repository and enter the source directory:

git clone https://github.com/rosasgianluigi-bot/RTL8188GU-Debian13.git
cd RTL8188GU-Debian13

Compile:

make

Install:

sudo make install

After installation, refresh the module dependency database:

sudo depmod -a

Then reconnect the USB adapter or reload the driver as appropriate for the system.

Checking the USB Device

Verify that the adapter is visible:

lsusb

The expected device ID is:

0bda:b711

You can also check the USB driver association with:

lsusb -t
Checking the Wireless Interface

List wireless interfaces:

iw dev

or:

ip link

A wireless interface should appear when the adapter has been successfully initialized.

The interface name may be generated from the adapter's MAC address, for example:

wlxXXXXXXXXXXXX
NetworkManager

If NetworkManager is installed, available Wi-Fi networks can be listed with:

nmcli device wifi list

To connect:

nmcli --ask device wifi connect "YOUR_WIFI_NAME" ifname YOUR_WIFI_INTERFACE

The --ask option allows NetworkManager to request the Wi-Fi password without placing it in the command line or in this documentation.

Firmware

The RTL8710B platform uses firmware associated with the RTL8710B/RTL8188GU device family.

The Linux rtl8xxxu driver uses firmware files such as:

rtlwifi/rtl8710bufw_SMIC.bin
rtlwifi/rtl8710bufw_UMC.bin

This repository also contains RTL8710B firmware data embedded in the driver source.

Firmware licensing

Firmware licensing is separate from the licensing of the driver source code.

This repository does not make a blanket claim that firmware binaries are GPL-licensed. Users should verify the applicable upstream/vendor terms before redistributing firmware separately.

Source and Firmware Provenance

The driver source contains Realtek copyright notices and GPLv2 references in individual source files.

The project should therefore be understood as:

a community-maintained driver based on Realtek-derived source code;
not an official Realtek driver distribution;
modified to build against the Debian 13 / Linux 6.12 kernel API;
with original source copyright and license notices preserved.

Firmware should be treated separately from the driver source with respect to licensing and redistribution.

Known Considerations
USB mode switching

Some adapters based on this hardware initially appear as a USB storage/CD-ROM device:

0BDA:1A2B

After usb-modeswitch, the wireless function becomes:

0BDA:B711

If the wireless device does not appear, checking lsusb before and after USB mode switching can help identify the problem.

LED

A non-illuminated LED does not necessarily mean that the adapter is not working.

Always verify the actual USB enumeration, kernel driver, wireless interface and network connection.

Kernel compatibility

This repository specifically documents the modifications required for the tested Debian 13 kernel:

6.12.107+deb13-amd64

Other kernel versions may require additional changes.

Verification Summary

The documented setup has been verified through the following stages:

USB device enumeration.
USB mode switching to 0BDA:B711.
RTL8710B firmware initialization.
Wireless interface creation.
Wireless network detection.
NetworkManager connection.
Successful Wi-Fi operation on Debian 13.
Backup

A complete local backup of the working environment was created separately from this Git repository.

The backup contains:

driver source;
Git history;
firmware files used during testing;
installed kernel module;
rebuilt kernel module;
system information;
checksums;
documentation.

The backup is intentionally kept separate from the public Git repository.

Disclaimer

This repository is provided as a technical record of a working configuration.

Hardware revisions, firmware revisions, kernel versions and distribution configurations may differ between systems.

No guarantee is made that the driver will work unchanged on every RTL8188GU / RTL8710BU device.

Always keep a working network connection available when testing an out-of-tree wireless driver.

Credits

Original project:

McMCCRU/rtl8188gu

Additional kernel compatibility work and Debian 13 testing:

Gianluigi Rosas

Test platform:

Debian GNU/Linux 13.7 — Linux 6.12.107
