Nested Full disk encryption guide (Arch Linux)

Disclamer: This is intended to be used as a reference side by side with the arch linux installation guide on the wiki

Recognized benefits:

[*] Encryption Redundancy

[*] Multiple keys must be entered to decrypt a single partition

[*] Decreased chance of data recovery via cold boot attacks # aka 3 is more difficiult to recover than 1 

[*] Decreased chance of shoulder surfing

 

1. Run "fdisk -l" and identify what partiton you want 
to encrypt, for me it was /dev/sda3 because I'm going to have an nested 
encrypted LVM setup

 

2. encrypt the initial partition:

cryptsetup luksFormat -s 512 -h sha512 -c aes-xts-plain64 /dev/sda3

 

3. open the encrypted partition

cryptsetup open /dev/sda3 crypta

 

4. encrypt the unencrypted version of the encrypted partition (nesting)

cryptsetup luksFormat -s 512 -h sha512 -c serpent-xts-plain64 /dev/mapper/crypta

 

5. open the encrypted parititon /dev/mapper/crypta

cryptsetup open /dev/mapper/crypta cryptb

 

5. encrypt the unencrypted partiton (cryptb)

crypsetup luksFormat -s 512 -h sha512 -c twohfish-xts-plain64 luksFormat /dev/mapper/cryptb

 

6. open the encrypted parition /dev/mapper/cryptb

cryptsetup open /dev/mapper/cryptb cryptc

 

7. now you can do what I did if you want which is to create an lvm inside the nested encryption like so

 

pvcreate /dev/mapper/cryptc # make it a physical volume

 

vgcreate VolGroup /dev/mapper/cryptc # create volume group

 

8. Create all the logical volumes

 

lvcreate -L 10G VolGroup -n tmp

lvcreate -L 15G VolGroup -n root

lvcreate -L 50G VolGroup -n usr

lvcreate -L 25G VolGroup -n var

lvcreate -L 16G VolGroup -n swap

lvcreate -L 25G VolGroup -n home

lvcreate -l 100%FREE VolGroup -n data

 

9. format the logical volumes with filesystems

 

mkfs.ext4 /dev/mapper/VolGroup-root

mkfs.ext4 /dev/mapper/VolGroup-home

etc....

 

10. mount the logical volumes, install the base packages, and chroot into the new systems root directory

 

 

11. Create multiple encrypt hooks

# cp /usr/lib/initcpio/install/encrypt /etc/initcpio/install/encrypt2

# cp /usr/lib/initcpio/hooks/encrypt  /etc/initcpio/hooks/encrypt2

 

# cp /usr/lib/initcpio/install/encrypt /etc/initcpio/install/encrypt3

# cp /usr/lib/initcpio/hooks/encrypt  /etc/initcpio/hooks/encrypt3

 

# sed -i "s/cryptdevice/cryptdevice2/" /etc/initcpio/hooks/encrypt2

# sed -i "s/cryptkey/cryptkey2/" /etc/initcpio/hooks/encrypt2

 

12. edit /etc/mkinitcpio.conf and add "encrypt encrypt2 encrypt3" to the HOOKS=(... Line

 

13. Generate the new kernel image with "mkinitcpio -p <kernel package" for me its "mkinitcpio -p linux-hardened"

edit "/etc/default/grub" and add the following to the "GRUB_CMDLINE_LINUX="" line so it looks like this:

 

GRUB_CMDLINE_LINUX="cryptdevice=/dev/sda3:crypta cryptdevice2=/dev/mapper/crypta:cryptb cryptdevice3=/dev/mapper/cryptb:cryptc"

 

14. Generate the grub configuration file with "grub-mkconfig -o /boot/grub/grub.cfg"
