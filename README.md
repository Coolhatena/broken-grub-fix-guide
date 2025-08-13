# broken-grub-fix-guide
### Disclaimer: This guide is half a rant against W*indows and half a guide I made to solve a problem I face often with my Dual Boot installation, if you are using a Linux distro different than Debian probably there will be some small changes, I tried to point out everything that might change in this guide, but I can't cover ALL posible Linux environments.

## WHAT DO I DO? A WINDOWS UPDATE BROKE MY DUAL BOOT AND NOW MY PC BOOTS DIRECTLY TO WINDOWS!
Welcome to the Dual Boot hell, if you have this kind of Linux/Windows installation you will face this situation at least once (Or five, ten, twenty times...) in a lifetime, but dont worry, your Linux installation, files, and everthing you had is most surely still there, your GRUB its the only thing that is broke, and you can fix it.

## STEP 1: GET A LIVE USB OF YOUR LINUX DISTRO
We have to use a live usb to get access to a linux terminal capable of manipulating our system files, we are NOT going to reinstall the OS (You can do it, but the whole purpose of reading this guide is to get back your linux installation, so it would be kind of pointless)

Once inside the live OS, do all the setup you need to use it confortably and open a terminal.

## STEP 2: IDENTIFY YOUR DRIVES
I currently have my dual boot in two separate drives, but this guide should help you even if you have both OS in a single drive.

Use `lsblk -f` on your terminal 
```
lsblk -f
```

This will give you an output like this for every drive/partition in your system:
```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
nvme1n1     259:0    0  500G  0 disk  
├─nvme1n1p1 259:1    0   100M  0 part /boot/efi
├─nvme1n1p2 259:2    0   16G   0 part [SWAP]
├─nvme1n1p3 259:3    0  100G   0 part /
└─nvme1n1p4 259:4    0  384G   0 part /home
```

You have to identify which partitions belong to your Linux install, in a standard installation you should have at least 3 partitions:
* A FAT32 partition: This is where your efi boot is installed.
* A ext4 partition: This is where your root `/` is installed, basically all your Linux OS.
* A swap partition: This partition is kind of irrelevant for the steps on this guide, but it can help your identify your Linux drive/partitions.

In my PC, my root is `nvme1n1p2` and my boot is `nvme1n1p3`, you have to adapt the following commands to your particular partitions. 

## STEP 3: MOUNT THE NEEDED SYSTEM FOLDERS
Now that you know exactly where are your Linux files, use the following commands to mount the necesary folders.

REMINDER: my root is `nvme1n1p2` and my boot is `nvme1n1p3`, change them depending on what your `lsblk -f` output was.

```
sudo mount /dev/nvme0n1p2 /mnt
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo mount /dev/nvme0n1p1 /mnt/boot/efi
sudo mount -t devpts devpts /mnt/dev/pts
```

Then, you have to mount your system efivars, this may not be necesary for everyone, but if your are not sure its better doing it:

```
sudo mount -t efivarfs efivarfs /sys/firmware/efi/efivars
```

## STEP 4: REGENERATE YOUR GRUB
Once everything is mounted, run this to access to your mounted root folder as root user:
```
sudo chroot /mnt
```

You should see your terminal change from something like `user@debian` to `root@debian` .

Now, to be sure your `efivars` are mounted correctly, type `efibootmgr` in your root terminal. 
```
efibootmgr
```

If you did everything right until this point you should see something like:
```
BootCurrent: 0001
Timeout: 5 seconds
BootOrder: 0001,0000,0002
Boot0000* Windows Boot Manager
Boot0001* USB CDROM: yourliveusb
Boot0002* UEFI: Network Device
... (Any bootable alternative your have in your computer)
```

If you see the correct output, now run the following command:
```
sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=debian
```
As you can see, this command is pretty Debian specific, if you have any other Linux distro, search which configuration you need here to regenerate your disto's GRUB (Or ask ChatGPT, he will surely know).
Once you run your grub-install, your should see an output like: 
```
Installing for x86_64-efi platform.
Installation finished. No errors reported.
```
If you see that output, it means you can now update your GRUB:
```
sudo update-grub
```
You will see a lot of cool techobabble on the screen, this means your GRUB is available again in your BIOS.

## STEP 5: ENJOY USING A FREE OPEN OS THAT ITS NOT AN INTRUSIVE SELFISH PIECE OF TRASH SOFTWARE
Depending on your computer and how much your W*ndows installation messed with your boot and BIOS, you may have to do some extra tinkering, but that totally depends on your PC, in my case i require some extra BIOS tricks to revive my system but i will omit that in this guide because its not Linux related.

I understand that, if you are using Dual Boot, you must have some ulterior motive to keep using that damn Microsoft OS, and after this recovery process W*ndows could not appear on the GRUB menu, dont worry. 
Once you have access to you Linux OS again, use `sudo update-grub` in your terminal again and you will see how your GRUB automatically detects you OS installations.
```
sudo update-grub
```
