# Archlinux Installion Guide
## Pre-installation
### Acquire an installation image
Visit the [**Download**](https://archlinux.org/download/) download The ISO file
### Change the console font
To set up The console font and fontsize
```
setfont /usr/share/kbd/consolefonts/LatGrkCyr-12x22.psfu.gz
```
### Connect to the internet
To set up a network connection, just use **ip** command
```
ip link
ip link set wlan up
iwlist wlan0 scan | grep ESSID
wpa_passphrase network_ESSID paddword > network.conf
wpa_supplicant -c network.conf -i wlan0 &
dhcpcd &
ping www.baidu.com
```
### Update the system clock
Use **timedatectl** to ensure the system clock
```
timedatectl set-ntp true
```
### Partition the disks
List you disks and partition
UEFI with GPT
| Mount point | Partition | Partition type | Suggested size |
| :---: | :---: | :---: | :---: |
|/mnt/boot|/dev/efi_system_partition|EFI system partition| At least 512 MiB|
```
fdisk -l
cfdisk /dev/sda
```
