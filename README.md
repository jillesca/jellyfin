# jellyfin

In my setup Jellyfin will run as a docker container on a raspberry Pi 4 2GB. The media will be on an external USB, so I want this USB to be automounted everytime the pi is restarted.

## USB setup

### format usb

First thing is to format your usb, in my case I started with  `exfat` format, which gave me errors when testing. Therefore I ended formatting the usb as `ntfs`. To format you can do the following:

```bash
pi@raspberrypi:~$ df -h
S.ficheros     Tamaño Usados  Disp Uso% Montado en
/dev/root         26G   9,2G   16G  38% /
devtmpfs         776M      0  776M   0% /dev
tmpfs            937M      0  937M   0% /dev/shm
tmpfs            937M    17M  920M   2% /run
tmpfs            5,0M   4,0K  5,0M   1% /run/lock
tmpfs            937M      0  937M   0% /sys/fs/cgroup
/dev/mmcblk0p6   253M    49M  204M  20% /boot
tmpfs            188M      0  188M   0% /run/user/1000
/dev/sda1         30G   8,3G   22G  29% /mnt/jellyfinMedia
pi@raspberrypi:~$ sudo umount /dev/sda1
pi@raspberrypi:~$ sudo mkfs.ntfs /dev/sda1
Cluster size has been automatically set to 4096 bytes.
Initializing device with zeroes: 100% - Done.
Creating NTFS volume structures.
mkntfs completed successfully. Have a nice day.
```

### Prepare the system

```bash
sudo apt-get update -y && sudo apt-get upgrade -y
sudo mkdir /mnt/nfsshare
sudo chown -R pi:pi /mnt/nfsshare
sudo find /mnt/nfsshare/ -type d -exec chmod 755 {} \;
sudo find /mnt/nfsshare/ -type f -exec chmod 644 {} \;
```

### Configure the NFS

```bash
pi@raspberrypi:/media$ id pi
uid=1000(pi) gid=1000(pi) grupos=1000(pi),4(adm),20(dialout),24(cdrom),27(sudo),29(audio),44(video),46(plugdev),60(games),100(users),105(input),109(netdev),999(spi),998(i2c),997(gpio),995(docker)
pi@raspberrypi:/media$ sudo vim /etc/exports
pi@raspberrypi:/media$ cat /etc/exports
# /etc/exports: the access control list for filesystems which may be exported
#  to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/mnt/nfsshare *(rw,all_squash,insecure,async,no_subtree_check,anonuid=1000,anongid=1000)
```

Replace the `*` on the line `/mnt/nfsshare *` above with the IP segment you want to allow. In my case I used my LAN on my real deployment.

### Automount the USB on reboot

```bash
pi@raspberrypi:/mnt/nfsshare$ sudo blkid
/dev/mmcblk0p1: LABEL_FATBOOT="RECOVERY" LABEL="RECOVERY" UUID="5C7F-80DB" TYPE="vfat" PARTUUID="0004283f-01"
/dev/mmcblk0p5: LABEL="SETTINGS" UUID="8f1c81ed-b0b2-4c82-920a-c814e6c2ae7b" TYPE="ext4" PARTUUID="0004283f-05"
/dev/mmcblk0p6: LABEL_FATBOOT="boot" LABEL="boot" UUID="024C-430B" TYPE="vfat" PARTUUID="0004283f-06"
/dev/mmcblk0p7: LABEL="root" UUID="c06fca97-5d2b-41b6-9971-fc45bf639373" TYPE="ext4" PARTUUID="0004283f-07"
/dev/sda1: UUID="748F97411AA2CB11" TYPE="ntfs" PTTYPE="dos" PARTUUID="971c36c6-01"
/dev/mmcblk0: PTUUID="0004283f" PTTYPE="dos"

pi@raspberrypi:/mnt/nfsshare$ sudo vim /etc/fstab
pi@raspberrypi:/mnt/nfsshare$ cat /etc/fstab
proc            /proc           proc    defaults          0       0
/dev/mmcblk0p6  /boot           vfat    defaults          0       2
/dev/mmcblk0p7  /               ext4    defaults,noatime  0       1
UUID=748F97411AA2CB11 /mnt/nfsshare ntfs defaults,auto,users,rw,nofail,umask=000 0 0
# a swapfile is not a swap partition, no line here
#   use  dphys-swapfile swap[on|off]  for that
```

### Mount the usb manually to start

```bash
pi@raspberrypi:/mnt/nfsshare$ sudo mount /mnt/nfsshare/
pi@raspberrypi:/mnt/nfsshare$ ll
total 0
pi@raspberrypi:/mnt/nfsshare$ df -h
S.ficheros     Tamaño Usados  Disp Uso% Montado en
/dev/root         26G   9,2G   16G  38% /
devtmpfs         776M      0  776M   0% /dev
tmpfs            937M      0  937M   0% /dev/shm
tmpfs            937M    17M  920M   2% /run
tmpfs            5,0M   4,0K  5,0M   1% /run/lock
tmpfs            937M      0  937M   0% /sys/fs/cgroup
/dev/mmcblk0p6   253M    49M  204M  20% /boot
tmpfs            188M      0  188M   0% /run/user/1000
/dev/sda1         30G    66M   30G   1% /mnt/nfsshare
```

[Guide used to configure NFS](https://pimylifeup.com/raspberry-pi-nfs/)

## Docker config

[Instructions to install jellyfin on docker](https://jellyfin.org/docs/general/administration/installing.html#docker)
