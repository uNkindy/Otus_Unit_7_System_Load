### Домашнее задание №7 (Работа с загрузчиком)
#### Попасть в систему без пароля несколькими способами.
* Создан Vagrantfile с следующим тюнингом (прилагается):
  - Добавлен IDE диск на 1Gb;
  - Добавлена загрузка GUI;
  - За основу взят дистрибутив Centos 7.
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

* __Способ 3. rw init=/sysroot/bin/sh
  - В параметры загрузки добавлен параметр __rw init=/sysroot/bin/sh__;
  - Система загружена сразу в режиме __rw__:
 sh-4.2# touch /home/vagrant/test2 && echo "test2" > /home/vagrant/test
sh-4.2# cat /home/vagrant/test2
test2
sh-4.2#
```
#### 2. Смонтировать систему в LVM, после чего переименовать в VG.
* 

