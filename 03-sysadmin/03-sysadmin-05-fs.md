## 03-sysadmin-05-fs
1. Узнайте о [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных) файлах.
	```bash
	Done
	```
2. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?
	```bash
	Нет, не могут, т.к. указывают на один и тот же inode
3. Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

    ```bash
    Vagrant.configure("2") do |config|
      config.vm.box = "bento/ubuntu-20.04"
      config.vm.provider :virtualbox do |vb|
        lvm_experiments_disk0_path = "/tmp/lvm_experiments_disk0.vmdk"
        lvm_experiments_disk1_path = "/tmp/lvm_experiments_disk1.vmdk"
        vb.customize ['createmedium', '--filename', lvm_experiments_disk0_path, '--size', 2560]
        vb.customize ['createmedium', '--filename', lvm_experiments_disk1_path, '--size', 2560]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk0_path]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk1_path]
      end
    end
    ```

    Данная конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2.5 Гб.
	```bash
	# Готово, только я использую для заданий домашний сервер с гипервизором.
	❯ lsblk
	NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
	...
	vdb     252:16   0  2.6G  0 disk
	vdc     252:32   0  2.6G  0 disk
	```
4. Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.
	```bash
	❯ fdisk /dev/vdb
	...
	Command (m for help): g
	Created a new GPT disklabel (GUID: 547A4A63-BC13-024B-91E5-F014AD3E7381).
	
	Command (m for help): n
	Partition number (1-128, default 1):
	First sector (2048-5369822, default 2048):
	Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-5369822, default 5369822): +2G

	Created a new partition 1 of type 'Linux filesystem' and of size 2 GiB.

	Command (m for help): n
	Partition number (2-128, default 2):
	First sector (4196352-5369822, default 4196352):
	Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5369822, default 5369822):

	Created a new partition 2 of type 'Linux filesystem' and of size 573 MiB.

	Command (m for help): w
	The partition table has been altered.
	Calling ioctl() to re-read partition table.
	Syncing disks.
	```
5. Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.
	```bash
	❯ sfdisk -d /dev/vdb | sfdisk /dev/vdc
	```
6. Соберите `mdadm` RAID1 на паре разделов 2 Гб.
	```bash
	❯ sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/vd[bc]1
	```
7. Соберите `mdadm` RAID0 на второй паре маленьких разделов.
	```bash
	❯ sudo mdadm --create /dev/md1 --level=0 --raid-devices=2 /dev/vd[bc]2
	```
8. Создайте 2 независимых PV на получившихся md-устройствах.
	```bash
	❯ pvcreate /dev/md[01]
	❯ pvdisplay /dev/md[01]
	  "/dev/md0" is a new physical volume of "<2.00 GiB"
	  --- NEW Physical volume ---
	  PV Name               /dev/md0
	  VG Name
	  PV Size               <2.00 GiB
	  Allocatable           NO
	  PE Size               0
	  Total PE              0
	  Free PE               0
	  Allocated PE          0
	  PV UUID               XOvzwx-0PxV-096Y-tlBd-fGoT-2deS-jS4e26

	  "/dev/md1" is a new physical volume of "1.11 GiB"
	  --- NEW Physical volume ---
	  PV Name               /dev/md1
	  VG Name
	  PV Size               1.11 GiB
	  Allocatable           NO
	  PE Size               0
	  Total PE              0
	  Free PE               0
	  Allocated PE          0
	  PV UUID               nQi5mR-Zrh9-v4eK-EdkF-VTdz-yYsv-HB6otG
	```
9. Создайте общую volume-group на этих двух PV.
	```bash
	❯ vgcreate VG0 /dev/md[01]
	❯ vgdisplay VG0
	  --- Volume group ---
	  VG Name               VG0
	  System ID
	  Format                lvm2
	  Metadata Areas        2
	  Metadata Sequence No  1
	  VG Access             read/write
	  VG Status             resizable
	  MAX LV                0
	  Cur LV                0
	  Open LV               0
	  Max PV                0
	  Cur PV                2
	  Act PV                2
	  VG Size               <3.11 GiB
	  PE Size               4.00 MiB
	  Total PE              796
	  Alloc PE / Size       0 / 0
	  Free  PE / Size       796 / <3.11 GiB
	  VG UUID               dpaQpw-HSDN-Z3fT-aoPU-r1kI-OEpq-pq9wcs
	```
10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.
	```bash
	❯ lvcreate --name LV0 --type linear --size 100M VG0 /dev/md1
	```
11. Создайте `mkfs.ext4` ФС на получившемся LV.
	```bash
	❯ mkfs.ext4 /dev/VG0/LV0
	```
12. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.
	```bash
	❯ mkdir /tmp/new && mount /dev/VG0/LV0 /tmp/new
	```
13. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.
	```bash
	❯ wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz
	```
14. Прикрепите вывод `lsblk`.
	```bash
	❯ lsblk
	NAME          MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
	loop0           7:0    0   48M  1 loop  /snap/snapd/16778
	loop1           7:1    0  103M  1 loop  /snap/lxd/23541
	loop2           7:2    0 63.2M  1 loop  /snap/core20/1623
	sr0            11:0    1    4M  0 rom
	vda           252:0    0    4G  0 disk
	├─vda1        252:1    0  3.9G  0 part  /
	├─vda14       252:14   0    4M  0 part
	└─vda15       252:15   0  106M  0 part  /boot/efi
	vdb           252:16   0  2.6G  0 disk
	├─vdb1        252:17   0    2G  0 part
	│ └─md0         9:0    0    2G  0 raid1
	└─vdb2        252:18   0  573M  0 part
	  └─md1         9:1    0  1.1G  0 raid0
		└─VG0-LV0 253:0    0  100M  0 lvm   /tmp/new
	vdc           252:32   0  2.6G  0 disk
	├─vdc1        252:33   0    2G  0 part
	│ └─md0         9:0    0    2G  0 raid1
	└─vdc2        252:34   0  573M  0 part
	  └─md1         9:1    0  1.1G  0 raid0
		└─VG0-LV0 253:0    0  100M  0 lvm   /tmp/new
	```
15. Протестируйте целостность файла:

    ```bash
    ❯ gzip -t /tmp/new/test.gz
    ❯ echo $?
    0
    ```
16. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.
	```bash
	❯ pvmove /dev/md1 /dev/md0
	  /dev/md1: Moved: 44.00%
  	  /dev/md1: Moved: 100.00%
	```
17. Сделайте `--fail` на устройство в вашем RAID1 md.
	```bash
	❯ mdadm /dev/md0 --fail /dev/vdb1
	mdadm: set /dev/vdb1 faulty in /dev/md0
	```
18. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.
	```bash
	❯ dmesg -e | tail
	[Sep24 21:08] md/raid1:md0: not clean -- starting background reconstruction
	[  +0.000003] md/raid1:md0: active with 2 out of 2 mirrors
	[  +0.000010] md0: detected capacity change from 0 to 4188160
	[  +0.002409] md: resync of RAID array md0
	[ +10.228247] md: md0: resync done.
	[ +10.423303] md1: detected capacity change from 0 to 2336768
	[Sep24 21:13] EXT4-fs (dm-0): mounted filesystem with ordered data mode. Opts: (null). Quota mode: none.
	[Sep24 21:15] dm-1: detected capacity change from 204800 to 8192
	[Sep24 21:16] md/raid1:md0: Disk failure on vdb1, disabling device.
				  md/raid1:md0: Operation continuing on 1 devices.
	```
19. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

    ```bash
    ❯ gzip -t /tmp/new/test.gz
   	❯ echo $?
    0
    ```
20. Погасите тестовый хост, `vagrant destroy`.
	```bash
	Done
	```
---