# DISCLAIMER
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
# Preinstallation
## BIOS update
Make sure that the latest BIOS update is installed. You can usually find this on your laptop/motherboard manufacturer's website.
## BIOS settings
Make sure you have Secure Boot disabled and Virtualisation enabled.  
They should be pretty easy to find. You can always Google where it is on your specific machine but it's usually in a somewhat relevant category like Security.  
On my test rig(ThinkPad t460) it can be found under:
- Security > Secure Boot > Secure Boot > Disabled
- Security > Virtualization > Intel (R) Virtualization Technology > Enabled  
  
## Create the Arch installer USB
Download the ISO file and flash it on to a USB stick or any other bootable medium(Using something like [Balena Etcher](https://github.com/balena-io/etcher)
For more information you can refer to the ArchWiki [here](https://wiki.archlinux.org/index.php/USB_flash_installation_media)
# Main installation
## Boot from the Arch installer USB
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
