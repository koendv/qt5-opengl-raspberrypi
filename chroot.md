# Setting up a chroot on raspbian

Download Raspbian Image
```
wget https://downloads.raspberrypi.org/raspbian_full_latest
```
Extract Raspbian image
```
unzip 2019-09-26-raspbian-buster.zip
mv 2019-09-26-raspbian-buster.img chroot.img
```
Add disk space for 16gb sdcard
```
fallocate -l 16g chroot.img
```
Set up image as loop device
```
losetup -P /dev/loop0 chroot.img
```
Check file system
```
e2fsck -y -f /dev/loop0p2
```
Expand partition
```
growpart /dev/loop0 2
resize2fs /dev/loop0p2
```
Mount partition
```
mount -o rw /dev/loop0p2  /mnt
mount -o rw /dev/loop0p1 /mnt/boot
mount --bind /dev /mnt/dev/
mount --bind /sys /mnt/sys/
mount --bind /proc /mnt/proc/
mount --bind /dev/pts /mnt/dev/pts
```
Fix ld.so.preload
```
sed -i 's/^/#/g' /mnt/etc/ld.so.preload
```
Chroot to raspbian
```
chroot /mnt /bin/bash
```
Fix locale
```
perl -pi -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/g' /etc/locale.gen
locale-gen en_US.UTF-8
update-locale en_US.UTF-8
```
At this point, the chroot is ready for use. Do whatever you wanted to do in the chroot. Afterwards, exit the chroot.
```
exit
```
Revert ld.so.preload
```
sed -i 's/^#//g' /mnt/etc/ld.so.preload
```
Unmount everything
```
umount /mnt/{dev/pts,dev,sys,proc,boot,}
```
Unmount loop device
```
losetup -d /dev/loop0
```

[Source](https://wiki.debian.org/RaspberryPi/qemu-user-static)

Not truncated.

