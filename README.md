# RTL8188GU / RTL8710BU USB Wi-Fi on Debian 13

RTL8188GU / RTL8710BU USB Wi-Fi on Debian 13

The Linux drivers work for the Realtek RTL8188GU / RTL8710BU USB Wi-Fi adapter with USB ID:

0BDA:B711

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

info->beacon.head info->beacon.head_len info->beacon.tail info->beacon.tail_len 2. cfg80211_rtw_set_monitor_channel

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

A:

.usbdrv.driver.shutdown = rtw_dev_shutdown,

These changes are included in the commit:

220e7561cb0690ffc2352cdc0bfd80112ea04e6d

Commit message:

Fixed the build for Debian 13 kernel 6.12.

Installs the required build tools and kernel header files.

For a Debian system using the currently running kernel:

sudo apt install build-essential linux-headers-$(uname -r)

Clone this repository and access the source directory:

git clone https://github.com/rosasgianluigi-bot/RTL8188GU-Debian13.git cd RTL8188GU-Debian13

Compile:

make

Install:

sudo make install

After installation, update the module's dependency database with the command:

sudo depmod -a

Then reconnect the USB adapter or reinstall the driver, depending on your system's needs.

USB Device Verification

Verify that the adapter is visible with the command:

lsusb

The expected device ID is:

0bda:b711

You can also verify the USB driver association with:

lsusb -t Check Wireless Interface

List wireless interfaces:

IW development

O:

IP ​​link

Once the adapter has been successfully initialized, a wireless interface should appear.

The interface name can be generated from the adapter's MAC address, for example:

wlxXXXXXXXXXXXX NetworkManager

If NetworkManager is installed, available Wi-Fi networks can be listed with:

nmcli Wi-Fi Device List

To connect:

nmcli --ask device wifi connect "YOUR_WIFI_NAME" ifname YOUR_WIFI_INTERFACE

The --ask option allows NetworkManager to prompt for the Wi-Fi password without having to enter it on the command line or in this documentation.

Firmware

The RTL8710B platform uses firmware associated with the RTL8710B/RTL8188GU device family.

The Linux rtl8xxxu driver uses firmware files such as:

rtlwifi/rtl8710bufw_SMIC.bin rtlwifi/rtl8710bufw_UMC.bin

This repository also contains the RTL8710B firmware data embedded in the driver source code.

Firmware License

The firmware license is separate from the driver source code license.

This repository does not absolutely guarantee that the firmware binaries are released under the GPL license. Users are advised to verify the applicable license terms provided by the manufacturer/vendor before redistributing the firmware separately.

Firmware Origin and Provenance

The driver source code contains Realtek copyright notices and references to the GPLv2 license in the individual source files.

The project should therefore be considered:

A community-maintained driver based on Realtek-derived source code; it is not an official Realtek driver distribution; Modified to compile with the Debian 13 / Linux 6.12 kernel APIs; with the original source code copyright and license notices preserved.

Firmware should be treated separately from the driver source code for licensing and redistribution purposes.

Notes on switching USB modes

Some adapters based on this hardware initially present themselves as USB storage/CD-ROM devices:

0BDA:1A2B

After enabling USB mode, the wireless function becomes:

0BDA:B711

If the wireless device does not appear, checking the lsusb command before and after switching USB modes may help identify the problem.

A dark LED does not necessarily mean the adapter is not working.

Always check the actual USB port enumeration, kernel driver, wireless interface, and network connection.

Troubleshooting: USB stick not detected on reboot (boot race condition)

On modern systems such as Debian 13 (Kernel 6.12+), a synchronization problem may occur at boot: the USB stick is correctly detected by lsusb (ID 0bda:b711), but the Wi-Fi network interface (wlx...) does not appear in ip link unless you physically unplug and replug the device.

This happens because the module is loaded by the kernel before the USB stick's firmware has completed the electronic transition following the mode change.

To automatically and permanently resolve this issue on any USB port on your PC, follow these steps to create a dedicated Systemd service that performs a soft reboot of the device at boot.

1. Remove the module from preload
Make sure the module is not present in the /etc/modules file. Open the file:

sudo nano /etc/modules
If you see the 8188gu line, delete it, save (CTRL+O, Enter), and exit (CTRL+X).

2. Create the automatic restart service
Create a new service file in Systemd:

sudo nano /etc/systemd/system/rtl8188gu-restart.service

<img width="1168" height="315" alt="restart" src="https://github.com/user-attachments/assets/94b8a9df-e50e-4dc1-8ed7-a067256eb5f6" />

Save the file (CTRL+O, Enter) and exit (CTRL+X).

3. Enable the service
Inform Systemd of the change and enable the service so that it starts automatically every time the computer is turned on:

From the terminal:

sudo systemctl daemon-reload

sudo systemctl enable rtl8188gu-restart.service

Done! Upon the next reboot, the Unico/Realtek dongle will be soft-reset, and the Wi-Fi interface will be active and ready for use right from the start, regardless of the USB port it's plugged into.

Kernel Compatibility

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

Disclaimer

This repository is provided as technical documentation of a working configuration.

Hardware revisions, firmware revisions, kernel versions, and distribution configurations may vary from system to system.

There is no guarantee that the driver will work without modification on all RTL8188GU/RTL8710BU devices.

When testing a wireless driver outside of the installation package, you must always have a working network connection.

Credits and Acknowledgements
This project is a fork updated and adapted to modern Linux kernels. Special thanks to:

Thanks to @McMCCRU for the original rtl8188gu repository, which provided the source code and initial support for older releases such as Ubuntu 20.04.
Without their initial work reverse engineering and cleaning the Realtek code, it would not have been possible to extend support for this Wi-Fi dongle to current Debian releases.

Further kernel compatibility work and testing on Debian 13:

Gianluigi Rosas

Test Platform:

Debian GNU/Linux 13.7 — Linux 6.12.107

UNICO WA2763 chip Realtek Semiconductor Corp. RTL8188GU 802.11n WLAN adapter


<img width="300" height="638" alt="Unicowa2763" src="https://github.com/user-attachments/assets/cbe73a20-87c3-43ba-82c0-2c668a5c70e1" />
