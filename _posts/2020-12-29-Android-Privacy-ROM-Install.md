---
published: true
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

### Checkpoint: First Failure
Then when I tried to reboot into recovery, it wouldn't let me do so. It actually didn't let me do anything else at all after this, not even restart, boot into the OS, or shut down. I was stuck in a bootloop. At this point I knew I screwed something up and I read the first warning at the top of the LineageOS instructions. It only works with Android 10, not Android 11...

### Unbricking
I used the factory Pixel images from Google (found [here](https://developers.google.com/android/images)) to flash to my device. Scrolled down to the Pixel 4 section and found the most recent Android 10 version, and by clicking the "Flash" link it opens the Android Flash Tool. I needed to use Chrome to do it, but it was able to detect my phone in the browser via ADB and install a working OS on it again.

### Flashing Recovery (again)
Checked if adb is still working
```shell
adb devices
adb reboot bootloader
fastboot devices
fastboot flash boot lineage-17.1-20201227-recovery-flame.img
fastboot reboot
adb reboot bootloader
```
Navigated through the menu and factory reset the data, and sideloaded the Lineage for MicroG zip.
`adb sideload lineage-17.1-20201204-microG-flame.zip`
"Reboot system now"

### Checkpoint: Second Failure
It rebooted into the same old regular Android 10 that I had flashed from the factory Pixel image with the Android Flash Tool.

The recovery had given me an error that said the image is older than expected, so I bet it's probably expecting a newer ROM image than the age of the Recovery. Since my ROM is from Dec 4th and the Recovery is from Dec 27th it didn't work.

Unfortunately, the oldest Recovery image available on their website at the moment is 12/06, which is still newer than my 12/04 Lineage4microG image.

I could either:
1. wait until the newer Lineage4microG image comes out for Jan 2021, use the 12/27/2020 recovery image, and then flash the 01/??/2021 Lineage4microG zip, or
2. if I don't feel like waiting for them to make a new release and post it to the Lineage4microG website, I could build my own image. They have instructions on their website for how to build an image with their Docker container.

I have to decide how I want to proceed.

#### TO BE CONTINUED...


### Slight Change of Plans
I found something a couple days ago called [CalyxOS](https://calyxos.org/), which is another privacy/security focused ROM, however this one has _official_ MicroG support and comes bundled with it straight after installation.
I decided to work on installing this one instead. Other notable pros I found were that I don't have to mess with getting a recovery to work myself, or really doing much of anything myself. Their install instructions say to 1) get the latest build image from their website, put it in a new folder, 2) download the install script and put it in the same folder, 3) `chmod +x` to make the install script an executable and then run it. It automatically takes care of everything for you from there on and walks you through the steps on the terminal screen of what you have to do.

On my Pixel I flashed the most recent Android 10 again from the Android Flash Tool, then enabled dev tools and usb debugging, etc. Then I ran their install script. The script walked me through unlocking the bootloader, installing, then relocking the bootloader again (which is apparently another pro for this ROM as leaving the bootloader unlocked is a security vulnerability for someone to get into your phone or flash another OS over it). Also, the build I got yesterday from their site was based off Android 11, while LOS for MicroG is still based off of Android 10.

I installed the OS on the phone yesterday and I've been working on getting used to it and getting all my apps installed on it today, and I am really liking this. However, there's definitely some issues that I'll either need to solve, find a way around, or just learn to deal with.

### Bugs
- MicroG lets me use most apps, but I can't even get logged into Snapchat.
- Also, most applications that use Google Maps (or even Google Maps itself) doesn't work. Which is a real shame since there's not very many other good navigation apps out there. Even Waze doesn't work.
- MicroG gave me issues with signing into a Google account at first. I actually had to download an older version of the CalyxOS image, `flame-factory-2020.09.22.12.zip` which is based off Android 10, rather than the current `flame-factory-2021.01.06.14.zip`. I was able to find the older one by messing with the URL of the download link for the file of the actual one and it downloaded the older one. When I downloaded this old one, I put it into Airplane mode and also didn't connect to wifi during setup, nor did I install any of those other recommended apps it suggests. I enabled MicroG to be installed during setup. Then I went to the MicroG settings and turned off all the features from within there (Google device registration, Cloud Messaging, Google SafetyNet, Location modules, etc.) Then I went back and enabled wifi, and was able to log into a Google account. Also, when I connected to wifi and disabled airplane mode the phone started automatically downloading the latest OTA update for the actual current version. That software update installed and I now had the up-to-date CalyxOS based off Android 11 but also with my Google accounts signed in.
- With the Aurora app store, something I noticed is that if there's certain apps that I can't seem to find, it might be because they're only available for US markets, so with the VPN that I use I have to make sure to connect to an USA server, completely close Aurora and restart, and when I search for it then I can usually find it.