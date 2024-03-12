# Connect to Wifi network

```
# Use root
sudo su


# Option 1
wifi-menu


# Option 2
iwconfig wlp3s0 essid wifi-name
dhcpcd wlp3s0
ip link set wlp3s0 down
netctl restart wlp3s0-123456789


# WPA supplicant
sudo pacman -S wpa_supplicant dhclient dialog
cat /etc/netctl/my_static_profile
Interface=enp1s0
Connection=ethernet
IP=static
Address=('192.168.1.10/24')
Gateway=('92.168.1.1')
DNS=('192.168.1.1')
```


# Create Partition Size
Create 4 primary partitions:
- EFI 512M # Must be first partition
- Root
- Home
- Swap

*NOTE*: +512M so you don't need to specific range of start and end sector 
```
fdisk -l
fdisk /dev/sda

###########################
# =====> Causion!! <======
##########################
#
# => Format partitions:
#		mkfs.fat -F32 /dev/sda1        # 
#		mkfs.ext4 /dev/sda2            # /mnt
#		mkfs.ext4 /dev/sda3            # /home
#
# => Swap:
#		mkswap /dev/sda4
#		swapon /dev/sda4
#
# => Mount
# 		mount /dev/sda2 /mnt
#		mkdir /mnt/home
# 		mount /dev/sda3 /mnt/home
```

# Update Mirror 
```
# Option 1: select by ranking 
sudo pacman -S reflector
sudo /usr/bin/reflector --protocol https --latest 20 --sort rate --save /etc/pacman.d/mirrorlist


# Option 2: some old mirror used to use
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
echo "Server = http://mirror.xtom.com/archlinux/\$repo/os/\$arch"  > /etc/pacman.d/mirrorlist
```

# Based Installation
base vs base-devel:
 - https://www.archlinux.org/groups/x86_64/base/
 - https://www.archlinux.org/groups/x86_64/base-devel/
```
pacstrap /mnt base base-devel linux linux-firmware vim
genfstab -U /mnt > /mnt/etc/fstab
```
*Note# If error related to core.db, ....db then check its content. If the content is full of html, then there is something block the connection. This happen especially at GDCE


# Real OS
Basic Configuration

```
arch-chroot /mnt

# Time
ln -sf /usr/share/zoneinfo/Asia/Phnom_Penh /etc/localtime
hwclock --systohc

# Language
echo -e "en_US.UTF-8 UTF-8"   >> /etc/locale.gen
echo -e "km_KH UTF-8"   >> /etc/locale.gen
locale-gen


echo "yourpc" >> /etc/hostname
echo -e "127.0.0.1\tlocalhost" > /etc/hosts
echo -e "::1\t\tlocalhost" >> /etc/hosts
echo -e "127.0.1.1\tyourpc" >> /etc/hosts
```

## User User and Password
- Root Password
```
passwd 
```

- Add User
```
useradd yourpc -m yourpc
su yourpc passwd
```


 Bootloader
```
# this customerize is interested since it is allow you to select any default kernel to boot by default
pacman -S grub efibootmgr grub-customizer 


# /dev/sda1 isn't the root partition, it is the efi partition which is likely the one created by microsoft if we use dual boot. Otherwise, we need to create this partition manually 
mkdir /boot/efi
mount /dev/sda1 /boot/efi 
grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi
grub-mkconfig -o /boot/grub/grub.cfg
```

# Sample from grub-customizer
cat /etc/default/grub
```
GRUB_TIMEOUT="1"
GRUB_DEFAULT="Advanced options for Linux>Linux, with Linux linux"
GRUB_SAVEDEFAULT="true"
GRUB_TIMEOUT_STYLE="menu"
GRUB_TERMINAL_INPUT="console"
GRUB_GFXMODE="auto"

# Change grub delay
GRUB_FORCE_HIDDEN_MENU="true"
```



# Change Boot Order 
Please check /dev/sdaX accordingly 

## If access from arch-chroot
```
mount /dev/sda4    /mnt
mount /dev/sda3    /mnt/boot/efi
```

## If access from Arch linux
```
mount /dev/sda3 /boot/efi
```

## Print out all available boot menu and copy the order your want. eg, 0003 or 0004
```
efibootmgr
```

## Delete any boot menu your don't want at
```
sudo rm -rf /boot/efi/EFI/boot-name
```

## Change default boot to your fav eg "0003" that you got from `sudo efibootmgr` 
## Doesn't seem to work
#echo "0003" > /boot/efi/BOOTNXT

## Alternative Solution 
/etc/systemd/system/my-auto-select-boot-menu.service

```
[Unit]
Description=My Command

[Service]
ExecStart=efibootmgr -o 0003 -n 0003
Type=simple

[Install]
WantedBy=multi-user.target
```



# Basic Package
Check this page for some group of plasma. `https://wiki.archlinux.org/title/KDE`. 

```
pacman -Syu telegram-desktop kcolorpicker ark lrzip lzop p7zip unarchiver unrar tar
pacman -Syu dhclient tor npm vlc git xorg plasma dolphin netctl redshift transmission-gtk wget bash-completion ntfs-3g curl eog 
pacman -Syu linux-lts-headers linux-lts  bleachbit konsole kdeconnect
pacman -Syu tree ufw spectacle simplescreenrecorder ripgrep gparted eog rsync
pacman -Syu ttf-fira-code ttf-hack-nerd otf-monaspace-nerd ttf-firacode-nerd ttf-jetbrains-mono-nerd ttf-noto-nerd

# Spell check and font
pacman -Syu enchant hunspell-en_US aspell-en ttf-bitstream-vera ttf-liberation adobe-source-sans-pro-fonts ttf-dejavu 

# Codex
pacman -S flac

# Security 
pacman -Syu amd-ucode
grub-mkconfig -o /boot/grub/grub.cfg
```

# Update firewall
```
sudo ufw default deny
sudo ufw allow from 192.168.0.0/24
sudo ufw allow from 192.168.0.0/16
sudo ufw limit ssh
sudo ufw enable
````

# INSTALL VIA YAY
```
yay -Syu firefox-nighty google-chrome-dev intellij-idea-ue-eap jdk-openjdk ttf-symbola hurl
```


# Wifi 
```
sudo pacman -S wpa_supplicant wireless_tools networkmanager
sudo pacman -S modemmanager mobile-broadband-provider-info usb_modeswitch
```


# not sure what it is but know it is UI # sudo pacman -S nm-connection-editor network-manager-applet
```
sudo systemctl enable NetworkManager.service
sudo systemctl enable wpa_supplicant.service
sudo systemctl disable dhcpcd.service
```


# Add netspeed from KDE Widget
 Make makepkg build a little faster 
```
echo -e "
MAKEFLAGS="-j$(nproc)"
" | sudo tee -a /etc/makepkg.conf
```


 Yay Installation
```
cd /tmp
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si

 Enable coloring
echo -e "
\n\n\n\n
Color
CheckSpace
UseDelta     = 0.7
VerbosePkgLists
RemoteFileSigLevel = Required
ParallelDownloads = 10
" | sudo tee -a /etc/pacman.conf
```


# Some package
```
pacman -S ufw && ufw enable && ufw status verbos && systemctl enable ufw.service samsung_magician


# DNS
pacman -S systemd-resolved

# Config DNS here 
suodo vim /etc/systemd/resolved.conf

## Note do not enable DNSSEC otherwise it won't work
DNS=45.90.28.0#Acer--v15--ID-OF-NEXT-DNS.dns.nextdns.io
DNS=2a07:a8c0::#Acer--v15--ID-OF-NEXT-DNS.dns.nextdns.io
DNS=45.90.30.0#Acer--v15--ID-OF-NEXT-DNS.dns.nextdns.io
DNS=2a07:a8c1::#Acer--v15--ID-OF-NEXT-DNS.dns.nextdns.io
DNSOverTLS=yes
Cache=yes


# cleaner, alternative and more star than bleachbit
yay czkawka-gui
```


## Auto clean cache weekly
```
sudo pacman -S pacman-contrib 
sudo systemctl enable paccache.timer
```




#Nvidia
https://gist.github.com/joariasl/e58ca997d2581236dc56#install-intel-video-driver



https://wiki.archlinux.org/index.php/CPU_frequency_scaling
https://wiki.archlinux.org/index.php/Dnscrypt-proxy
https://wiki.archlinux.org/index.php/Swap_on_video_ram





pacman -R vi nano

# install after in Linux system not in live disk
os-prober


# grub-install: cannot find /boot/efi directory
# 1. run mkinitcpio -p linux
# 2. delete os-prober from live disk and reinstall install grub package
# 3. grub-install /dev/sda (make sure it is in sda not sdb)










# INSTALLING PRINTER: https://unix.stackexchange.com/questions/359531/installing-hp-printer-driver-for-arch-linux
```
# Everything is root
pacman -Sy cups
pacman -S hplip
hp-setup -i
gpasswd -a theUserNameOfPC sys
```



 PHP Installation
```
pacman -S pacman -S composer npm
yay php72 php72-fpm php72-pgsql php72-redis php72-mcrypt



echo "
extension=oci8.so
extension=ldap
extension=mysqli
extension=pdo_mysql
extension=pdo_pgsql
extension=pgsql
" | sudo tee -a /etc/php72/php.ini
# NOTE phpize72 is included in php72
```


- Sample nginx.conf 
```
server {
    listen 80;
    listen [::]:80;
    server_name company-api.test;

    root /laravel-project/public/; 
    index index.html index.htm index.php;


    location / {
       try_files $uri $uri/ /index.php?$query_string;
    }

    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;


    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php72-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).# {
        deny all;
    }
}
```






# GPU Driver
##  Mesa
Link: https://wiki.archlinux.org/title/AMDGPU
```
pacman -S mesa
```

### Accelerated video decoding
```
sudo pacman -S libva-mesa-driver   mesa-vdpau
```


## Vulkan
Link: https://wiki.archlinux.org/title/Vulkan

```
sudo pacman -S vulkan-icd-loader  


# Driver, one is required but you can try both and see which is better
# Steam require 32 bit version, so need to install it too.
sudo pacman -S vulkan-radeon amdvlk lib32-vulkan-radeon lib32-amdvlk
```

### Verification
```
ls /usr/share/vulkan/icd.d/

vulkaninfo
```

### Switching between AMD drivers
#### Installation 
```
yay amd-vulkan-prefixes
```

#### Command
- vk_radv 
- vk_amdvlk
 
```
vk_pro command
```


### GPU Monitoring
- https://github.com/Umio-Yasuno/amdgpu_top
- https://github.com/Syllo/nvtop

```
yay amdgpu_top
sudo pacman -S nvtop
```

## Steam
Link: https://wiki.archlinux.org/title/steam
- Enable multi-lib
- Install OpenGL driver ```sudo pacman -S xf86-video-amdgpu```
- If you need to add library folders or add non-Steam games to your Steam library: ```sudo pacman -S xdg-desktop-portal```
- Font: ```sudo pacman -S  ttf-liberation lib32-fontconfig ttf-liberation``` and ```yay ttf-ms-win11-auto```
