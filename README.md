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
wpa_passphrase network_ESSID password > network.conf
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
mkfs.fat -F 32 /dev/efi_system_partition
mkswap /dev/swap_partition
```
### Mount the file systems
Mount the root volume to **/mnt** 
```
mount /dev/root_partition /mnt
mount --mkdir /dev/efi_system_partition /mnt/boot
swapon /dev/swap_partition
```
## Installation
### Install essential packages
```
pacstrap /mnt base linux linux-firmware base-devel
```
## Configure the system
### Fstab
```
genfstab -U /mnt >> /mnt/etc/fstab
```
### Chroot
```
arch-chroot /mnt
```
### Time Zone
```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```
### Localization
Edit /etc/locale.gen and uncomment en_US.UTF-8, zh_CN.UTF-8
```
exit
vim /mnt/etc/locale.gen
arch-chroot /mnt
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
echo "Font=LatGrkCyr-12x22" > /etc/vconsole.conf
```
### Network configuration
```
exit
echo "myhostname" > /etc/hostname
vim /mnt/etc/hosts
127.0.0.1	localhost
::1	localhost
127.0.0.1	myhostname.localdomain myhostname
arch-chroot /mnt
```
### Root password
```
passwd
```
### Install ucode nvim and other package
```
pacman -S inter-ucode neovim zsh wpa_supplicant dhcpcd
pacman -S amd-ucode neovim zsh wpa_supplicant dhcpcd
```
### Boot loader
**Bios**
```
pacman -S grub efibootmgr os-prober
grub-install --target=i386-pc /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```
**Uefi**
```
pacman -S grub efibootmgr os-prober
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch
grub-mkconfig -o /boot/grub/grub.cfg
``` 
### Reboot
```
exit
umount -R /mnt
killall wpa_supplicant dhcpcd
reboot
```
## Post-Installation
### Chang shell
```
chsh -s /usr/bin/zsh
```
### Users and groups
```
nvim /etc/profile
export EDITOR=nvim
ln -s /usr/bin/nvim /usr/bin/vi
useradd -m -G wheel user
passwd user
visudo
uncomment %wheel ALL=(ALL) ALL
```
### Network configuration 
```
wpa_passpharase network_ESSID password > /etc/wpa_supplicant/wpa_supplicant-wlan.conf
systemctl enable wpa_supplicant@wlan0.service
systemctl enable dhcpcd
nvim /etc/systemd/network/wireless-dhcp.network
[Match]
Name=wlan0

[Network]
DHCP=yes
```
### Numlock 
```
paru -S systemd-numlockontty
sudo systemctl enable numlockontty.service
sudo pacman -S numlockx
add to .xinitrc before exec dwm
numlockx &
```
### Fonts
```
sudo pacman -S man ttf-dejavu wqy-microhei adobe-source-code-pro-fonts ttf-nerd-fonts-symbols-2048-em
```
### Paru
```
cd /opt
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si
```
### Crontab
```
sudo pacman -S cronie
systemctl enable cronie
```
### Localization
```
add to .xinitrc
export LANG=zh_CN.UTF-8
export LANGUAGE=zh_CN:en_US
```
### Display server
```
pacman -S xf86-video-intel nvidia xorg-server xorg-apps picom xorg-xinit
cd ~
nvim .xinitrc
exec dwm
mkdir Arch
cd Arch
git clone git@github.com:dwm.git
git clone git@github.com:st.git
cd dwm && sudo make clean install
cd st && sudo make clean install
startx
```
### Oh my zsh
```
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
### Chinese input
```
sudo pacman -S fcitx5-im fcitx5-chinese-addons fcitx5-material-color fcitx5-pinyin-zhwiki fcitx5-nord 
nvim /etc/enviroment
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
add to dwm scripts autostart.sh
fcitx5 -d
reboot
fcitx5-configtool
ctrl + space 
```
### Google Chrome
```
paru -S google-chrome
google-chrome-stable >/dev/null 2>&1 &
alias chrome="google-chrome-stable -force-device-scale-factor=1.2 >/dev/null 2>&1"
extensions for chrome https://github.com/FelisCatus/SwitchyOmega/releases
proxylist https://github.com/gfwlist/gfwlist
```
### Clash
```
sudo pacman -S clash
alias proxy="clash >/dev/null 2>&1 &; export http_proxy=http://127.0.0.1:7890; export https_proxy=http://127.0.0.1:7890; export all_proxy=socks5://127.0.0.1:7890"
alias unproxy="pkill clash >/dev/null 2>&1 &; unset http_proxy; unset https_proxy; unset all_proxy"
alias myip="curl -s https://httpbin.org/ip | grep origin"
59 */3 * * * wget https://remote.com/clashfile -O config.yaml
```
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
### Dwm
```
git clone git@github.com:angiane/dwm.git
```
### St
```
git clone git@github.com:angiane/st.git
```
### Dmenu
```
git clone git@github.com:angiane/dmenu.git
```
### Kvm
```
sudo pacman -S qemu libvirt iptables-nft dnsmasq bridge-utils virt-manager
sudo systemctl enable libvirtd.service
https://wiki.archlinux.org/title/Libvirt_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E8%AE%BE%E7%BD%AE%E6%8E%88%E6%9D%83
add content to file
grep kvm /etc/group
usermod -aG libvirt $USER
usermod -aG kvm $USER
```


