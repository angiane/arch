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
```
fdisk -l
cfdisk /dev/sda
```
**UEFI with GPT**
| Mount point | Partition | Partition type | Suggested size |
| :-----: | :-----: | :-----: | :-----: |
|/mnt/boot|/dev/efi_system_partition|EFI system partition| At least 512 MiB|
|[SWAP]|/dev/swap_partition|Linux swap|< 4G 2xmem, > 4G mem+2G |
|/mnt|/dev/root_partition|Linux x86-64 root (/)| Remainder of the device|  

**BIOS with MBR**
| Mount point | Partition | Partition type | Suggested size |
| :-----: | :-----: | :-----: | :-----: |
|[SWAP]|/dev/swap_partition|Linux swap|< 4G 2xmem, > 4G mem+2G |
|/mnt|/dev/root_partition|lLinux| Remainder of the device |
### Format the partitions
```
mkfs.ext4 /dev/root_partition
mkfs.fat -F 32 /dev/
```

## Installation
### Git config
```
git config --global user.name "your name"
git config --global user.email "your email"
ssh-keygen -t rsa -C "your email"
ssh -T git@github.com
git init
git remote add origin git@github.com:yourname/repository.git
git rm --cached file
git branch yourbranch
git checkout yourbranch
git merge yourbranch
git push origin branch
```
## Post-Installation



