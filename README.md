# My Linux Post Install
### Distribution: Arch Linux

## Install:
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

# WORK IN PROGRESS









cat /etc/locale.conf
if not pt_BR, try:
echo LANG=pt_BR.UTF-8 > /etc/locale.conf
export LANG=pt_BR.UTF-8
echo "KEYMAP=br-abnt2" >> /etc/vconsole.conf
ln -s /usr/share/zoneinfo/America/Fortaleza /etc/localtime
hwclock --systohc --utc
echo "my_hostname" > /etc/hostname

### Arch Related
- pacman -S dhcpcd

If you install NetworkManager, forget this below:
- systemctl enable dhcpcd
- pacman -S dosfstools efibootmgr
- bootctl install
- EDIT /boot/loader with your configurations.
- echo "default arch" >> /boot/loader/loader.conf 


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

After, locate the file: ~/.local/share/applications/mimeapps.list
and add/change the entry:
```
inode/directory=nautilus-folder-handler.desktop
```
PS: May need a reboot or session refresh.