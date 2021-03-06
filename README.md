# DISCLAIMER

This guide is **ONLY** for EFI systems. Although a large part of the process is the same.  
**I am not responsible for any damages, loss of data, system corruption, or any
other mishap you may somehow cause by following this guide.**

This is mainly a step-by-step reminder/log for myself of how I installed Arch on
my laptop. I am putting this out there in case it is useful for someone else, it
is **not** intended to be an official guide. As a result, you may find that this
guide is very tedious or lists a lot of unnecessary/intuitive steps or just
straight up does things in a way that is considered bad practice. Apart from the
latter, this is intentional.

I will try, but I cannot promise that I will keep this guide up-to-date with the
the Arch Wiki. **If you are reading this a couple of months down the line and I
have not updated it in a while, you ~~might~~ will want to check with/refer to
the Arch Wiki either completely or at least in parallel to this guide as
settings/commands/recommendations might have changed.**

# Why should I even install Arch manually when archinstall is a thing?
I'll admit that the idea of a competent install script for Arch sounds very tempting, archinstall is not that(yet). Aside from very few options to work with, *very* poor default partitioning options and just generally frequent crashes when you even try to do something that is remotely hacky makes it a very bad option as of now. Hopefully it improves but for now manual is the way. 

# Preinstallation

## BIOS update

Make sure that the latest BIOS update is installed. You can usually find this on your laptop/motherboard manufacturer's website.

## BIOS settings

Make sure you have Secure Boot disabled and Virtualisation(If you plan on running VMs) enabled.  
They should be pretty easy to find. You can always Google where it is on your specific machine but it's usually in a somewhat relevant category like Security.  
On my test rig(ThinkPad T460) it can be found under:

- Security > Secure Boot > Secure Boot > Disabled
- Security > Virtualization > Intel (R) Virtualization Technology > Enabled

## Create the Arch installer USB

Download the ISO file and flash it on to a USB stick or any other bootable medium(Using something like [Balena Etcher](https://github.com/balena-io/etcher))
For more information you can refer to the ArchWiki [here](https://wiki.archlinux.org/index.php/USB_flash_installation_media)

# Main installation

## Boot from the Arch install USB

Interrupt the boot sequence(usually you just need to press the `Esc`, `F10`, or `Enter` key and navigate to the appropriate option) and boot into the USB we just flashed.

## Connect to the internet

An internet connection is required for installation. Plug in an ethernet cable(or connect to your WiFi using the `iwd` refer to the [ArchWiki](https://wiki.archlinux.org/index.php/Iwd)) and make sure you are connected to the internet:

```
ping google.com
```

## Update the system clock

```
timedatectl set-ntp true
```

## Partition the disks

<h5> Note: If you are installing Arch on an NVME drive you might see that your partitions are named like nvme0nX instead of sdaX. </h5><br>

Use cfdisk to make your partitions. You will need an EFI partition at the start of your disk(sda1) of at least 500Mb. Set its type as ESP.  
Allocate the rest of the drive to your root partition(I know, I know seperate home drive but it isn't _required_ to have a functional machine). Set it to primary and ext4 and select write to write changes to the drive(Don't forget to mark it with the boot flag!). Now select quit.

## Format the boot partition

```
mkfs.fat -F32 /dev/sda1
```

## Format the root partition

```
mkfs.ext4 /dev/sda2
```

## Mount the file systems

```
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

## Install the base packages

```
pacstrap /mnt base base-devel linux linux-firmware nano networkmanager
```

## Fstab

```
genfstab -U /mnt >> /mnt/etc/fstab
```

<h5> Note: Check the resulting file in `/mnt/etc/fstab` and make sure it covers boot and root.</h5><br>

## Chroot

```
arch-chroot /mnt
```

## Time zone

<h5> Note: Change the location as per your needs.</h5><br>

```
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
hwclock --systohc
```

## Locale

Uncomment `en_US.UTF-8 UTF-8` in `/etc/locale.gen,` then

```
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
```

## Hostname

<h5> Note: Change the hostname(Zensho in my case) to suit your needs</h5><br>

```
echo Zensho > /etc/hostname
```

In `/etc/hosts`, add:

```
127.0.0.1	localhost
::1		localhost
127.0.1.1	Zensho.localdomain Zensho
```

## Root password

```
passwd
```

## Configure mkinitcpio and create the initramfs image

In `/etc/mkinitcpio.conf`, the hooks must be:

```
HOOKS=(base udev autodetect keyboard keymap consolefont modconf block encrypt filesystems fsck)
```

Then run

```
mkinitcpio -P
```

## Boot loader and Intel microcode

Install and configure GRUB:

```
pacman -S grub efibootmgr
```

Add Microcode:  
`pacman -S intel-ucode`(only for Intel devices)  
`pacman -S amd-ucode`(only for AMD devices)

```
grub-mkconfig -o /boot/grub/grub.cfg
```

## Add user

<h5> Note: Replace the user name(In my case kami) with whatever you want</h5><br>

```
useradd -m kami
passwd kami
```

Add kami to sudoers:

```
pacman -S vim
visudo
```

1. Go to the line starting with "root".
2. Press `Y` twice to yank it.
3. Go to the next line and press `P` to paste it.
4. Use `X` to delete "root" from that line.
5. Press `I` to enter insert mode, and replace the deleted "root" by "kami".
6. Press `Esc`, then type ":wq", then press `Enter`.

## Reboot

```
exit
umount -R /mnt
shutdown now
```

Remove the Arch installer USB and power the computer back on.

## Internet

Find the wireless interface name (e.g., wlp4s0):

```
ip link
```

Enable the wireless interface:

```
sudo ip link set wlp4s0 up
```

NetworkManager will be used to manage connections:

```
sudo pacman -S networkmanager
sudo systemctl enable NetworkManager.service
sudo systemctl start NetworkManager.service
sudo systemctl mask systemd-resolved.service
```

Remove the `/etc/resolv.conf` file (if it exists), then:

```
sudo echo "nameserver 8.8.8.8" > /etc/resolv.conf
```

In `/etc/NetworkManager/NetworkManager.conf`, under the [main] section (create it if it does not exist) add:

```
dns=none
```

## yay

Install yay, an AUR helper:

```
cd ~
sudo pacman -S --asdeps go
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
cd ..
rm -rf yay
```

## Fonts

To cover most characters:

```
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-extra ttf-dejavu ttf-liberation
```

<h5> Note: noto-fonts-emoji is not part of the installation.</h5><br>

## Sound

```
sudo pacman -S pulseaudio pulseaudio-alsa
```

## Touchpad

```
sudo pacman -S xf86-input-libinput
```

###### Note: Put the backup file /etc/X11/xorg.conf.d/10-touchpad.conf

## Man pages

```
sudo pacman -S man-db man-pages
```

## X

### Basic X packages:

```
sudo pacman -S xorg-server xorg-xinit xorg-xrdb
```

## Desktop Enviroment(KDE Plasma)

```
sudo pacman -S plasma-meta
```

##### Note: Feel free to install any other DE(Like GNOME, i3, Xfce ,etc)

# Post-installation

## Firefox(or LibreWolf)

```
sudo pacman -S firefox
```

## TLP(Desktop users can skip this)

```
sudo pacman -S tlp
sudo pacman -S --asdeps acpi_call ethtool smartmontools x86_energy_perf_policy
sudo systemctl enable tlp.service
sudo systemctl mask systemd-rfkill.service
sudo systemctl mask systemd-rfkill.socket
```

## Okular(or Zathura)

```
sudo pacman -S okular
```

If prompted, choose phonon-qt5-vlc.

## htop

```
sudo pacman -S htop
```

## LibreOffice

```
sudo pacman -S libreoffice-still
```

## SSH

```
sudo pacman -S openssh sshpass
```

## VLC(or mpv)

```
sudo pacman -S vlc
```

## Pacman User Guide for Dummies

Installing packages:

```
sudo pacman -S [packages]
```

Updating packages:

```
sudo pacman -Syu
```

Removing and purging packages:

```
sudo pacman -Rns [packages]
```

Displaying a list of unused packages (orphans):

```
pacman -Qtdq
```

Removing unused packages (orphans):

```
sudo pacman -Rns $(pacman -Qtdq)
```

## yay

Updating packages:

```
yay -Syu --devel
```

## Misc.

Cannot write on external hard drive: https://askubuntu.com/a/172671

# Post Notes

## Disclaimers(Yes again)

Since this guide was written with the intent to act as a general guide I will not be covering many things. However all of these should be very easy to do on your own and if you're ever lost refer to ArchWiki, it has the answer to everything you're looking for.This includes:

- Fingerprint scanners
- Facial Identification
- Graphics Drivers
- The answer to life, the universe and everything
- Workarounds and fixes for niche problems that you may run into for your specific system. </br>

## Credits

This guide was based on @PhilippeOlivier guide for installing Arch on a ThinkPad, If you use a ThinkPad I suggest you use his guide instead considering it is tailormade for it. Also thanks to @CodingCellist whose disclaimer I shamelessly stole.

P.S.  
If for some unknown reason my English teacher is reading this I just want to say that I'm sorry.
