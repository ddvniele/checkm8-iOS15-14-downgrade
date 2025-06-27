# ⏱️ activate iOS 15/14 after a downgrade with futurerestore
this is a little guide that aims to help you downgrading your device from iOS 18/17/16 to iOS 15/14 avoiding every activation problem. i’d like to specify that this is entirely based on my experience so far and if you have any suggestion to make, don’t hesitate to do it.

## 📮 last updated: Nov 13, 2024
- updated to get rid of palera1n fakefs use (takes less time and less device storage)
- added troubleshooting page

>[!IMPORTANT]
>  you'll need:
> - a mac (probably possible with linux, not sure tho)
> - a checkm8 device (basically those that can be jailbroken with palera1n)
>   - except for iPhone X that would break the restore
>   - this means that iPhone XS/XR and up, iPad mini 5 and up, iPad air 3 and up, iPad pro 2018 and up, and iPad 8th gen and up are not supported
> - the ipsw files specific for your device of both the last version available and the version you want to downgrade to ([ipsw.me](https://ipsw.me/))
> - the blobs shsh2 of the version you want to downgrade to (specifically saved for your device when the version was still signed, maybe with [Tss Saver](https://tsssaver.1conan.com/) or smth)
> - [this file](https://github.com/ddvniele/checkm8-iOS15-14-downgrade/releases/download/downloads/mobileactivationd) (download it on your desktop)
> - palera1n installed on your mac ([see here](https://palera.in/))
> - the FileZilla app installed on your mac ([download here](https://github.com/ddvniele/checkm8-iOS15-14-downgrade/releases/download/downloads/FileZilla.zip))
> - the SSHRD_Script cloned on your mac (by running this command on a terminal window: <code>git clone https://github.com/verygenericname/SSHRD_Script --recursive</code>)

## 1. GRABBING FILES
the first thing to do is restoring your device to the latest iOS 16/17 version available with iTunes/Finder, and then normally activating it with the initial configuration setup as if it was a new device. then:
- connect your device to your mac and in a new terminal window run <code>palera1n -l</code> and follow the process
- after the device reboots, open the new palera1n app and install Sileo (set the password to "alpine" when it asks you)
- open Sileo, reload the sources and do all the updates that are needed
- search for Filza File Manager and install it
- open Filza; you need to grab some files from the internal storage and send them to your mac (use airdrop or whatever):
  - go into /var/containers/Data/System and open the folder that ends with "mobileactivationd", then "Library", then open "activation_records" and send the file "activation_record.plist" to your mac (you'll probably need to change the file permissions in order to do it)
  - do the same entering on the folder "internal" instead of "activation_records" and grab the file "data_ark.plist"
  - go into /var/mobile/Library and grab the **entire** "FairPlay" folder (you can zip it and then unzip it on your mac)
  - go into /var/wireless/Library/Preferences and grab the file whose name ends with "nobackup.plist"
- you can now put your device into dfu mode and then in pwndfu (you can follow [this guide](https://github.com/ddvniele/iOS-64bit-dualboot-guide#3-enter-pwndfu-mode) i wrote some times ago)

## 2. DOWNGRADING WITH FUTURERESTORE
ensure that all the files you previously grabbed are secured on your mac before restoring.
- download the futurerestore script [here](https://github.com/ddvniele/checkm8-iOS15-14-downgrade/releases/download/downloads/futurerestore) and put it into a folder called "futurerestore" located on your desktop.
- put into this folder also the shsh2 file of the blobs and the ipsw for the version you want to downgrade to
- verify if your device has a baseband. you can search on internet for it. usually wifi iPads don't have it
- open a terminal window and type <code>cd</code>, then a space, then drag and drop the futurerestore folder on this window and press enter
- enter the command <code>chmod +x ./futurerestore</code>
- enter the command <code>./futurerestore -t (SHSH) (IPSW) --use-pwndfu --latest-sep --latest-baseband</code>
  - if your device has no baseband, replace --latest-baseband with --no-baseband
  - replace (SHSH) with the name of the file that contains your blobs (don't forget the extension), same thing for the ipsw file in (IPSW)
  - usually the first tries won't succeed; rerun the same command until the restore succeeds
- now wait for the restore to finish without unplugging your device from your mac

## 3. BOOTING SSH RAMDISK
when the device finishes restoring and boots into the setup screen, it's time to boot into its ssh ramdisk.
- put your device into dfu mode (you can search on internet how to do it, it's really simple)
- open now another terminal window, and enter the command <code>cd SSHRD_Script</code> in order to use the ssh ramdisk
- enter the command <code>./sshrd.sh (iOS version)</code>, replacing (iOS version) with the actual version you downgraded to
- enter then the command <code>./sshrd.sh boot</code> to boot the ramdisk
- enter the command <code>./sshrd.sh ssh</code> to connect to the ssh
- enter the command <code>mount_filesystems</code> to mount system files

## 4. MOBILEACTIVATIOND
it's time to use the mobileactivationd file you downloaded at the beginning.
- on the same terminal window as the steps before, run <code>ldid -e /mnt1/usr/libexec/mobileactivationd > /mnt1/ents.xml</code>
- run <code>mv /mnt1/usr/libexec/mobileactivationd /mnt1/usr/libexec/mobileactivationd_backup</code>
- open now FileZilla, go to File > Site Manager, put inside the window that shows up [these settings](https://user-images.githubusercontent.com/96156354/234979392-bf090edd-4516-4d41-ab1e-4731dc771f0e.png) and then click on "Connect"
- go now into /mnt1/usr/libexec and drag and drop here the mobileactivationd file downloaded at the beginning of this guide
- return into the terminal window you used before and run <code>chmod 755 /mnt1/usr/libexec/mobileactivationd</code>
- then run <code>ldid -S/mnt1/ents.xml /mnt1/usr/libexec/mobileactivationd</code>
- enter now the command <code>reboot</code> and wait for your device to reboot

## 5. TRANSFER ACTIVATION FILES
now we have to transfer the other files we grabbed in step 1 into the device.
- boot into ssh ramdisk again, putting your device into dfu mode, and in the same terminal window as before, running <code>./sshrd.sh boot</code> and then <code>./sshrd.sh ssh</code>
- mount now the filesystems: <code>mount_filesystems</code>
- open FileZilla again and connect again to your device in the same way you did before (follow [these settings](https://user-images.githubusercontent.com/96156354/234979392-bf090edd-4516-4d41-ab1e-4731dc771f0e.png) again)
- go into /mnt2/mobile/Media/Downloads and create a new directory named "1", then drag and drop here your Activation folder (a folder that contains all the files we grabbed in step 1)
- move the "1" folder into /mnt2/mobile/Media
- move the "data_ark.plist" contained inside the Activation folder into /mnt2/containers/Data/System and then enter each folder, then the Library folder, until you find a folder called "internal" that contains a "data_ark.plist" that you need to delete and replace with the "data_ark.plist" file of your Activation folder
- in the same folder where is contained the "internal" folder, create a new directory called "activation_records" and drag and drop here the "activation_record.plist" that we grabbed in step 1
- drag and drop your FairPlay folder into /mnt2/mobile/Library
- delete the file that ends with "nobackup.plist" into /mnt2/wireless/Library/Preferences and replace it with your own file grabbed in step 1
- return into the terminal window you used before and run <code>reboot</code>
enjoy now your device fully activated on iOS 15!

## ⁉️ TROUBLESHOOTING
if you are unable to successfully activate your device because of some errors, you can find some solutions for most of the common errors on the [troubleshooting page](https://github.com/ddvniele/checkm8-iOS15-14-downgrade/blob/main/troubleshooting.md)

## 📥 NOT AN ICLOUD BYPASS!
this is not a method for bypassing an iCloud lock. in fact, you can't grab files with Filza if you can't activate your device on the newest iOS available.

## 📂 credits
- [Orangera1n](https://gist.github.com/Orangera1n), who was the first to write a guide about it
- [kjutzn](https://github.com/kjutzn) with his project [cry-ptex1](https://github.com/kjutzn/cry-ptex1) and his great help into his discord server
- [SSHRD_Script](https://github.com/verygenericname/SSHRD_Script) by [verygenericname](https://github.com/verygenericname)
- [palera1n](https://palera.in/) team
