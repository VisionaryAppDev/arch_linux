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

# Based Installation
base vs base-devel:
 - https://www.archlinux.org/groups/x86_64/base/
 - https://www.archlinux.org/groups/x86_64/base-devel/

 Update Mirror 
```
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
echo "Server = http://mirror.xtom.com/archlinux/\$repo/os/\$arch"  > /etc/pacman.d/mirrorlist
```

 Installing
```
pacstrap /mnt base base-devel linux linux-firmware vim
genfstab -U /mnt > /mnt/etc/fstab
```

*Note# If error related to core.db, ....db then check its content. If the content is full of html, then there is something block the connection. This happen especially at GDCE


# Real OS
 Basic Configuration
 ```
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/Asia/Phnom_Penh /etc/localtime
hwclock --systohc
locale-gen
echo "yourpc" >> /etc/hostname
echo -e "127.0.0.1\tlocalhost" > /etc/hosts
echo -e "::1\t\tlocalhost" >> /etc/hosts
echo -e "127.0.1.1\cc.localdomain cc" >> /etc/hosts
```

 User User and Password
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
pacman -S grub efibootmgr grub-customizer # this customerize is interested since it is allow you to select any default kernel to boot by default
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
GRUB_SAVEDEFAULT="false"
GRUB_DISABLE_OS_PROBER="true"
GRUB_TIMEOUT_STYLE="menu"
GRUB_TERMINAL_INPUT="console"
GRUB_GFXMODE="auto"
```



 Change Boot Order 
Please check /dev/sdaX accordingly 

# If access from arch-chroot
mount /dev/sda4    /mnt
mount /dev/sda3    /mnt/boot/efi

# If access from Arch linux
mount /dev/sda3 /boot/efi

# Print out all available boot menu and copy the order your want. eg, 0003 or 0004
efibootmgr

# Delete any boot menu your don't want at
sudo rm -rf /boot/efi/EFI/boot-name

# Change default boot to your fav eg "0003" that you got from `sudo efibootmgr` 
# Doesn't seem to work
#echo "0003" > /boot/efi/BOOTNXT

# Alternative Solution: /etc/systemd/system/my-auto-select-boot-menu.service
```
[Unit]
Description=My Command

[Service]
ExecStart=efibootmgr -o 0003 -n 0003
Type=simple

[Install]
WantedBy=multi-user.target
```


# Restart to take effect

 Basic Package
Check this page for some group of plasma. `https://wiki.archlinux.org/title/KDE`. 

```
pacman -Syu telegram-desktop kcolorpicker ark lrzip lzop p7zip unarchiver unrar tar
pacman -Syu dhclient tor npm vlc git xorg plasma dolphin netctl redshift transmission-gtk wget bash-completion ntfs-3g curl eog 
pacman -Syu linux-lts-headers linux-lts  bleachbit konsole
pacman -Syu tree ufw spectacle simplescreenrecorder ripgrep gparted eog

# Update firewall
```
sudo ufw default deny
sudo ufw allow from 192.168.0.0/24
sudo ufw allow from 192.168.0.0/16
sudo ufw limit ssh
sudo ufw enable
````

# INSTALL VIA YAY
yay -Syu firefox-nighty google-chrome-dev intellij-idea-ue-eap jdk-openjdk otf-fira-code ttf-symbola


# Wifi 
sudo pacman -S wpa_supplicant wireless_tools networkmanager
sudo pacman -S modemmanager mobile-broadband-provider-info usb_modeswitch


# not sure what it is but know it is UI # sudo pacman -S nm-connection-editor network-manager-applet
sudo systemctl enable NetworkManager.service
sudo systemctl enable wpa_supplicant.service
sudo systemctl disable dhcpcd.service


# Add netspeed from KDE Widget
```

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
TotalDownload
CheckSpace
UseDelta     = 0.7
VerbosePkgLists
RemoteFileSigLevel = Required
" | sudo tee -a /etc/pacman.conf
```



 Change grub delay
```
sudo vim /etc/default/grub => GRUB_FORCE_HIDDEN_MENU="true"
pacman -S ufw && ufw enable && ufw status verbos && systemctl enable ufw.service thermald xf86-input-libinput
pacman-optimize samsung_magician
```


 Postgres Installation
```
# Installing PSQL: https://www.netarky.com/programming/arch_linux/Arch_Linux_PostgreSQL_database_setup.html
pacman -Syu postgresql

# Before you can do anything, you must initialize a database storage area (cluster) on disk. In file system terms, a database cluster is a single directory under which all data is stored. It is completely up to you where you choose to store your data. There is no default, although locations such as /usr/local/pgsql/data or /var/lib/postgres/data are popular.
sudo mkdir /var/lib/postgres/data

# Change the owner of the /var/lib/postgres directory and its contents to the postgres user (the default user set up by the install):
sudo chown -c -R postgres:postgres /var/lib/postgres

# To initialize a database cluster, use the command initdb, which is installed with PostgreSQL. This must be done as the postgres user, so become this user:
sudo -i -u postgres
initdb -D '/var/lib/postgres/data' # this one is in postgres console

# Start service
sudo systemctl start postgresql

# PostgreSQL is now running. By creating another PostgreSQL user as per your local Arch user ($USER), you can access the PostgreSQL database shell directly instead of having to log in as the postgres user:
createuser -s -U postgres --interactive # after enter your pc username

createdb myDatabaseName
psql -d myDatabaseName
\du


# Allow access from anywhere
sudo echo 'host    all             all              0.0.0.0/0' >> /var/lib/postgres/data/hba_file.conf
sudo echo "listen_addresses = '*'" >> /var/lib/postgres/data/postgresql.conf


# mount opt from home
echo "/home/yourpc/app/opt /opt none bind 0 0" >> /etc/fstab

systemctl enable postgresql.service
```


 Mariadb Installation 
```
sudo pacman -S mariadb
systemctl enable mysqld.service
```


# Starting Service
systemctl enable thermald.service




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


