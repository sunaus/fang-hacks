
1. Internet access is blocked,
2. Update ntp server from time.google.com to local ntp server in file,
   /media/mmcblk0p2/data/etc/scripts/02-ntpd
3. Set TZ to AEST-10AEDT,M10.1.0,M4.1.0/3 on GUI or edit in file /etc/TZ
4. night mode always on, so change to day mode, and reboot
vi /media/mmcblk0p2/data/usr/bin/fang-ir-control.sh
#!/bin/sh
echo "IR script started"
## change to day mode
##-------------------
gpio_ms1 -n 2 -m 1 -v 1
gpio_aud write 1 0 0
echo 0x40 > /proc/isp/filter/saturation
exit 0
##-------------------

