Debian 13 Test Notes
Hardware

Tested USB Wi-Fi adapter:

USB Vendor ID: 0x0BDA
USB Product ID: 0xB711
USB ID: 0BDA:B711
USB description: 802.11n WLAN Adapter
Realtek chipset family: RTL8710BU / RTL8188GU

The adapter may initially enumerate as:

0BDA:1A2B

and then switch to:

0BDA:B711

through USB mode switching.

Test Environment

The working test environment was:

OS: Debian GNU/Linux 13.7 (trixie)
Kernel: 6.12.107+deb13-amd64
Compiler: GCC 14.2.0
Architecture: x86_64

Kernel information:

6.12.107+deb13-amd64
Driver Source

The driver source used for the Debian 13 build is based on the Realtek RTL8188GU / RTL8710B driver source available from:

https://github.com/McMCCRU/rtl8188gu

The source tree contains support for the RTL8710B family and includes the RTL8710B firmware data in:

hal/rtl8710b/hal8710b_fw.c

The source contains the following firmware arrays:

array_mp_8710b_fw_ap
array_mp_8710b_fw_nic
array_mp_8710b_fw_wowlan

The NIC firmware array contains 23750 bytes.

Debian 13 / Kernel 6.12 Build

The original source required small source-level changes for the Debian 13 kernel 6.12 API.

The changes made for this repository are located in:

os_dep/linux/ioctl_cfg80211.c
os_dep/linux/usb_intf.c
os_dep/linux/ioctl_cfg80211.c

The following kernel API changes were required:

cfg80211_rtw_change_beacon() was updated to use the current struct cfg80211_ap_update interface.
Beacon data accesses were updated through the beacon member.
cfg80211_rtw_set_monitor_channel() was updated to provide the required network-device argument.
os_dep/linux/usb_intf.c

The USB shutdown callback was updated from the old nested driver structure to the current structure:

.usbdrv.driver.shutdown = rtw_dev_shutdown

These changes allow the source to compile against the Debian 13 kernel 6.12 headers.

Build Result

The driver source was successfully compiled on Debian 13 with kernel:

6.12.107+deb13-amd64

The resulting kernel module is:

8188gu.ko

The module reports the driver version:

v5.2.20.2_28373.20180619

The module contains an USB alias for:

0BDA:B711

The module is an out-of-tree kernel module and therefore is not signed by the Debian kernel build system.

Firmware Configuration

The source configuration enables the RTL8710B target:

CONFIG_RTL8710B

The driver source also enables embedded firmware:

CONFIG_EMBEDDED_FWIMG 1

and:

LOAD_FW_HEADER_FROM_DRIVER

The external firmware-file option is not enabled in this configuration:

CONFIG_FILE_FWIMG

is disabled.

The RTL8710B firmware data is therefore present in the driver source itself.

USB Detection

The adapter can use USB mode switching.

The initial USB device may appear as:

0BDA:1A2B

After USB mode switching it appears as:

0BDA:B711

The final device identifies itself as a Realtek:

802.11n WLAN Adapter

The adapter uses USB 2.0 and exposes a vendor-specific USB interface.

Kernel Driver Detection

The Debian kernel also contains the in-tree rtl8xxxu driver, which has an USB ID entry for:

0BDA:B711

For this reason, systems containing both 8188gu and rtl8xxxu may associate the adapter with either driver depending on driver binding and probe order.

This repository does not assume that every successful Wi-Fi test was performed exclusively by the custom 8188gu module.

When verifying which driver is actually handling the adapter, check:

readlink -f /sys/class/net/<interface>/device/driver

and:

ethtool -i <interface>

when the wireless interface is present.

Wireless Interface

During testing, the adapter created a wireless interface named:

wlx90de806b12e1

The interface operated in managed mode and reported an active wireless network.

A successful NetworkManager Wi-Fi connection was also observed during testing.

The wireless network name, password, MAC address and other private connection information are intentionally not included in this document.

NetworkManager Test

A wireless connection was tested with NetworkManager using:

nmcli --ask device wifi connect "<SSID>" ifname <interface>

NetworkManager reported successful activation of the wireless device.

The actual SSID and credentials are intentionally omitted from this public repository.

Firmware Loading Observation

During one of the kernel probes, the in-tree rtl8xxxu driver reported:

RTL8710BU rev A (SMIC)

and loaded:

rtlwifi/rtl8710bufw_SMIC.bin

The firmware reported:

Firmware revision 16.0

This observation is documented separately from the custom 8188gu build.

It demonstrates that the Debian kernel's rtl8xxxu driver also has support for this USB ID and RTL8710BU device.

Important Driver Distinction

There are two separate facts that should not be confused:

The custom RTL8188GU / RTL8710B source in this repository was successfully adapted and compiled against Debian 13 kernel 6.12.
The USB adapter was successfully detected and used for a wireless connection during testing.

The kernel logs from the documented test session also show rtl8xxxu probing the device and loading the external SMIC firmware.

Therefore, this document does not claim that the recorded NetworkManager connection was exclusively provided by the custom 8188gu module.

A future test can establish this conclusively by verifying the bound kernel driver with:

ethtool -i <interface>

or:

readlink -f /sys/class/net/<interface>/device/driver
Reproducing the Build

Clone or obtain the source tree and enter the driver directory.

Build against the currently running kernel:

make -C /lib/modules/$(uname -r)/build M=$PWD modules

If the source tree is configured for the target kernel, the resulting module should be:

8188gu.ko

Check the module information with:

modinfo ./8188gu.ko

Verify that the module contains the expected USB alias:

modinfo ./8188gu.ko | grep -i 0BDA
Installing the Module

For a local test installation:

sudo install -D -m 0644 8188gu.ko \
  /lib/modules/$(uname -r)/kernel/drivers/net/wireless/8188gu.ko

Then update the module dependency database:

sudo depmod -a

Load the module:

sudo modprobe 8188gu

After connecting the adapter, inspect:

ip link

and:

iw dev

If iw is not installed, install the Debian package providing it before performing the wireless-interface checks.

Checking the Active Driver

When the wireless interface exists, replace <interface> with its actual name:

ethtool -i <interface>

Also check the driver symlink:

readlink -f /sys/class/net/<interface>/device/driver

These checks are important when both 8188gu and rtl8xxxu are available on the system.

USB Mode Switching

If the adapter initially appears as:

0BDA:1A2B

check the USB device with:

lsusb

After USB mode switching, the expected WLAN device is:

0BDA:B711

Check the USB topology with:

lsusb -t
Test Summary

The following points were verified on Debian 13:

Item	Result
Debian 13.7	Tested
Kernel 6.12.107	Tested
RTL8710BU / RTL8188GU USB adapter	Detected
USB ID 0BDA:B711	Detected
USB mode switching 0BDA:1A2B → 0BDA:B711	Observed
Custom 8188gu source	Compiled
Kernel 6.12 API adjustments	Applied
8188gu.ko generated	Yes
RTL8710B embedded firmware	Present in source
Wireless interface	Observed
NetworkManager Wi-Fi connection	Successfully tested
Exclusive attribution of that connection to 8188gu	Not established
Privacy

No Wi-Fi password, NetworkManager secret, personal network credentials, MAC address, USB serial number, or other private connection information should be committed to this repository.

Reproducibility

The goal of this document is to provide enough information for another Debian 13 user with the same RTL8188GU / RTL8710BU USB adapter to reproduce the build and independently verify which kernel driver is actually handling the device.

For a definitive custom-driver test, verify the active driver before and after loading 8188gu and record the output of:

ethtool -i <interface>

and:

readlink -f /sys/class/net/<interface>/device/driver

along with the relevant kernel messages:

dmesg | grep -iE '8188|8710|rtl8xxxu|firmware|wlan'
