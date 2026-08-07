
1. Internet access is blocked,
2. Update ntp server from time.google.com to local ntp server in file,
   /media/mmcblk0p2/data/etc/scripts/02-ntpd
3. Set TZ to AEST-10AEDT,M10.1.0,M4.1.0/3 on GUI or edit in file /etc/TZ
4. night mode always on, so change to day mode, and reboot

vi /media/mmcblk0p2/data/usr/bin/fang-ir-control.sh
#-------------------------------------------------------
#!/bin/sh

# 1. Based on /proc/isp/ae/gain
echo "AE-Gain Precision Auto-IR Control Started"

gpio_ms1 -n 2 -m 1 -v 1
gpio_aud write 1 1 0
gpio_aud write 0 2 1
gpio_aud write 1 0 0
echo 0x40 > /proc/isp/filter/saturation
sleep 3
IR_ON=0

while :
do
    HEX_GAIN="$(cat /proc/isp/ae/gain 2>/dev/null)"
    if [ -n "$HEX_GAIN" ]; then
        GAIN=$(printf "%d" "$HEX_GAIN" 2>/dev/null || echo 0)
        if [ $GAIN -gt 6800 ]; then
            if [ $IR_ON -eq 0 ]; then
                echo 0x0 > /proc/isp/filter/saturation
                gpio_aud write 1 0 1
                gpio_ms1 -n 2 -m 1 -v 0
                IR_ON=1
            fi
        elif [ $GAIN -lt 6000 ]; then
            if [ $IR_ON -eq 1 ]; then
                gpio_ms1 -n 2 -m 1 -v 1
                gpio_aud write 1 0 0
                echo 0x40 > /proc/isp/filter/saturation
                IR_ON=0
            fi
        fi
    fi
    sleep 5
done

exit 0

# 2. change to day mode
##-------------------
gpio_ms1 -n 2 -m 1 -v 1
gpio_aud write 1 0 0
echo 0x40 > /proc/isp/filter/saturation
exit 0
##-------------------

# 3. OLD SCRIPT BACKUP
echo "IR script started"

# ir_init
gpio_ms1 -n 2 -m 1 -v 1
gpio_aud write 1 1 0
gpio_aud write 0 2 1
gpio_aud write 1 0 0

sleep 3

# ir loop
IR_ON=0

while :
do
    DAY="$(gpio_aud read 2)"
    if [ $DAY -eq 1 ]
    then
        if [ $IR_ON -eq 1 ]
        then
            gpio_ms1 -n 2 -m 1 -v 1
            gpio_aud write 1 0 0
            echo 0x40 > /proc/isp/filter/saturation
            IR_ON=0
        fi
    else
        if [ $IR_ON -eq 0 ]
        then
            echo 0x0 > /proc/isp/filter/saturation
            gpio_aud write 1 0 1
            gpio_ms1 -n 2 -m 1 -v 0
            IR_ON=1
        fi
    fi
    sleep 3
done
#-------------------------------------------------------



