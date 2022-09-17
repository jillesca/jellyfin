# jellyfin

In my setup Jellyfin will run as a docker container on a raspberry Pi 4 2GB. The media will be on an external USB, so I want this USB to be automounted everytime the pi is restarted.

## USB setup

Review usb is present

```bash{1,10}
pi@raspberrypi:~$ lsblk -fp
NAME             FSTYPE LABEL    UUID                                 FSAVAIL FSUSE% MOUNTPOINT
/dev/sda
└─/dev/sda1      exfat  Untitled 6251-D213
/dev/mmcblk0
├─/dev/mmcblk0p1 vfat   RECOVERY 5C7F-80DB
├─/dev/mmcblk0p2
├─/dev/mmcblk0p5 ext4   SETTINGS 8f1c81ed-b0b2-4c82-920a-c814e6c2ae7b
├─/dev/mmcblk0p6 vfat   boot     024C-430B                             203,9M    19% /boot
└─/dev/mmcblk0p7 ext4   root     c06fca97-5d2b-41b6-9971-fc45bf639373   15,6G    34% /

pi@raspberrypi:~$ sudo mkdir /mnt/jellyfinMedia
pi@raspberrypi:~$ sudo mount -t exfat /dev/sda1 /mnt/jellyfinMedia
pi@raspberrypi:~$ cd /mnt/jellyfinMedia/
pi@raspberrypi:/mnt/jellyfinMedia$ ll
total 8,3G
-rwxr-xr-x 1 root 8,3G abr  3 16:49 1080p.BluRay.x264.mkv*
```

Auto mount usb

```bash
pi@raspberrypi:/mnt/jellyfinMedia$ sudo blkid
/dev/mmcblk0p1: LABEL_FATBOOT="RECOVERY" LABEL="RECOVERY" UUID="5C7F-80DB" TYPE="vfat" PARTUUID="0004283f-01"
/dev/mmcblk0p5: LABEL="SETTINGS" UUID="8f1c81ed-b0b2-4c82-920a-c814e6c2ae7b" TYPE="ext4" PARTUUID="0004283f-05"
/dev/mmcblk0p6: LABEL_FATBOOT="boot" LABEL="boot" UUID="024C-430B" TYPE="vfat" PARTUUID="0004283f-06"
/dev/mmcblk0p7: LABEL="root" UUID="c06fca97-5d2b-41b6-9971-fc45bf639373" TYPE="ext4" PARTUUID="0004283f-07"
/dev/mmcblk0: PTUUID="0004283f" PTTYPE="dos"
/dev/sda1: LABEL="Untitled" UUID="6251-D213" TYPE="exfat" PARTUUID="971c36c6-01"

pi@raspberrypi:/mnt/jellyfinMedia$ sudo cp /etc/fstab /etc/fstab.back
pi@raspberrypi:/mnt/jellyfinMedia$ sudo vim /etc/fstab
pi@raspberrypi:/mnt/jellyfinMedia$ cat /etc/fstab
proc            /proc           proc    defaults          0       0
/dev/mmcblk0p6  /boot           vfat    defaults          0       2
/dev/mmcblk0p7  /               ext4    defaults,noatime  0       1
UUID=6251-D213 /mnt/jellyfinMedia/ exfat defaults,auto,users,rw,nofail 0 0
# a swapfile is not a swap partition, no line here
#   use  dphys-swapfile swap[on|off]  for that
pi@raspberrypi:/mnt/jellyfinMedia$ cat /etc/fstab.back
proc            /proc           proc    defaults          0       0
/dev/mmcblk0p6  /boot           vfat    defaults          0       2
/dev/mmcblk0p7  /               ext4    defaults,noatime  0       1
# a swapfile is not a swap partition, no line here
#   use  dphys-swapfile swap[on|off]  for that

UUID=6251-D213 /mnt/jellyfinMedia/ exfat defaults,auto,users,rw,nofail 0 0
```

[Guide use to automount usb](https://www.shellhacks.com/raspberry-pi-mount-usb-drive-automatically/)

## Docker config

[Instructions to install jellyfin on docker](https://jellyfin.org/docs/general/administration/installing.html#docker)
