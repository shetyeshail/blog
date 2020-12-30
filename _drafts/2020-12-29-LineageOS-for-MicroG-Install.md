---
published: false
---
I have a Pixel 4, and I have been debating installing either [GrapheneOS](https://grapheneos.org/) or [LineageOS](https://lineageos.org/) on it for a while. Then I found out about something called [microG](https://microg.org/). It basically provides open-source free-as-in-freedom alternatives to the proprietary APIs and services that Google provides for Android, such as through Google Play Services. They build their own version of LineageOS that comes bundled with their libraries. That ROM is called [LineageOS for microG](https://lineage.microg.org/) (yes, I know, it's a very clever name). I figured that would be a good step in the right direction for me to wean myself off of using Google's Android to using something that is more open-source and without as much of the invasive Google software.

This post is basically documenting my journey (even all the parts I screwed up).

## DISCLAIMER
THIS IS A WORK IN PROGRESS, DO NOT TRY TO REPLICATE THIS PROCESS. I AM NOT RESPONSIBLE FOR ANY DAMAGES YOU MAY MAKE TO ANY DEVICES BY FOLLOWING THIS POST.

## Materials
- Thinkpad T450s running Arch Linux, pacman, yay for AUR, sudo access, etc.
- Internet access
- Chrome browser
- Pixel 4
- USB-A to USB-C cable

## And so the story begins...
### Downloading images
I started by following the instructions on the [LineageOS for microG](https://lineage.microg.org/) website. They mention to download the latest image for the phone, which I downloaded for the Pixel 4 (the codename for this model is 'flame'). The file is called `lineage-17.1-20201204-microG-flame.zip`.

Then I downloaded the latest recovery from [https://download.lineageos.org/flame](https://download.lineageos.org/flame). The file is called `lineage-17.1-20201227-recovery-flame.img`.

### Installing ADB and fastboot
Installed adb and fastboot for Arch by following the instructions [here](https://wiki.archlinux.org/index.php/Android_Debug_Bridge).
```shell
yay -S android-udev
yay -S android-tools
yay -S android-completion
```

### Unlocked Developer Mode on Pixel 4
Need to enable Developer Tools in Settings by clicking on Build Version multiple times. Then from within Developer options we need to enable USB Debugging.

Then I connected the phone to the computer via the usb cable. The phone prompted me to trust the computer for debugging and I accepted.

### Unlocking Bootloader
`adb devices` should show you that the laptop sees the phone via adb
`adb reboot bootloader` should take you into the bootloader.
`fastboot devices` should show you that the device is now connected in fastboot mode
`fastboot flashing unlock` unlocks the bootloader.

### Re-Enable Developer Mode and USB Debugging
Same as before, just have to do it again.

### Tried Flashing Recovery
`adb reboot bootloader` to boot into the bootloader again
`fastboot devices` the phone should show up again so we know it's working
`fastboot flash boot lineage-17.1-20201227-recovery-flame.img` flashes the recovery image to the boot partition.

### Checkpoint: First Mistake
Then when I tried to reboot into recovery, it wouldn't let me do so. It actually didn't let me do anything else at all after this, not even restart, boot into the OS, or shut down. I was stuck in a bootloop. At this point I knew I screwed something up and I read the first warning at the top of the LineageOS instructions. It only works with Android 10, not Android 11...



