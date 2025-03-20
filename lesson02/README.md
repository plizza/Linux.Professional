# Работа с mdadm

## Добавляю к ВМ 5 новых дисков
```sh
lshw -short | grep disk
/0/100/1.1/0.0.0    /dev/cdrom  disk           QEMU DVD-ROM
/0/100/1.1/0.0.0/0  /dev/cdrom  disk
/0/100/4/0/0.0.0    /dev/sda    disk           10GB QEMU HARDDISK
/0/100/4/0/0.1.1    /dev/sdb    disk           1073MB QEMU HARDDISK
/0/100/4/0/0.2.2    /dev/sdc    disk           1073MB QEMU HARDDISK
/0/100/4/0/0.3.3    /dev/sdd    disk           1073MB QEMU HARDDISK
/0/100/4/0/0.4.4    /dev/sde    disk           1073MB QEMU HARDDISK
/0/100/4/0/0.5.5    /dev/sdf    disk           1073MB QEMU HARDDISK

Далее history

## Создаю RAID-6

mdadm --create --verbose /dev/md0 -l 6 -n 5 /dev/sd{b,c,d,e,f}

mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 1046528K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

## Проверяю

cat /proc/mdstat
Personalities : [linear] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid6 sdf[4] sde[3] sdd[2] sdc[1] sdb[0]
      3139584 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/5] [UUUUU

## Пробую сломать рейд

Далее history

   24  mdadm /dev/md0 --fail /dev/sde 
   25  cat /proc/mdstat 
   26  mdadm -D /dev/md0
   27  mdadm /dev/md0 --remove /dev/sde 

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       -       0        0        3      removed
       4       8       80        4      active sync   /dev/sdf

       3       8       64        -      faulty   /dev/sde

## Возвращаю диск в рейд

   28  mdadm /dev/md0 --add /dev/sde

## Созда. GPT таблицу, пять разделов и монтирую их в системе

   31  parted -s /dev/md0 mklabel gpt
   32  parted /dev/md0 mkpart primary ext4 0% 20%
   33  parted /dev/md0 mkpart primary ext4 20% 40%
   34  parted /dev/md0 mkpart primary ext4 40% 60%
   35  parted /dev/md0 mkpart primary ext4 60% 80%
   36  parted /dev/md0 mkpart primary ext4 80% 100%
   37  for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
   38  mkdir -p /raid/part{1,2,3,4,5}
   39  for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done

df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           392M  812K  391M   1% /run
/dev/sda1       9.8G  1.5G  7.9G  16% /
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           392M  8.0K  392M   1% /run/user/0
/dev/md0p1      586M   24K  543M   1% /raid/part1
/dev/md0p2      587M   24K  544M   1% /raid/part2
/dev/md0p3      586M   24K  543M   1% /raid/part3
/dev/md0p4      587M   24K  544M   1% /raid/part4
/dev/md0p5      586M   24K  543M   1% /raid/part5
