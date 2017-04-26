# My Linux Post Install
### Distribution: Arch Linux

## Arch Install:
#### Check if EFI Mode
If lists EFI variables, means its EFI Mode(or somehting compatible)
```
efivar -l
```

If not using US Keyboard, locate your keymap:
```
/usr/share/kbd/keymaps
```

Mine is ABNT2 keyboard, so here:
```
/usr/share/kbd/keymaps/i386/qwerty/br-abnt2.map.gz
```

And load it with:
```
loadkeys br-abnt2
```

With EFI, we need a "/boot" partition.
We need three partitions:
```
/boot - fat32 - 500MB (boot)
/ - (root) - ext4 (or another)
swap - linux-swap maybe 8GB.
```

To list all drivers you may use `lsblk` and find the drive to install. Mine is sda.

Find your disk size:
```
parted /dev/sda
```
Type `print`
And seek for "Disk /dev/sda:"
```
(parted) print                                                            
Modelo: ATA Hitachi HTS54501 (scsi)
Disco /dev/sda: 160GB
Tamanho do setor (lógico/físico): 512B/512B
Tabela de partições: gpt
```

### These steps we're going to partition the disk.
If unsure, you may use cfdisk, fdisk or something.

Now we need to delete all partitions or create a GPT partition table(WILL DESTROY EVERYTHING)
```
mklabel gpt
```

mkpart ESP fat32 1049kB 538MB
set 1 boot on

mkpart primary ext4 538MB [SIZE]
mkpart primary linux-swap [SIZE] 100%

In the SIZE put YOUR_HD_TOTAL_SIZE minus SWAP_SIZE
Mine I got: 160 - 8 = 152GB
So I use:
```
//your root size:
mkpart primary ext4 538MB 152GB
//your swap gonna be the last partition, so 152 - 160 (100%)
mkpart primary linux-swap 152GB 100%
```

Now type in `quit` command to exit.

Check yourt partition again with(if /dev/sda is your partition)
```
lsblk /dev/sda
```

Now type: 
mkfs.vfat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
mkswap /dev/sda3
swapon /dev/sda3

Mount your root '/' partition to a temporary location, where you gonna install the system:
```
mount /dev/sda2 /mnt
```

Mount the boot '/boot' partition:
```
mkdir -p /mnt/boot
mount /dev/sda1 /mnt/boot
```

Now install the packages:
```
pacstrap /mnt base base-devel
```

Generate fstab with:
```
genfstab -U -p /mnt >> /mnt/etc/fstab
```
May need some edit, mine is:
```
# 
# /etc/fstab: static file system information
#
# <file system>	<dir>	<type>	<options>	<dump>	<pass>
# /dev/sda2 UUID=8ba5159d-2f6a-4c17-a3ea-b27b1351a79b	/         	ext4      	rw,relatime,data=ordered	0 2

# /dev/sda1
UUID=D9B8-A21A      	/boot     	vfat      	rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro	0 1

# /dev/sda3
UUID=e2408dea-f636-4e90-944e-38960ab2e5f7	none      	swap      	defaults  	0 0

```

The last number, is the boot checking order, its a good practice put:
```
/boot - boot - fat - Order: 1
/ - root - ext4 - Order: 2
/swap - swap - Order: 0
```

If you have SSD, may be a good practice to edit ext4 partition options and put:
```
rw,relatime,data=ordered,discard
```

# WORK IN PROGRESS
# WORK IN PROGRESS
# WORK IN PROGRESS


For more info:  https://wiki.archlinux.org/index.php/Solid_State_Drives

### Configuring the base system

Need to chroot to transfer the new system
```
arch-chroot /mnt
```

And configure the locale for the new system, at the file '/etc/locale.gen' uncomment each you'd like to use.
After, regen the locale table:
```
locale-gen
```
# WORK IN PROGRESS
# WORK IN PROGRESS

```
check locale:
cat /etc/locale.conf
echo LANG=pt_BR.UTF-8 > /etc/locale.conf
export LANG=pt_BR.UTF-8
echo "KEYMAP=br-abnt2" >> /etc/vconsole.conf
```

Fix your TimeZone:
```
ln -s /usr/share/zoneinfo/America/Fortaleza /etc/localtime
```
Enable hardware clock
```
hwclock --systohc --utc
```

Create a nice hostname:
```
echo "myhostname" > /etc/hostname
```

Edit `/etc/hosts` and update the new hostname:
```
#<ip-address>	<hostname.domain.org>	<hostname>
127.0.0.1	localhost.localdomain	localhost myhostname
::1		localhost.localdomain	localhost myhostname
```

Install DHCPD but if going to install NetworkManager, be careful:
```
pacman -S dhcpcd
```

If you install NetworkManager, forget the "systemctl enable" below:
```
systemctl enable dhcpcd
```

Create admin password:
```
passwd
```

I'm not using GRUB, I use systemd-boot (was called gummiboot).

Need some tools for EFI:
```
pacman -S dosfstools
```

And install systemd-boot to the boot partition:
```
bootctl --path=/boot install
```

Now create Arch Linux boot entry, mine is:
The file name: /boot/loader/entries/arch.conf 

```
title ArchLinux
linux /vmlinuz-linux
initrd /initramfs-linux.img
options root=/dev/sda2 rw elevator=deadline nosplash resume=/dev/sda3 nmi_watchdog=0 video=SVIDEO-1:d
```

The video=SVIDEO for a fix on my old Macbook. If you have a VGA like mine `Intel Corporation Mobile GM965/GL960`, you may need this too.

Now we defined the default loader:
```
echo "default arch" >> /boot/loader/loader.conf 
```
Mine is:
```
timeout 2
default arch
#default linux-lts
```
The timeout is for faster boot. The # is a comment to avoid delete something I can use later.

If you want a LTS Kernel (Long Term Support), install `linux-lts` package and insert another creating a new file `/boot/loader/entries/linux-lts.conf`
Mine looks like this: 
```
title Linux_LTS
linux /vmlinuz-linux-lts
initrd /initramfs-linux-lts.img
options root=/dev/sda2 rw elevator=deadline nosplash resume=/dev/sda3 nmi_watchdog=0 video=SVIDEO-1:d
```

Finally, exit and reboot the system.

##### This Install guide was heavly based on a medium post witch encouraged me to run Arch on old macbook: https://medium.com/@philpl/arch-linux-running-on-my-macbook-2ea525ebefe3


# After-Install

## Creating a new user:
```
groupadd users
useradd -m -g users -G wheel -s /bin/bash new_user_name
```
Create a password:
```
passwd new_user_name
```

## Enable Sudo:
```
pacman -S sudo
edit /etc/sudoers
```
At the end of `sodoers' file, add this:
```
%wheel ALL=(ALL) ALL
```

## The Problem: Nautilus won't open rar/zip files and show "Extract here.." menu.

On Arch, must install the follow:
- extra/unrar 
- core/tar (maybe already installed)
- core/bzip2 (maybe already installed)
- core/gzip (maybe already installed)
- extra/rsync
- extra/libzip
- extra/zip
- extra/unzip
- extra/p7zip
- aur/rar (Not needed if everything works fine)

- file-roller (compress file)
- xarchiver (extract here)

## The Problem: Chrome won't open nautilus and focus the selected file when using "Show in folder" menu (pt-BR: "Abrir na pasta").

Just create file below and save at: /usr/share/applications/

File: nautilus-folder-handler.desktop

```
[Desktop Entry]
Encoding=UTF-8
_Name=Open Folder
TryExec=nautilus
Exec=nautilus --no-desktop %U
NoDisplay=true
Terminal=false
Icon=folder-open
StartupNotify=true
Type=Application
MimeType=x-directory/gnome-default-handler;x-directory/normal;inode/directory;application/x-gnome-saved-search;
X-GNOME-Bugzilla-Bugzilla=GNOME
X-GNOME-Bugzilla-Product=nautilus
X-GNOME-Bugzilla-Component=general
```

After, locate the file `~/.local/share/applications/mimeapps.list`
and add/change the "inode" entry:
```
inode/directory=nautilus-folder-handler.desktop
```
PS: May need a reboot or session refresh.