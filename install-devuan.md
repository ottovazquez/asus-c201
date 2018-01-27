## Enable developer mode
See [how to enable developer mode](https://wiki.debian.org/InstallingDebianOn/Samsung/ARMChromebook#Enabling_Developer_Mode)

## Setup bootable USB
Create [Devuan](https://files.devuan.org/devuan_jessie/embedded/devuan_jessie_1.0.0_armhf_chromeveyron.img.xz) bootable USB using [Etcher](https://etcher.io) or `dd`. [RPI](https://www.raspberrypi.org/documentation/installation/installing-images/mac.md) has a good guide.


## Install Devuan

```
# boot from USB (Ctrl + U)
# login as root/toor
MNT_DIR=/mnt/setup
mkdir -p ${MNT_DIR}
```

```
# Backup some utilities from CrOS if don't have them yet
mount /dev/sdb3 ${MNT_DIR} -o r,discard
mkdir -p /usr/local/crbin
cp /dev/sdb3/usr/bin/cgpt   		/usr/local/crbin/
cp /dev/sdb3/usr/sbin/flashrom 	/usr/local/crbin/
umount ${MNT_DIR}
```

```
# resize partitions using CrOS cgpt tool
# /dev/mmcblk0p1  (STATE):  2G as swap
# /dev/mmcblk0p5 (ROOT-A): 13G as rootfs
# /dev/mmcblk0p5 (ROOT-B): 16M, not used
alias cgpt=/usr/local/crbin/cgpt
cgpt show /dev/mmcblk0

cgpt add -i 1 -b 26587136 -s 4194304  -t data   -l STATE  /dev/mmcblk0
cgpt add -i 5 -b 315392   -s 32768    -t rootfs -l ROOT-B /dev/mmcblk0
cgpt add -i 3 -b 348160   -s 26238976 -t rootfs -l ROOT-A /dev/mmcblk0
cgpt show /dev/mmcblk0

# set flags
cgpt add -i 2 -S 1 -T 15 -P 15 /dev/mmcblk0
cgpt add -i 4 -S 1 -T 15 -P 0  /dev/mmcblk0
cgpt add -i 6 -S 1 -T 15 -P 0  /dev/mmcblk0
cgpt show /dev/mmcblk0

sync
mkswap /dev/mmcblk0p1
mkfs.ext4 /dev/mmcblk0p3
```

```
# insert second Devuan USB and adjust devices as required 
# copy `kernel` and `rootfs` partitions
dd if=/dev/sdb1 of=/dev/mmcblk0p2
dd if=/dev/sdb2 of=/dev/mmcblk0p3
resize2fs /dev/mmcblk0p3

# plan-b untested 
# mount /dev/mmcblk0p3 ${MNT_DIR} -o rw,discard
# cp -ax / ${MNT_DIR}/
```

```
# setup Devuan 
mount /dev/mmcblk0p3 ${MNT_DIR} -o rw,discard

# update fstab
FS_UUID=$(blkid | grep mmcblk0p3 | grep -Eo '[0-9a-f]{8}-([0-9a-f]{4}-){3}[0-9a-f]{12}')
SW_UUID=$(blkid | grep mmcblk0p1 | grep -Eo '[0-9a-f]{8}-([0-9a-f]{4}-){3}[0-9a-f]{12}')
echo "\
UUID=${FS_UUID} /     ext4    errors=remount-ro,noatime,nodiratime,discard  0       1
UUID=${SW_UUID} none  swap    sw                                            0       0
" > ${MNT_DIR}/etc/fstab

# update sources
echo "\
deb http://auto.mirror.devuan.org/merged/ jessie           main non-free contrib
deb http://auto.mirror.devuan.org/merged/ jessie-security  main contrib non-free
deb http://auto.mirror.devuan.org/merged/ jessie-updates   main contrib non-free
deb http://auto.mirror.devuan.org/merged/ jessie-backports main contrib non-free
" > /etc/apt/sources.list

# this should work...
chroot ${MNT_DIR}
adduser bob
apt-get update
apt-get install sudo
usermod -a -G sudo,users bob
exit

umount ${MNT_DIR}
```

## Resources

- [Install Debian on Chromebook](https://wiki.debian.org/InstallingDebianOn/Samsung/ARMChromebook)
- [Debian Jessie on Samsung3](https://www.neowin.net/forum/topic/1173005-replacing-chrome-os-with-debian-jessie-on-the-samsung-series-3-chromebook/)
