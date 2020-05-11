# raid2
1. Подключил тестовый репозиторий #git clone https://github.com/erlong15/otus-linux
2. В Vagrantfile добавил 5-й диск
:sata5 => {
 :dfile => './sata5.vdi',
 :size => 250,
 :port => 5
},
3. Убедился, что система видит диски #lsblk
4. И #sudo lshw -short | grep disk
5. Или #sudo fdisk -l
6. Но решил не использовать все диски и создать RAID 1
7. Занулил диски #sudo mdadm --zero-superblock --force /dev/sd{b,c,d,e,f}
8. Создал RAID1 #sudo mdadm --create --verbose /dev/md/md0 -l 1 -n 2 /dev/sd{b,c}
9. Проверил собрался ли RAID нормально #cat /proc/mdstat
Personalities : [raid1]
md127 : active raid1 sdc[1] sdb[0]
      254976 blocks super 1.2 [2/2] [UU]

unused devices: <none>
[vagrant@otuslinux mdadm]$ sudo mdadm --detail --scan --verbose
ARRAY /dev/md/md0 level=raid1 num-devices=2 metadata=1.2 name=otuslinux:md0 UUID=f9b13d77:bce9f776:00088b25:7c09e81a
   devices=/dev/sdb,/dev/sdc
10. Более подробо #sudo mdadm -D /dev/md/md0
/dev/md/md0:
           Version : 1.2
     Creation Time : Mon May 11 17:38:13 2020
        Raid Level : raid1
        Array Size : 254976 (249.00 MiB 261.10 MB)
     Used Dev Size : 254976 (249.00 MiB 261.10 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Mon May 11 17:38:15 2020
             State : clean
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : resync

              Name : otuslinux:md0  (local to host otuslinux)
              UUID : f9b13d77:bce9f776:00088b25:7c09e81a
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
11. #lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda       8:0    0   40G  0 disk
`-sda1    8:1    0   40G  0 part  /
sdb       8:16   0  250M  0 disk
`-md127   9:127  0  249M  0 raid1
sdc       8:32   0  250M  0 disk
`-md127   9:127  0  249M  0 raid1
sdd       8:48   0  250M  0 disk
sde       8:64   0  250M  0 disk
sdf       8:80   0  250M  0 disk
12.Или #cat /proc/mdstat
Personalities : [raid1]
md127 : active raid1 sdc[1] sdb[0]
      254976 blocks super 1.2 [2/2] [UU]
13. Убедился что информация верна #sudo mdadm --detail --scan --verbose
ARRAY /dev/md/md0 level=raid1 num-devices=2 metadata=1.2 name=otuslinux:md0 UUID=f9b13d77:bce9f776:00088b25:7c09e81a
   devices=/dev/sdb,/dev/sdc
14. Добавил информацию в конфиг
 #echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
 #sudo mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
15. Добавил в RAID ещё диск #sudo mdadm --add /dev/md/md0 /dev/sdd
И проверил #sudo mdadm -D /dev/md/md0
/dev/md/md0:
           Version : 1.2
     Creation Time : Mon May 11 17:38:13 2020
        Raid Level : raid1
        Array Size : 254976 (249.00 MiB 261.10 MB)
     Used Dev Size : 254976 (249.00 MiB 261.10 MB)
      Raid Devices : 2
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Mon May 11 17:55:20 2020
             State : clean
    Active Devices : 2
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 1

Consistency Policy : resync

              Name : otuslinux:md0  (local to host otuslinux)
              UUID : f9b13d77:bce9f776:00088b25:7c09e81a
            Events : 18

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc

       2       8       48        -      spare   /dev/sdd
16. Выше видно что появился диск sdd со статусом запасно - spare
17. Вывел из строя один из дисков в рейде # sudo mdadm /dev/md0 --fail /dev/sdc
18. И получил автоматическую замену диска 
    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       2       8       48        1      active sync   /dev/sdd

       1       8       32        -      faulty   /dev/sdc
19.Выше видно дефектный диск sdc - faulty
