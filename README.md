# RTL8188GU / RTL8710BU USB Wi-Fi on Debian 13

The configuration documented in this repository was tested on Debian 13 with the Debian 6.12 kernel series.

The Linux driver stack supports the Realtek RTL8188GU / RTL8710BU USB Wi-Fi adapter with USB ID:

```text
0BDA:B711
```

On the tested system, the adapter was successfully initialized by the Linux kernel USB Wi-Fi driver stack. Depending on kernel configuration and driver priority, the device may be handled by the in-tree `rtl8xxxu` driver or by the compiled `8188gu` module.

This repository documents a working configuration tested on Debian 13 with the Debian 6.12 kernel series.

**Important:** This is not an original Realtek driver. The project is based on source code derived from other Realtek drivers. The original copyright and license notices contained in the source files have been retained.

## Hardware

* **Chip family:** Realtek RTL8710B / RTL8188GU
* **USB Vendor ID:** `0BDA`
* **USB Product ID:** `B711`
* **USB description:** 802.11n WLAN Adapter
* **USB bus:** USB 2.0
* **Wi-Fi:** 802.11n, 2.4 GHz
* **Architecture:** 1T1R

The adapter may initially be listed as:

```text
0BDA:1A2B
```

and then change to:

```text
0BDA:B711
```

via `usb-modeswitch`.

## Tested System

The configuration documented here was tested on:

* **Operating System:** Debian GNU/Linux 13.7 (trixie)
* **Kernel:** `6.12.107+deb13-amd64`
* **GCC:** `14.2.0`
* **Architecture:** x86_64
* **Desktop:** KDE Plasma

The kernel header files used for compilation matched those of the running kernel.

## Result

The driver source code was successfully adapted and compiled for the Debian 13 / Linux 6.12 kernel.

The resulting kernel module is:

```text
8188gu.ko
```

The adapter can be detected as an RTL8710B / RTL8188GU device and can create a wireless network interface.

Depending on kernel configuration and driver priority, the device may be handled by the in-tree `rtl8xxxu` driver or by the compiled `8188gu` module.

During testing, a working wireless connection was verified using NetworkManager. Available kernel logs do not establish that this connection was managed exclusively by the custom `8188gu` module.

### Interface Example

```text
wlxXXXXXXXXXXXX
```

The actual interface name depends on the USB adapter's MAC address.

<img width="505" height="47" alt="nmcli device" src="https://github.com/user-attachments/assets/b7d25f36-914c-4b66-a417-7ed3cdfa4f91" />

## LED Behavior

The RTL8188GU / RTL8710BU USB adapter can function correctly even when its physical LED is not illuminated.

Therefore, LED activity must not be used as the sole indicator of whether the adapter is working.

During the analysis of the driver source code, the following points were verified:

* `CONFIG_RTW_SW_LED` is enabled in the driver configuration.
* The LED framework is initialized.
* The driver calls `SwLedOn_8710BU()` and `SwLedOff_8710BU()`.
* In the current RTL8710B USB implementation, these functions update the internal LED state (`bLedOn`) but do not perform direct hardware writes to the LED registers/GPIOs.

Therefore, the following conditions are possible while the adapter is operating normally:

* no solid LED;
* no blinking LED during network traffic;
* no LED activity after connecting to a Wi-Fi network.

The absence of LED activity alone should **not** be considered evidence of a driver or hardware failure.

The adapter can be correctly detected, managed by the driver, connected to a Wi-Fi network, and used normally without any visible LED indication.

To verify actual operation, use:

```bash
lsusb
iw dev
ip link
nmcli device status
```

Network connectivity should be verified independently of the physical LED.

## Original Project

The starting point of this work is:

**McMCCRU/rtl8188gu**

The original project identifies the device as:

**RTL8188GU (RTL8710B) — VID:PID `0x0BDA:0xB711`**

The original repository did not contain a separate `LICENSE` file. Its source files contain the original copyright notices and the GPLv2 license, where applicable.

This repository therefore preserves the original source files and their copyright and license headers.

## Fixes for Debian 13 / Kernel 6.12

The original source code required modifications to compile correctly with the Debian 13 / Linux 6.12 kernel headers.

The following files were modified.

### `os_dep/linux/ioctl_cfg80211.c`

#### `cfg80211_rtw_change_beacon`

The function parameter was updated from:

```c
struct cfg80211_beacon_data *info
```

to:

```c
struct cfg80211_ap_update *info
```

As a result, the beacon data references were updated from:

```c
info->head
info->head_length
info->tail
info->tail_length
```

to:

```c
info->beacon.head
info->beacon.head_len
info->beacon.tail
info->beacon.tail_len
```

#### `cfg80211_rtw_set_monitor_channel`

The current kernel API requires the network device parameter:

```c
struct net_device *ndev
```

The function declaration was updated accordingly.

### `os_dep/linux/usb_intf.c`

The USB driver shutdown callback was updated from:

```c
.usbdrv.drvwrap.driver.shutdown = rtw_dev_shutdown,
```

to:

```c
.usbdrv.driver.shutdown = rtw_dev_shutdown,
```

These changes are included in the commit:

```text
220e7561cb0690ffc2352cdc0bfd80112ea04e6d
```

Commit message:

```text
Fixed the build for Debian 13 kernel 6.12.
```

## Build and Installation

Install the required build tools and kernel header files:

```bash
sudo apt install build-essential linux-headers-$(uname -r)
```

Clone this repository and access the source directory:

```bash
git clone https://github.com/rosasgianluigi-bot/RTL8188GU-RTL8710BU-Debian13.git
cd RTL8188GU-RTL8710BU-Debian13
```

Compile the driver:

```bash
make
```

Install the compiled module:

```bash
sudo make install
```

After installation, update the module dependency database:

```bash
sudo depmod -a
```

Then reconnect the USB adapter or reload the driver, depending on the current system state.

## USB Device Verification

Verify that the adapter is detected by the USB subsystem:

```bash
lsusb
```

The expected USB device ID is:

```text
0bda:b711
```

The adapter may initially appear as:

```text
0bda:1a2b
```

and then switch to:

```text
0bda:b711
```

after USB mode switching.

### USB Driver Association

To verify which USB driver is associated with the device:

```bash
lsusb -t
```

Look for the wireless adapter and its associated kernel driver.

### Wireless Interface

List the available wireless interfaces:

```bash
iw dev
```

or:

```bash
ip link
```

After successful initialization, a wireless interface should appear.

The interface name may be generated from the adapter's MAC address, for example:

```text
wlxXXXXXXXXXXXX
```

The actual interface name depends on the USB adapter's MAC address.

### NetworkManager

If NetworkManager is installed, check the device status:

```bash
nmcli device status
```

Available Wi-Fi networks can be listed with:

```bash
nmcli device wifi list
```

To connect to a network:

```bash
nmcli --ask device wifi connect "YOUR_WIFI_NAME" ifname YOUR_WIFI_INTERFACE
```

The `--ask` option allows NetworkManager to request the Wi-Fi password interactively without storing the password in the command line or in this documentation.

### Basic Verification Sequence

A simple verification sequence is:

```bash
lsusb
lsusb -t
iw dev
ip link
nmcli device status
```

These checks verify USB enumeration, kernel driver association, wireless interface creation, and NetworkManager device status independently of the physical LED.

## Firmware

The RTL8710B platform uses firmware associated with the RTL8710B / RTL8188GU device family.

The Linux `rtl8xxxu` driver uses firmware files such as:

```text
rtlwifi/rtl8710bufw_SMIC.bin
rtlwifi/rtl8710bufw_UMC.bin
```

During testing on Debian 13, the firmware file loaded by `rtl8xxxu` was:

```text
rtlwifi/rtl8710bufw_SMIC.bin
```

The firmware was successfully loaded by the kernel during adapter initialization.

This repository also contains RTL8710B firmware data embedded in the driver source code.

**Important:** The firmware is separate from the Linux driver source code and should be treated separately for licensing and redistribution purposes.

## Firmware License

The firmware license is separate from the driver source code license.

This repository does not claim that the firmware binaries are released under the GPL license.

Users should verify the applicable license and redistribution terms provided by the manufacturer or vendor before redistributing firmware binaries separately.

## Firmware Origin and Provenance

The driver source code contains Realtek copyright notices and references to the GPLv2 license in the individual source files.

This project should therefore be considered:

* a community-maintained driver based on Realtek-derived source code;
* not an official Realtek driver distribution;
* modified to compile with the Debian 13 / Linux 6.12 kernel APIs;
* distributed with the original source-code copyright and license notices preserved.

Firmware should be treated separately from the driver source code for licensing and redistribution purposes.

## Notes on Switching USB Modes

Some adapters based on this hardware may initially present themselves as a USB storage or CD-ROM device.

The initial USB identification may be:

```text
0BDA:1A2B
```

After the USB mode is switched, the wireless adapter may appear as:

```text
0BDA:B711
```

This mode switching can be handled by `usb-modeswitch`, depending on the device and system configuration.

To check the current USB identification:

```bash
lsusb
```

If the adapter is initially listed as `0BDA:1A2B`, wait briefly and check again after the mode-switching process:

```bash
lsusb
```

The expected wireless-device identification is:

```text
0BDA:B711
```

If the wireless device does not appear, check the USB enumeration and kernel messages before making any driver changes.

For example:

```bash
lsusb
```

and:

```bash
dmesg | tail -n 50
```

A dark LED does not necessarily indicate that the adapter has failed.

Always verify the actual USB enumeration, kernel driver association, wireless interface, and network connectivity independently of the physical LED.

## Troubleshooting

### USB Adapter Detected but Wireless Interface Is Missing After Reboot

On the tested Debian 13 system, the USB adapter could be detected correctly after boot with:

```bash
lsusb
```

showing:

```text
0bda:b711
```

while the wireless interface was not initially present in:

```bash
ip link
```

The problem was related to USB device initialization during the boot process.

### Check USB Authorization

If the adapter appears in `lsusb` but no wireless interface is created, check whether the USB device is authorized:

```bash
cat /sys/bus/usb/devices/<device>/authorized
```

The `<device>` path is system-dependent and must be replaced with the actual USB device path.

If the result is:

```text
0
```

the USB device has been detected but is not authorized.

To authorize the device manually:

```bash
echo 1 | sudo tee /sys/bus/usb/devices/<device>/authorized
```

After authorization, check:

```bash
ip link
```

and:

```bash
iw dev
```

On the tested system, authorizing the USB device caused the wireless interface to appear.

### USBGuard

USBGuard was also checked during the investigation.

To verify its state:

```bash
systemctl is-enabled usbguard
```

and:

```bash
systemctl is-active usbguard
```

If USBGuard is not intentionally used on the system, it should not be assumed to be the cause of the problem.

Check its state before making configuration changes.

### udev USB Authorization Rule

A udev rule was tested to automatically authorize USB devices:

```text
/etc/udev/rules.d/01-usb-allow-all.rules
```

with:

```text
SUBSYSTEM=="usb", ACTION=="add", ENV{DEVTYPE}=="usb_device", ATTR{authorized}="1"
```

The rule can be inspected with:

```bash
cat /etc/udev/rules.d/01-usb-allow-all.rules
```

The rule was verified with `udevadm`.

Note that `udevadm test` operates in test mode and does not itself write the resulting value to the device's `authorized` attribute.

After changing udev rules, reload the rules:

```bash
sudo udevadm control --reload-rules
```

and:

```bash
sudo udevadm trigger
```

Then reconnect the adapter or reboot and verify:

```bash
lsusb
```

```bash
ip link
```

```bash
iw dev
```

### Important

The exact USB device path, for example:

```text
2-4
```

is system-dependent and must not be assumed to be identical on another computer.

The diagnostic sequence should therefore be:

```bash
lsusb
ip link
iw dev
ls /sys/bus/usb/devices/
```

Then inspect the `authorized` attribute of the corresponding USB device.

No custom systemd reset service is required as part of the documented configuration.

The tested system was successfully able to initialize the adapter after boot without requiring a physical unplug/replug operation.

## Kernel Compatibility

This repository specifically documents the changes required for the tested Debian 13 kernel:

```text
6.12.107+deb13-amd64
```

Other kernel versions may require additional changes.

## Verification Summary

The documented configuration was verified through the following steps:

1. USB device enumeration.
2. USB mode switching to `0BDA:B711`.
3. RTL8710B firmware initialization.
4. Creation of the wireless interface.
5. Detection of available Wi-Fi networks.
6. Successful connection using NetworkManager.
7. Successful use of the adapter for Wi-Fi on Debian 13.

The available test logs do not establish that the successful network connection was managed exclusively by the custom `8188gu` module.

Depending on kernel configuration and driver priority, the device may instead be handled by the in-tree `rtl8xxxu` driver.

## Backup

A complete local backup of the working environment was created separately from this Git repository.

The backup contains, where applicable:

* driver source code;
* Git history;
* firmware files used during testing;
* installed kernel module;
* recompiled kernel module;
* system information;
* checksums;
* documentation.

The backup is intentionally kept separate from the public Git repository.

## Disclaimer

This repository is provided as technical documentation of a working configuration.

Hardware revisions, firmware revisions, kernel versions, USB controllers, and distribution configurations may vary from system to system.

There is no guarantee that the driver will work without modification on all RTL8188GU / RTL8710BU devices.

When testing a wireless driver or changing network configuration, always make sure that an alternative working network connection is available when possible.

## Credits and Acknowledgements

This project is based on the work contained in the original `rtl8188gu` project by **McMCCRU** and has been adapted and tested for modern Linux kernels.

Special thanks to:

**@McMCCRU**

for the original `rtl8188gu` repository, which provided the source code and initial support for older Linux releases.

Further kernel compatibility work and testing on Debian 13:

**Gianluigi Rosas**

### Test Platform

* **Operating System:** Debian GNU/Linux 13.7
* **Kernel:** `6.12.107+deb13-amd64`
* **Desktop:** KDE Plasma
* **Adapter:** UNICO WA2763
* **Chip:** Realtek RTL8188GU / RTL8710BU
* **USB ID:** `0BDA:B711`
* **Wi-Fi:** 802.11n, 2.4 GHz

UNICO WA2763 — Realtek Semiconductor Corp. RTL8188GU 802.11n WLAN Adapter

<img width="300" height="638" alt="Unicowa2763" src="https://github.com/user-attachments/assets/cbe73a20-87c3-43ba-82c0-2c668a5c70e1" />
