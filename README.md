# Poseidon

Poseidon is my personal arch configuration, this repository contains some general instructions and configuration file to install it
##### Note: I have a dual boot, so windows must be installed first, and a windows EFI system partition must exist.
## Setting up arch
### 1.- Make a bootable USB containing arch
You can install any .iso file from [Arch linux download page](https://archlinux.org/download/)

Now plug the bootable USB to your computer, start from BIOS, and select it as the first device to boot. Reboot the computer to boot from USB.

Select the first option to install x86_64 arch
### 2.- Connect to wifi
```bash
# iwctl
[iwd]# device list
```
Take note of your device used to connect to wifi, it is usually called wlan0

Now, get the name of all the networks to which you can connect
```bash
[iwd]# station wlan0 get-networks
```
Now, connect to the network, it will ask you for the password after running the next command, only add it.
```bash
[iwd]# station wlan0 connect <SSID>
``` 
### 3.- Create partitions
Use fdisk to create the following partitions:
Type | Size 
:---: | :---:
EFI filsesystem | 512M
Swap | Depends on RAM
Linux filesystem | The amount you want to give to arch

### 4.- Configure partitions
```bash
# mkfs.ext4 /dev/<disk partition containing linux filesystem>
# mkfs.fat -F32 /dev/<disk partition containing EFI filesystem>
```

### 5.- Mount partitions
```bash
# mount /dev/<disk partition containing linux filesystem> /mnt
# mkdir /mnt/boot
# mount /dev/<disk partition containing EFI filesystem> /mnt/boot
```
### 6.- Install and setup os packages
```bash
# pacstrap /mnt base linux linux-firmware
# genfstab -U /mnt >> /mnt/etc/fstab
# arch-chroot /mnt
$ mkswap /dev/<disk partition containing swap>
$ swapon /dev/<disk partition containing swap>
```
### 7.- Install vim and configure files
```bash
$ pacman -S vim
$ vim /etc/fstab
```
Now, add the following line at the end of your file:
```
/dev/<disk partition containing swap> none swap defaults 0 0
```
And exit vim

Now, configure clock
```bash
$ ln -sf /usr/share/zoneinfo/<continent>/<timezone> /etc/localtime
$ hwclock --systohc
```
Replace <continent> and <timezone> with your information.

Now, run
```bash
$ vim /etc/locale.gen
```
And scroll down until you see
```
#en_US.UTF-8 UTF-8
```
And uncomment it.

Now, run
```bash
$ echo "LANG=en_US.UTF-8" >> /etc/locale.conf
$ echo poseidon > /etc/hostname
$ vim /etc/hosts
```
Add the following information to the file:
```
127.0.0.1       localhost
::1             localhost
127.0.1.1       poseidon.localdomain     poseidon
```

### 8.- Add users and create dual boot
Add a password to root user
```bash
$ passwd
```
Install packages needed for dual boot
```bash
$ pacman -S grub efibootmgr os-prober
```
Install extra packages you may need
```bash
$ pacman -S networkmanager
```
Mount the EFI system partition you SHOULD HAVE from windows into a directory.
```bash
$ mount /dev/<EFI filesystem windows created> /mnt2
```
We are ready to configure grub
```bash
$ grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
$ vim /etc/default/grub
```
Uncomment the last line, which says
```
GRUB_DISABLE_OS_PROBER=false
```
Continue configuration
```bash
$ grub-mkconfig -o /boot/grub/grub.cfg
```
If you installed network manager, run
```bash
$ systemctl enable NetworkManager
```
Now add your user
```bash
$ useradd -mG wheel <your_user>
$ passwd <your_user>
```
### 9.- Exit and reboot
```bash
$ exit
$ umount -a
$ reboot
```
Hopefully, everything goes ok and dual boot is correctly setted up.


## Personalizing arch, installing GUI and applications
Login with root user
### 1.- Configure internet again
If you installed and enabled networkmanager, this should be easy
```bash
$ nmcli r wifi on                                       # Usually not necessary, turn device's wifi on
$ nmcli d wifi list                                     # Check the exact name of your network
$ nmcli d nwifi connect <SSID> password <your_password> # Connect to your wifi
```
### 2.- Convert your user to superuser
```bash
$ pacman -Syu
$ pacman -S sudo
$ vim /etc/sudoers
```
Now, uncommment the line which says
```
#%wheel ALL=(ALL:ALL) ALL
```
And add
```
<your_user> ALL=(ALL:ALL) ALL
```
below the line that says
```
root ALL=(ALL:ALL) ALL
```
Reboot and login into your user
```bash
$ reboot
```
### 3.- Install and configure display manager, terminal, and window manager
 
Display manager             | lightdm
--- | ---
Display manager greeter     | lightdm-gtk-greeter
Window manager              | Qtile
Terminal                    | Alacritty
File manager                | Thunar
Computer theme              | Adwaita-dark

Adwaita-dark comes with gnome-themes-extra

```bash
$ sudo pacman -S lightdm lightdm-gtk-greeter alacritty qtile dmenu lxappearance thunar gnome-themes-extra
$ sudo systemctl enable lightdm
$ reboot
``` 

If everything goes right, proceed with the next step. If lightdm fails, press ctrl+alt+F2
and run 
```bash
$ sudo lightdm --test-mode --debug 
```
Install needed packages that error log tells you

### 4.- Configure colors, alacritty, qtile and lightdm
Copy the contents of the next files from this repo to your computer:

~/.xprofile

~/.config/qtile/config.py

~/.config/alacritty/alacritty.yml

~/Desktop/wallpapers/macross.png

~/Desktop/wallpapers/background.png
```bash
$ reboot
```
Now, open lxappearance, go to "Widget" and select Adwaita-dark
###### .xprofile and .config/qtile/config.py contain specifications of my computer, as the configuration of two monitors, so you may need to change them

### 5.- Install and configure used apps
```bash
$ sudo pacman -S firefox
$ sudo pacman -S noto-fonts-cjk noto-fonts-emoji noto-fonts # Japanese chars
$ sudo pacman -S git
$ mkdir ~/vscode && cd ~/vscode
$ git clone https://aur.archlinux.org/visual-studio-code-bin.git
$ cd visual-studio-code-bin
$ makepkg -si
$ cd ../..
$ rm -r vscode
$ sudo pacman -S virtualbox # install virtualbox-host-modules
$ sudo pacman -S linux linux-headers
```
