### Домашнее задание №7 (Работа с загрузчиком)
#### Попасть в систему без пароля несколькими способами.
* Создан Vagrantfile с следующим тюнингом (прилагается):
  - Добавлен IDE диск на 1Gb;
  - Добавлена загрузка GUI;
  - За основу взят дистрибутив Centos 7.
___
* __Способ 1__. init=/bin/sh => при этом способе грузимся сразу в рутовую консоль, перемонтируем систему в __rw__
  - В параметры загрузки добавлен параметр __init=/bin/sh__ (стерты все параметры до монтирования);
  - Проверена возможность записи файлов в директорию __/home/vagrant/__:
```console
sh-4.2# touch /home/vagrant/test && echo "test" > /home/vagrant/test
touch: cannot touch '/home/vagrant/test': Read-only file system
```
  - Перемонтировал ФС в __rw__:
```console
sh-4.2# mount -o remount, rw /
sh-4.2# touch /home/vagrant/test && echo "test" > /home/vagrant/test
sh-4.2# cat /home/vagrant/test
test
sh-4.2#
```
* __Способ 2__. rd.break => при этом способе мы прерываем монтирование рутовой файловой системы,рутовая система будет находиться в директории __/sysroot__ (запускаем шелл перед процедурой pivot_root())
  - В параметры загрузки добавлен параметр __rd.break__, система загружена в emergency mode.
  - Перемонтируем корневую систему в __rw__ и меняем пароль __root__:
```console
switch_root:/# mount -o remount, rw /sysroot
switch_root:/# chroot /sysroot
switch_root:/# passwd root
Changing password for user root.
New password:
BAD PASSWORD: The password fails the dictionary check - it is based on a dictionary word
Retype new password:
passwd: all authentification tokens updated successfully.
sh-4.2# touch /.autorelabel
sh-4.2#
```
  - ВМ перезагружена, зашел в терминал под пользьвателем __root__, пароль __changeme__.

* __Способ 3__. rw init=/sysroot/bin/sh => при этом способе система загружается в emergency mode.
  - В параметры загрузки добавлен параметр __rw init=/sysroot/bin/sh__;
  - Система загружена сразу в режиме __rw__:
```console
sh-4.2# touch /home/vagrant/test2 && echo "test2" > /home/vagrant/test
sh-4.2# cat /home/vagrant/test2
test2
sh-4.2#
```
___
#### 2. Смонтировать систему в LVM, после чего переименовать в VG.
* Перемонтируем систему в rw, переименуем Volume Group в NewVolGroup00
```console
[root@localhost vagrant]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g    0 
[root@localhost vagrant]# vgrename VolGroup00 NewVolGroup00
Cannot archive volume group metadata for VolGroup00 to read-only filesystem.
[root@localhost vagrant]# mount -o remount,rw /         
[root@localhost vagrant]# vgrename VolGroup00 NewVolGroup00
Volume group "VolGroup00" successfully renamed to "NewVolGroup00"
```
* Изменим название VG в [/etc/fstab](https://github.com/uNkindy/Otus_Unit_7_System_Load/blob/main/fstab), [/etc/default/grub](https://github.com/uNkindy/Otus_Unit_7_System_Load/blob/main/grub), [/boot/grub2/grub.cfg](https://github.com/uNkindy/Otus_Unit_7_System_Load/blob/main/grub.cfg);
* Создадим __initrd__, ребутнемся и проверим имя volume group
```console
[root@localhost vagrant]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
 *** Creating image file ***
 *** Creating image file done ***
 *** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
 [root@localhost vagrant]# reboot
 ```
 ```console
 [root@localhost vagrant]# vgs
 VG             #PV #LV #SN Attr   VSize   VFree
 NewVolGroup00   1   2   0 wz--n- <38.97g    0 
 ```
___
#### 3. Добавление модуля в initrd
* Создал папку в __/usr/lib/drakut/module.d/01test__
* Добавил 2 скрипта __test.sh, module-setup.sh__ и сделал данный скрипты исполняемыми:
```console
[root@localhost 01test]# chmod +x test.sh
[root@localhost 01test]# chmod +x module-setup.sh
[root@localhost 01test]# ls -l /usr/lib/dracut/modules.d/01test/
total 8
-rwxr-xr-x. 1 root root 126 Sep  6 15:44 module-setup.sh
-rwxr-xr-x. 1 root root 333 Sep  6 16:42 test.sh
[root@localhost 01test]# 
```
* Пересобрал образ initrd
```console
[root@localhost 01test]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
.
.
.
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-1127.el7.x86_64.img' done ***
[root@localhost 01test]# 
```
```console 
[root@localhost 01test]# dracut -f -v
.
.
.
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-1127.el7.x86_64.img' done ***
[root@localhost 01test]# 
```
* Проверил загрузку модуля 


