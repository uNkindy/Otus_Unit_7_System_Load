### Домашнее задание №7 (Работа с загрузчиком)
#### Попасть в систему без пароля несколькими способами.
* Создан Vagrantfile с следующим тюнингом (прилагается):
  - Добавлен IDE диск на 1Gb;
  - Добавлена загрузка GUI;
  - За основу взят дистрибутив Centos 7.
___
* __Способ 1__. init=/bin/
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
* __Способ 2__. rd.break
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

* __Способ 3__. rw init=/sysroot/bin/sh
  - В параметры загрузки добавлен параметр __rw init=/sysroot/bin/sh__;
  - Система загружена сразу в режиме __rw__:
```console
sh-4.2# touch /home/vagrant/test2 && echo "test2" > /home/vagrant/test
sh-4.2# cat /home/vagrant/test2
test2
sh-4.2#
```
___
__Разница между методами получения shell:__ 
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
* Изменим название VG в [/etc/fstab](), [/etc/default/grub](), [/boot/grub2/grub.cfg]();
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
* 
