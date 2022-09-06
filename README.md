### Домашнее задание №7 (Работа с загрузчиком)
#### Попасть в систему без пароля несколькими способами.
* Создан Vagrantfile с следующим тюнингом (прилагается):
  - Добавлен IDE диск на 1Gb;
  - Добавлена загрузка GUI;
  - За основу взят дистрибутив Centos 7.
* __Способ 1__. init=/bin/
  - В параметры загрузки добавлен параметр __ init=/bin/sh__ (стерты все параметры до монтирования);
  - Проверена возможность записи файлов в директорию __/bome/vagrant/__:
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
switch_root:/# 

