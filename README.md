# liveusb
![liveusb](https://github.com/alkisg/liveusb/raw/master/liveusb.png)

This project allows you to create a grub2-based USB flash drive with the following boot options that work under **both** BIOS and UEFI firmwares:
 * Ubuntu live CDs directly in .iso format
 * Windows 10 setup, as prepared by the media creation tool
 * Boot from network, using iPXE
 * Memory test

The main idea is that you download [liveusb.zip](https://github.com/alkisg/liveusb/raw/master/liveusb.zip), extract it to get a 300 MiB fat32 image named liveusb.img, and then write that image to your drive. Finally, you resize the partition to fill the free space.

Initially, the liveusb can only boot from the network and do a memory test. You're supposed to manually copy the ubuntu.iso and the Windows files later on.

# Linux instructions
To create the liveusb drive under Linux, run the following commands, while replacing sdx with your actual device:
```
wget -nv https://github.com/alkisg/liveusb/raw/master/liveusb.zip
unzip liveusb.zip
sudo umount /dev/sdx*
sudo dd if=liveusb.img of=/dev/sdx bs=1M status=progress
sudo partprobe /dev/sdx
sudo apt install --yes gparted
sudo gparted /dev/sdx
```
Resize the /dev/sdx1 partition to fill the free space and close gparted. Finally, you may need to run the following command to [work around a bug in parted](https://wiki.archlinux.org/index.php/Parted#Resized_FAT32_partition_then_unrecognized_on_Windows):
```
echo -ne '\xeb\x58\x90' | sudo dd conv=notrunc bs=1 count=3 of=/dev/sdx1
```
Users that have fully updated Ubuntu 18.04 or newer, don't need to run that command as [the fix has been SRU'ed](https://bugs.launchpad.net/ubuntu/+source/parted/+bug/1820090).

# Windows instructions
Download [liveusb.zip](https://github.com/alkisg/liveusb/raw/master/liveusb.zip) and unzip it to get liveusb.img.

Then download [Rufus](https://rufus.ie/) and use it to write liveusb.img to a USB flash drive.

Finally, use [EaseUS Partition Master Free](https://www.easeus.com/partition-manager/epm-free.html) to resize the fat32 partition to fill the free space.

# Add the Ubuntu .iso files
The [grub.cfg](https://github.com/alkisg/liveusb/blob/master/grub.cfg) file included in liveusb expects you to download the following two iso images and put them inside the /liveusb/ubuntu/ directory:
 * [ubuntu-mate-18.04.1-desktop-amd64.iso](http://cdimage.ubuntu.com/ubuntu-mate/releases/18.04.1/release/ubuntu-mate-18.04.1-desktop-amd64.iso)
 * [ubuntu-mate-18.04.1-desktop-i386.iso](http://cdimage.ubuntu.com/ubuntu-mate/releases/18.04.1/release/ubuntu-mate-18.04.1-desktop-i386.iso)

If you want to add more .isos, you can put them there but you also need to edit /grub/grub.cfg. Autodetection of .iso files will be implemented in a couple of years when [Ubuntu grub will support it](https://bugs.launchpad.net/ubuntu/+source/grub2-signed/+bug/1818557).

# Add the Windows boot files
Visit the [Windows 10 media creation tool download page](https://www.microsoft.com/software-download/windows10) and run the tool to create a second USB flash drive, that only supports Windows 10.

Then in that drive rename /efi/boot/bootx64.efi to /efi/boot/bootx64m.efi.

Finally, copy all those files to the liveusb drive. The /efi directory exists in both drives, so answer "Merge" when prompted, but no files have the same name, so nothing will get overwritten.

# Add memtest.efi
I don't think memtest86.efi is redistributable, so you need to download it yourself. Get and unzip [memtest86-usb.zip](https://www.memtest86.com/downloads/memtest86-usb.zip), extract memtest86-usb.img from it, write it to a second USB flash drive with dd or Rufus, and finally copy /EFI/BOOT/BOOTX64.efi from there to /liveusb/efi/memtest86.efi.

# What does liveusb include?
Liveusb is created by running the [liveusb script](https://github.com/alkisg/liveusb/blob/master/liveusb) in my own PC. You can run it yourself to generate your own liveusb.img, as long as you have both grub-efi-amd64-signed (for UEFI) and grub-pc-bin (for BIOS) installed. The significant files in the USB flash drive are as follows. You are supposed to provide the files mentioned by "YOU":
```
.
├── EFI
│   └── BOOT
│       ├── bootx64.efi  # UEFI: Ubuntu's signed bootloader, shimx64.efi
│       └── bootx64m.efi  # YOU: Microsoft's efi bootloader, renamed
├── grub
│   └── grub.cfg  # The grub configuration file
└── liveusb
    ├── efi
    │   ├── ipxe.efi  # UEFI: Network boot (iPXE)
    │   └── memtest86.efi  # YOU: Memory test
    ├── linux16
    │   ├── ipxe.lkrn  # BIOS: Network boot (iPXE)
    │   └── memtest86+.bin  # BIOS: Memory test
    ├── memdisk
    │   └── dos.img  # BIOS: DOS, for BIOS flashing etc
    └── ubuntu
        ├── ubuntu-mate-18.04.1-desktop-amd64.iso  # YOU: Ubuntu MATE 64 bit
        └── ubuntu-mate-18.04.1-desktop-i386.iso  # YOU: Ubuntu MATE 32 bit
```
