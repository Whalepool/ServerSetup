# Whalepool Arch Linux Server Setup  


#### With windows dual boot ? 
If you're planning on a dual boot, first install windows. Make sure it is matching the arch you're going to install (either both bios or UEFI).  
If when you're making your Windows USB, if possible to choose I would suggest GPT (not MBR) and UEFI (if supported by your bios)   
After installing windows resize the disk by right clicking the 'start' button, going to 'disk management' then clicking the large partition then 'shrink'. 

Boot the arch linux USB.   
  
--- 

# Disks & Partitions  
  
- Theory, we want to create a normal 'boot' area, then a fully encrypted main '/' area.   
- Assuming we have  no windows (windows usually adds 1-3 partitions so just increase numbers bellow accordingly).     
- You can use some cools like `lsblk -f`, `fdisk -l` to help you in this part.   
- Assumming our main block is /dev/sda  
    
We want basically create 2 (3 if on UEFI) partitions.   
1. (UEFI ?) - 500M, EFI filesystem, code EF00   
2. Boot   -  100M, Linux filesystem   
3. Root (/) - Rest of disk, Linux filesystem   

```bash 
# create partitions 
cfdisk /dev/sda 
``` 

You will end up with something that looks like this ... 
```bash
Device  |  Start |  End | Sectors | Size | Type 
# .. blah blah
# maybe some windows partitions here if dual booting 
/dev/sda1 | xxx | xxx | xxx | 100M | EFI System 
/dev/sda2 | xxx | xxx | xxx | 500M | Linux filesystem 
/dev/sda3 | xxx | xxx | xxx | 90G | Linux filesystem 
```
  
Now we want format them.    
```bash
# if doing UEFI
mkfs.vfat -F 32 /dev/sda1
# For t he normal boot partition 
mkfs.ext2 /dev/sda2
# Leave 3 for now..
```  
  
Turn the root (/) into an encrypted disk   
```bash   
# Initialize the LUKS partition  
# Itertime is how long the hashing takes, slower pcs, slower hasing, i like around 5000-50,000 depending on machine 
cryptsetup luksFormat --iter-time 5000 /dev/sda3   
# Open her up
cryptsetup open /dev/sda3 crypt_root   
# Format her   
mkfs.ext4 /dev/mapper/crypt_root   
```   
   
Thats the paritioning & formatting sorted. Easy, now we mount it all.   
```bash 
# mount the encrypted disk to /mnt 
mount /dev/mapper/crypt_root /mnt
# Make a boot folder
mkdir /mnt/boot
# Mount our unencrypted linux file system to the boot folder we just made 
mount /dev/sda2 /mnt/boot
# FOR UEFI, lets get that in aswell
mkdir /mnt/boot/efi 
mount /dev/sda1 /mnt/boot/efi   
```   
   
# Pacstrap it up  

IF you're on UEFI you'll need `efibootmgr` and `os-prober` 
```bash
# Obviously you can go 'core' while ur testing messing around..   
pacstrap /mnt base base-devel   
# Or, you can go full blown, cuz you know this shits going to work. 
pacstrap /mnt base base-devel git zsh vim gptfdisk grub xorg xorg-xinit i3-wm demnu i3status i3lock deluge gimp nano maim teamspeak3 telegram-desktop htop curl wget unzip termite nfs-utils ntfs-3g efibootmgr os-prober openvpn alsa-utils pavucontrol pulseaudio-alsa qtox 
# ... I'm sure  you can thin kof more.  
# Bonus packages you may want
git mlocate keepassxc ttj-dejavu openssh nemo eog bdf-unifont ttf-croscore brasero ffmpeg npm nodejs filezilla audacious 

```  
Once this is done I recommend you mount any other servers or disks while you're here...  
```bash
# You will need to make the mount folders aswell... 
mount -t ntfs-3g /dev/sdaX /mnt/mnt/windows/disk/mount/point 
```

Generate the fstab config file 
```bash   
genfstab -p -U /mnt > /mnt/etc/fstab
## Aftewards append to fstab:   
/swapfile none swap defaults 0 0
```   
  
# Login to the system we're building 
```bash
arch-chroot /mnt   
# Set the time
hwclock --systohc --utc 
# Uncomment the locale you want in 
vim /etc/locale.gen 
# Set the LANG variable:
vim /etc/locale.conf 
LANG=en_US.UTF-8   # for example ..  
#  

``` 
  
# Internet connection 
If you're on wifi, I recommend this for you   
```bash
iw dev
ip link set interface up
ip link set interface up
iw dev interface scan | less
wpa_passphrase MYSSID passphrase > /etc/wpa_supplicant/example.conf  
wpa_supplicant -B -i interface -c /etc/wpa_supplicant/example.conf
dhcpcd interface
```
  
IF you're wired in, do this...  
```bash 
# Get your ethernet device name
ip a 
# Set a system job to start that device on boot
systemctl enable dhcpcd@DEVICENAME.service
```   
  
# While still arch-chroot in.. 
Setup a swap file  
```bash  
fallocate -l 512M /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile   
``` 

Make sure we have the 'encrypt' module loaded in mkinitcpo 
```bash
vim /etc/mkinitcpo.conf
#  Make sure the HOOKS line has 'encrypt' in it, like this..
HOOKS="... block encrypt filesystems .. etc"
```  
  
Creat the ramrisk envionment 
```bash 
mkinitcpio -p linux  
``` 

Once this is done, install grub, make config,  and reboot.. 
```bash 
# Normal
grub-insall 
# on UEFI 
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
# make the grub config
grub-mkconfig -o /boot/grub/grub.cfg   
# Edit the grub cfg file to make sure it includes the cryptdevice ... 
vim /boot/grub/grub.cfg 
``` 
Now, where it looks like this. ..   
`linux   /vmlinuz-linux root=UUID=xxx-xxx-xxxx-xxx rw  quiet`   
Instead, make it like this ...   
`linux   /vmlinuz-linux root=/dev/mapper/crypt_root cryptdevice=/dev/sda3:crypt_root rw  quiet`   
  
# Reboot.   
Ignore the fact that windows isn't booting if you're dual booting.  
If you're not dual booting, great, you're basically there, all should be good.   
  
Now, if you're dual booting...  you'll want to login as root then..   
```bash
# I recommend mount every single windows partition, because I have had issues with os-prober.. 
# anyway, get them mounted, then run ..  
os-prober 
# this should find your windows os. .  
# then remake the grub.cfg 
grub-mkconfig -o /boot/grub/grub.cfg 
# then edit it again to put in the cryptdevice bit.. 
vim /boot/grub/grub.cfg   
``` 

Now reboot, and your windows option should be there.   
  
# i3-wm   
If you're read to startx, then when you login to the terminal with your username, I recommend the following..  
```bash 
echo "xset s off" > .xinitrc
echo "xset s noblank" > .xinitrc
echo "xset -dpms" > .xinitrc
echo "exec i3" > .xinitrc
start x 
```

You should now be in i3.  Take it from here. 




## update pacman repos
pacman -Syy

## oh-my-zsh 
```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
# 
vim ~/.zshrc  # - add in 'archlinux, zsh-autosuggestions zsh-syntax-highlighting' plugins with git 

# Theme: norm
```

### Install sublime
https://www.sublimetext.com/docs/3/linux_repositories.html#pacman




