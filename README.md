# Thinkpad x1 carbon 6th gen withh Ubuntu (18.04)

## Fix trackpad
- Disable whilst typing. Add the following to `~/.bash_profile` or startup applications:

```
SYNAPTICS_ID=`xinput | grep "Synaptics TM3288-011" | sed -n 's/.*id=\([0-9]*\).*/\1/p'`
xinput set-prop $SYNAPTICS_ID "Synaptics Palm Detection" 1
xinput set-prop $SYNAPTICS_ID "Synaptics Palm Dimensions" 3, 3
```
Check for properties currently set:
```
xinput watch-props `xinput | grep "Synaptics TM3288-011" | sed -n 's/.*id=\([0-9]*\).*/\1/p'`
```

## Fix multitouch on apple magic trackpad 2

## Fix bluetooth headset issues (after suspend)

- Install newer BlueZ

```
sudo add-apt-repository ppa:bluetooth/bluez
sudo apt-get install bluez
sudo /etc/init.d/bluetooth restart
````

- Install pulseaudio control

```
sudo apt-get install pavucontrol
```

- Set pulse audio to use high-fidelity: Open pavucontrol. On Configuration tab select A2DP for the connected headset.

## Fixing S3 suspend

See original patch [here](https://delta-xi.net/#056)

### If you're feeling lazy:

1. Configure the BIOS:
    - Config > Thunderbolt (TM) 3 > Thunderbolt BIOS Assist Mode > Enabled
    - Security > Secure Boot must > Disabled
2. Copy [overrride_acpi](suspend/override_acpi) to `/boot`
3. Change this line in `/etc/grub.d/10_linux` (See changed file [10_linux](suspend/10_linux))
    - `initrd	${rel_dirname}/${initrd}`
    
    to read
    - `initrd	${rel_dirname}/acpi_override ${rel_dirname}/${initrd}`
4. Change this line in `/etc/default/grub` (See changed file [grub](grub))
    - `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash psmouse.synaptics_intertouch=1"`
    
    to read
    - `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash psmouse.synaptics_intertouch=1 mem_sleep_default=deep"`
5. Update grub
    - `sudo update-grub2`
6. Reboot



### Manual steps:
1. Configure the BIOS:
    - Config > Thunderbolt (TM) 3 > Thunderbolt BIOS Assist Mode > Enabled
    - Security > Secure Boot must > Disabled
2. Install needed software:
    - `sudo apt-get install iasl cpio`
3. Dump acpi dstd table (see mine here [dsdt.aml](dstd.aml)):
    - `sudo cat /sys/firmware/acpi/tables/DSDT > dsdt.aml`
4. Decompile:
    - `iasl -d dsdt.aml`
5. Patch it with [dsdt.patch](suspend/dstd.patch) ([original link](https://delta-xi.net/download/X1C6_S3_DSDT.patch)). Or use mine [dstd.dsl](suspend/dstd.dsl)
    - `patch --verbose < dstd.patch`
6. Compiled patched file:
    - `iasl -ve -tc dsdt.dsl`
7. Create CPIO archive
    - `mkdir -p kernel/firmware/acpi`
    - `cp dsdt.aml kernel/firmware/acpi`
    - `find kernel | cpio -H newc --create > acpi_override`
    - `sudo cp acpi_override /boot`
8. (Optionally) Try out the new image
    - Reboot
    - In grub menu press `e` on the first entry
    - edit the line that reads `initrd...` to read `initrd /boot/override_acpi...`
    - Boot
    - Once booted, check with `cat /sys/power/mem_sleep` that you get `[s2idle] deep` (deep will be the default after the next step)
9. Edit /etc/grub.d/10_linux` ([before](suspend/10_linux.org)/[after](suspend/10_linux)):
    - `initrd	${rel_dirname}/${initrd}`
    
    to read
    - `initrd	${rel_dirname}/acpi_override ${rel_dirname}/${initrd}`
10. Edit `/etc/default/grub` ([before](suspend/grub.org)/[after](suspend/grub)):
    - `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash psmouse.synaptics_intertouch=1"`
    
    to read
    - `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash psmouse.synaptics_intertouch=1 mem_sleep_default=deep"`
11. Update grub
    - `sudo update-grub2`
12. Reboot
13. Check that deep sleep is enabled and the default:
    - `cat /sys/power/mem_sleep` (Should give `s2idle [deep]`)

### Add suspend option to menu
- `sudo apt-get install gnome-shell-extension-suspend-button`
- Open `Tweaks` app and enable `Extensions > Suspend Button`

See https://github.com/laserb/gnome-shell-extension-suspend-button
