RTL8188GU / RTL8710BU USB Wi-Fi on Debian 13

The configuration was tested on Debian 13 with the Debian 6.12 kernel series.

The Linux drivers work for the Realtek RTL8188GU / RTL8710BU USB Wi-Fi adapter with USB ID:

On the tested system, the adapter was successfully initialized by the Linux kernel
USB Wi-Fi driver stack. The device can be handled by the in-tree rtl8xxxu driver
or by the compiled 8188gu module depending on kernel configuration and driver priority.

This repository documents a working configuration tested on Debian 13 with the Debian 6.12 series kernel.

Important: This is not an original Realtek driver. The project is based on source code derived from other Realtek drivers. The original copyright and license notices contained in the source files have been retained.

Hardware Chip Family: Realtek RTL8710B / RTL8188GU USB Vendor ID: 0BDA USB Product ID: B711 USB Description: 802.11n WLAN Adapter Bus: USB 2.0 Wi-Fi: 802.11n, 2.4 GHz Architecture: 1T1R

The adapter may initially be listed as:

0BDA:1A2B

and then change to:

0BDA:B711

via usb-modeswitch.

Tested System

The configuration documented here was tested on:

Operating System: Debian GNU/Linux 13.7 (trixie) Kernel: 6.12.107+deb13-amd64 GCC: 14.2.0 Architecture: x86_64 Desktop: KDE Plasma

The kernel header files used for compilation matched those of the running kernel.

Result

The driver source code was successfully adapted and compiled for the Debian 13 6.12 kernel.

The resulting kernel module is:

8188gu.ko

The adapter can be detected as an RTL8710B/RTL8188GU device and can create a wireless network interface.

<<<<<<< HEAD
The wireless interface was created successfully during testing.

Depending on kernel configuration and driver priority, the device may be handled by the in-tree rtl8xxxu driver or by the compiled 8188gu module.

During testing, a working wireless connection was verified using NetworkManager. Available kernel logs do not indicate that this connection was managed exclusively by the custom 8188gu module.

During testing, a working wireless connection was verified using NetworkManager. Available kernel logs do not indicate that this connection was managed exclusively by the custom 8188gu module.


Interface Example:

wlxXXXXXXXXXXXX

<img width="505" height="47" alt="nmcli device" src="https://github.com/user-attachments/assets/b7d25f36-914c-4b66-a417-7ed3cdfa4f91" />

The actual interface name depends on the USB adapter's MAC address.

LED Behavior

The RTL8188GU USB adapter can function properly even without LED activity.

Therefore, LED activity should not be used as the sole indicator of proper Wi-Fi adapter operation.

The analysis of the determining factors revealed the following:

CONFIG_RTW_SW_LED is enabled in the driver configuration.

The LED framework has been successfully initialized.

<<<<<<< HEAD
The driver code calls SwLedOn_8710BU() and SwLedOff_8710BU(), but in the current RTL8710B USB implementation these functions only update the internal LED state (bLedOn) and do not perform direct hardware writes to the LED registers/GPIOs.

Therefore:

a missing solid-state LED;

a missing blinking LED during traffic;

no LED activity after connection.

This should not be considered evidence of a driver or hardware failure.

The adapter can be fully detected, managed by the driver, connected to a WiFi network, and function normally without any visible LED indication.

Use the lsusb, iw dev, ip link, and NetworkManager commands to verify operation.

Original Project

The compilation calls SwLedOn_8710BU() and SwLedOff_8710BU(), but does not write it.

However, in the current RTL8710B USB implementation, these functions only update the internal LED state (bLedOn) and do not perform hardware writes to the LED registers/GPIOs.

Therefore:

a missing solid-state LED;

a missing blinking LED during traffic;

no LED activity after connection.

This should not be considered evidence of a driver or hardware failure.

The adapter can be fully detected, managed by the driver, connected to a WiFi network, and function normally without any visible LED indication.

Use the lsusb, iw dev, ip link, and NetworkManager commands to verify operation.

Original Project

The starting point of this work is:

McMCCRU/rtl8188gu

The original project identifies the device as:

RTL8188GU (RTL8710B) — VID:PID 0x0BDA:0xB711

The original repository did not contain a separate LICENSE file. Its source files contain the original copyright notices and the GPLv2 license, where applicable.

This repository therefore preserves the original source files and their copyright/license headers.

Fixes for Debian 13 / Kernel 6.12

The original source code needed modifications to compile correctly with the Debian 13 6.12 kernel headers.

The following files have been modified.

os_dep/linux/ioctl_cfg80211.c

cfg80211_rtw_change_beacon

The function parameter was updated from:

struct cfg80211_beacon_data *info

To:

struct cfg80211_ap_update *info

As a result, the beacon data references were updated from:

info->head info->head_length info->tail info->tail_length

To:

info->beacon.head
info->beacon.head_len
info->beacon.tail
info->beacon.tail_len

2. cfg80211_rtw_set_monitor_channel


The current kernel API requires the network device parameter:

struct net_device *ndev

Before:

struct cfg80211_chan_def *chandef

The function declaration has been updated accordingly.

os_dep/linux/usb_intf.c

The USB driver shutdown callback function has been updated from:

.usbdrv.drvwrap.driver.shutdown = rtw_dev_shutdown,

To:

.usbdrv.driver.shutdown = rtw_dev_shutdown,

These changes are included in the commit:

220e7561cb0690ffc2352cdc0bfd80112ea04e6d

Commit message:

Fixed the build for Debian 13 kernel 6.12.

The USB driver shutdown callback function has been updated from:


.usbdrv.drvwrap.driver.shutdown = rtw_dev_shutdown,

To:

.usbdrv.driver.shutdown = rtw_dev_shutdown,

These changes are included in the commit:

220e7561cb0690ffc2352cdc0bfd80112ea04e6d

Commit message:

Fixed the build for Debian 13 kernel 6.12.

## Build and Installation

Installs the required build tools and kernel header files.

For a Debian system using the currently running kernel:


sudo apt install build-essential linux-headers-$(uname -r)


Clone this repository and access the source directory:


git clone https://github.com/rosasgianluigi-bot/RTL8188GU-RTL8710BU-Debian13.git
cd RTL8188GU-RTL8710BU-Debian13


Compile the driver:

make

Install the compiled module:

sudo make install

After installation, update the module dependency database:


sudo depmod -a

Then reconnect the USB adapter or reload the driver, depending on the current system state.

## USB Device Verification

Verify that the adapter is detected by the USB subsystem:


lsusb


The expected USB device ID is:


0bda:b711


The adapter may initially appear as:

0bda:1a2b

and then switch to:

0bda:b711

after USB mode switching.

### USB Driver Association

To verify which USB driver is associated with the device:

lsusb -t

Look for the wireless adapter and its associated kernel driver.

### Wireless Interface

List the available wireless interfaces:


iw dev


or:

ip link


After successful initialization, a wireless interface should appear.

The interface name may be generated from the adapter's MAC address, for example:


wlxXXXXXXXXXXXX


The actual interface name depends on the USB adapter's MAC address.

### NetworkManager

If NetworkManager is installed, check the device status:


nmcli device status


Available Wi-Fi networks can be listed with:


nmcli device wifi list


To connect to a network:

nmcli --ask device wifi connect "YOUR_WIFI_NAME" ifname YOUR_WIFI_INTERFACE


The `--ask` option allows NetworkManager to request the Wi-Fi password interactively without storing the password in the command line or in this documentation.

### Basic Verification Sequence

A simple verification sequence is:

lsusb
lsusb -t
iw dev
ip link
nmcli device status

These checks verify the USB enumeration, kernel driver association, wireless interface creation, and NetworkManager device status independently of the physical LED.


## Firmware

The RTL8710B platform uses firmware associated with the RTL8710B / RTL8188GU device family.

The Linux `rtl8xxxu` driver uses firmware files such as:


rtlwifi/rtl8710bufw_SMIC.bin
rtlwifi/rtl8710bufw_UMC.bin


During testing on Debian 13, the firmware file loaded by `rtl8xxxu` was:


rtlwifi/rtl8710bufw_SMIC.bin

The firmware was successfully loaded by the kernel during adapter initialization.

This repository also contains RTL8710B firmware data embedded in the driver source code.

**Important:** The firmware is separate from the Linux driver source code and should be treated separately for licensing and redistribution purposes.


Firmware License

The firmware license is separate from the driver source code license.

This repository does not absolutely guarantee that the firmware binaries are released under the GPL license. Users are advised to verify the applicable license terms provided by the manufacturer/vendor before redistributing the firmware separately.

Firmware Origin and Provenance

The driver source code contains Realtek copyright notices and references to the GPLv2 license in the individual source files.

The project should therefore be considered:

A community-maintained driver based on Realtek-derived source code; it is not an official Realtek driver distribution; Modified to compile with the Debian 13 / Linux 6.12 kernel APIs; with the original source code copyright and license notices preserved.

Firmware should be treated separately from the driver source code for licensing and redistribution purposes.

## Notes on Switching USB Modes

Some adapters based on this hardware may initially present themselves as a USB storage or CD-ROM device.

The initial USB identification may be:


0BDA:1A2B


After the USB mode is switched, the wireless adapter may appear as:


0BDA:B711


This mode switching can be handled by `usb-modeswitch`, depending on the device and system configuration.

To check the current USB identification:

lsusb

If the adapter is initially listed as `0BDA:1A2B`, wait briefly and check again after the mode-switching process:


lsusb


The expected wireless-device identification is:


0BDA:B711


If the wireless device does not appear, check the USB enumeration and kernel messages before making any driver changes.

For example:


lsusb


and:

dmesg | tail -n 50


A dark LED does not necessarily indicate that the adapter has failed.

Always verify the actual USB enumeration, kernel driver association, wireless interface, and network connectivity independently of the physical LED.


Before creating custom reset services, verify USB authorization and USB security tools.

On modern systems such as Debian 13 (Kernel 6.12+), a synchronization problem may occur at boot: the USB stick is correctly detected by lsusb (ID 0bda:b711), but the Wi-Fi network interface (wlx...) does not appear in ip link unless you physically unplug and replug the device.

USB authorization can be affected by USBGuard, kernel parameters,
udev rules, or system security policies.

Check:

systemctl status usbguard

If USBGuard is not required, it can be disabled:

sudo systemctl disable --now usbguard

Verify:

systemctl is-enabled usbguard
systemctl is-active usbguard


### USB authorization verification

If lsusb detects the adapter but no wireless interface appears:

Check:

cat /sys/bus/usb/devices/<device>/authorized

If the result is:

0

the USB device is detected but not authorized.

Enable it with:

echo 1 | sudo tee /sys/bus/usb/devices/<device>/authorized

After authorization, the wireless interface should appear.

Example:

cat /sys/bus/usb/devices/<device>/authorized

returns:

0

After authorization:

echo 1 | sudo tee /sys/bus/usb/devices/<device>/authorized

the wireless interface appears.

If the problem persists after checking USB authorization settings, a systemd reset service can be used to automatically reinitialize the adapter during boot.
The exact cause may depend on USB controller timing, device firmware initialization,
or system USB authorization policies.

Troubleshooting
USB adapter detected but wireless interface is missing after reboot

On the tested Debian 13 system, the USB adapter could be detected correctly after boot with:

lsusb

showing:

0bda:b711

while the wireless interface was not initially present in:

ip link

The problem was related to USB device initialization during the boot process.

Check USB authorization

If the adapter appears in lsusb but no wireless interface is created, check whether the USB device is authorized:

cat /sys/bus/usb/devices/2-4/authorized

The device path can be different on another system. Use the actual USB device path shown by the system.

If the result is:

0

the USB device has been detected but is not authorized.

To authorize the device manually:

echo 1 | sudo tee /sys/bus/usb/devices/2-4/authorized

After authorization, check:

ip link

and:

iw dev

On the tested system, authorizing the USB device caused the wireless interface to appear.

USBGuard

USBGuard was also checked during the investigation.

To verify its state:

systemctl is-enabled usbguard

and:

systemctl is-active usbguard

If USBGuard is not intentionally used on the system, it should not be assumed to be the cause of the problem. Check its state before making configuration changes.

udev USB authorization rule

A udev rule was tested to automatically authorize USB devices:

/etc/udev/rules.d/01-usb-allow-all.rules

with:

SUBSYSTEM=="usb", ACTION=="add", ENV{DEVTYPE}=="usb_device", ATTR{authorized}="1"

The rule can be inspected with:

cat /etc/udev/rules.d/01-usb-allow-all.rules

The rule was verified with udevadm, but note that udevadm test operates in test mode and does not itself write the resulting value to the device's authorized attribute.

After changing udev rules, reload the rules:

sudo udevadm control --reload-rules

and:

sudo udevadm trigger

Then reconnect the adapter or reboot and verify:

lsusb
ip link
iw dev
Important

The exact USB device path, for example:

2-4

is system-dependent and must not be assumed to be identical on another computer.

The diagnostic sequence should therefore be:

lsusb
ip link
iw dev
ls /sys/bus/usb/devices/

and then inspect the authorized attribute of the corresponding USB device.

No custom systemd reset service is required as part of the documented configuration.

The tested system was successfully able to initialize the adapter after boot without requiring a physical unplug/replug operation.

This repository specifically documents the changes required for the tested Debian 13 kernel:

6.12.107+deb13-amd64

Other kernel versions may require additional changes.

Verification Summary

The documented configuration was verified through the following steps:

USB device enumeration. Switching USB mode to 0BDA:B711. Initializing the RTL8710B firmware. Creating the wireless interface. Detecting the wireless network. Connection to NetworkManager was successfully tested. The adapter was successfully used for Wi-Fi on Debian 13; the documented logs do not indicate exclusive use of the custom 8188gu module for this connection.

Backup

A complete local backup of the working environment was created separately from this Git repository.

The backup contains:

Driver source code; Git history; firmware files used during testing; installed kernel module; recompiled kernel module; system information; checksum; documentation.

The backup is intentionally kept separate from the public Git repository.
Kernel Compatibility

This repository specifically documents the changes required for the tested Debian 13 kernel:

6.12.107+deb13-amd64

Other kernel versions may require additional changes.

Verification Summary

The documented configuration was verified through the following steps:

USB device enumeration.
USB mode switching to 0BDA:B711.
RTL8710B firmware initialization.
Creation of the wireless interface.
Detection of available Wi-Fi networks.
Successful connection using NetworkManager.
Successful use of the adapter for Wi-Fi on Debian 13.

The available test logs do not establish that the successful network connection was managed exclusively by the custom 8188gu module. Depending on kernel configuration and driver priority, the device may instead be handled by the in-tree rtl8xxxu driver.

Backup

A complete local backup of the working environment was created separately from this Git repository.

The backup contains, where applicable:

driver source code;
Git history;
firmware files used during testing;
installed kernel module;
recompiled kernel module;
system information;
checksums;
documentation.

The backup is intentionally kept separate from the public Git repository.

Disclaimer

This repository is provided as technical documentation of a working configuration.

Hardware revisions, firmware revisions, kernel versions, USB controllers, and distribution configurations may vary from system to system.

There is no guarantee that the driver will work without modification on all RTL8188GU / RTL8710BU devices.

When testing a wireless driver or changing network configuration, always make sure that an alternative working network connection is available when possible.

Credits and Acknowledgements

This project is based on the work contained in the original rtl8188gu project by McMCCRU and has been adapted and tested for modern Linux kernels.

Special thanks to:

@McMCCRU

for the original rtl8188gu repository, which provided the source code and initial support for older Linux releases.

Further kernel compatibility work and testing on Debian 13:

Gianluigi Rosas

Test Platform
Operating System: Debian GNU/Linux 13.7
Kernel: 6.12.107+deb13-amd64
Desktop: KDE Plasma
Adapter: UNICO WA2763
Chip: Realtek RTL8188GU / RTL8710BU
USB ID: 0BDA:B711
Wi-Fi: 802.11n, 2.4 GHz

UNICO WA2763 chip Realtek Semiconductor Corp. RTL8188GU 802.11n WLAN adapter


<img width="300" height="638" alt="Unicowa2763" src="https://github.com/user-attachments/assets/cbe73a20-87c3-43ba-82c0-2c668a5c70e1" />
