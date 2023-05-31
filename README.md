		 Домашнее задание №7

 			Задание

- Попасть в систему без пароля несколькими способами
- Установить систему с LVM, после чего переименовать VG
- Добавить модуль в initrd

1. Пробуем попасть в систему через GUI VirtualBox. Перед этим возьмем из 
предыдущих занятий vagrantfile c LVM, в котором у нас будет centos7

2. Пдключаемся к virtualbox и запускаем машину.

Способ 1

- При выборе ядра для загрузки нажать "e" - и меняем настройки запуска в 
   в строке начинающейся с "linux16", в конец строки дописываем init=/bin/sh 
   и убираем из этой же строчки (console=tty0 console=ttyS0,115200n8)
   далее жмем F10 либо ctrl x

- Ждем пока система загрузится.

- Так как рутовая файловая система при этом монтируется в режиме Read-Only,
   мы перемонтируем ее в режим Read-Write командой mount -o remount,rw /

- Создадим файл и проверим что права на запись в каталог etc есть.

- Пример выполнения задания на скриншоте

![1.2 Image](https://github.com/[ilyaDragunov/dz-7/blob/main/1.2.JPG?raw=true)]

Способ 2

- При выборе ядра для загрузки нажать "e" - и меняем настройки запуска в
   в строке начинающейся с "linux16", в конец строки дописываем rd.break
   и убираем из этой же строчки (console=tty0 console=ttyS0,115200n8)
   далее жмем F10 либо ctrl x

- Ждем пока система загрузится.

- Попадаем в emergency mode. Наша корневая файловая система смонтирована
  (опять же в режиме Read-Only,далее будет пример, как попасть в нее и
   поменять пароль администратора

- Пример выполнения задания на скриншоте


[root@otuslinux ~]# mount -o remount,rw /sysroot
[root@otuslinux ~]# chroot /sysroot
[root@otuslinux ~]# passwd root
[root@otuslinux ~]# touch /.autorelabel

- После перегружаемся и заходим в систему с новым паролем.

- Пример выполнения задания на скриншоте

Способ 3 

- При выборе ядра для загрузки нажать "e" - и меняем настройки запуска в
   в строке начинающейся с "linux16", заменяем ro на rw init=/sysroot/bin/sh
   и убираем из этой же строчки (console=tty0 console=ttyS0,115200n8)
   далее жмем F10 либо ctrl x

- Ждем пока система загрузится.

- Для примера произвел сброс пароля.

- Пример выполнения задания на скриншоте


3. Устанавливаем систему с LVM, после чего переименовываем VG. В качестве готовой системы я 
   взял vagrantfile с предыдущего занятия с LVM

4. Первым делом посмотрим текущее состояние системы

```

[vagrant@lvm ~]$ sudo vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g    0

```

5. Переименуем нашу VL

```

[vagrant@lvm ~]$ sudo vgrename VolGroup00 OtusRoot
  Volume group "VolGroup00" successfully renamed to "OtusRoot"

```

6. Далее правим /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg. 
   Везде заменяем старое название на новое. 

7. Пересоздаем initrd image, чтобы он знал новое название Volume Group

```

[root@lvm ~]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
Executing: /sbin/dracut -f -v /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img 3.10.0-862.2.3.el7.x86_64
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
*** Including module: bash ***
*** Including module: nss-softokn ***
*** Including module: i18n ***
*** Including module: drm ***
*** Including module: plymouth ***
*** Including module: dm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 60-persistent-storage-dm.rules
Skipping udev rule: 55-dm.rules
*** Including module: kernel-modules ***
Omitting driver floppy
*** Including module: lvm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 56-lvm.rules
Skipping udev rule: 60-persistent-storage-lvm.rules
*** Including module: qemu ***
*** Including module: resume ***
*** Including module: rootfs-block ***
*** Including module: terminfo ***
*** Including module: udev-rules ***
Skipping udev rule: 40-redhat-cpu-hotplug.rules
Skipping udev rule: 91-permissions.rules
*** Including module: biosdevname ***
*** Including module: systemd ***
*** Including module: usrmount ***
*** Including module: base ***
*** Including module: fs-lib ***
*** Including module: shutdown ***
*** Including modules done ***
*** Installing kernel module dependencies and firmware ***
*** Installing kernel module dependencies and firmware done ***
*** Resolving executable dependencies ***
*** Resolving executable dependencies done***
*** Hardlinking files ***
*** Hardlinking files done ***
*** Stripping files ***
*** Stripping files done ***
*** Generating early-microcode cpio image contents ***
*** No early-microcode cpio image needed ***
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
[root@lvm ~]# reboot
Connection to 127.0.0.1 closed by remote host.
user@ubuntu-vm:~/DZ-OTUS/DZ-7$ vagrant ssh
user@ubuntu-vm:~/DZ-OTUS/DZ-7$
user@ubuntu-vm:~/DZ-OTUS/DZ-7$
user@ubuntu-vm:~/DZ-OTUS/DZ-7$
user@ubuntu-vm:~/DZ-OTUS/DZ-7$ vagrant ssh
Last login: Tue May 30 12:01:02 2023 from 10.0.2.2
[vagrant@lvm ~]$ lsblk
NAME                  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                     8:0    0   40G  0 disk
├─sda1                  8:1    0    1M  0 part
├─sda2                  8:2    0    1G  0 part /boot
└─sda3                  8:3    0   39G  0 part
  ├─OtusRoot-LogVol00 253:0    0 37.5G  0 lvm  /
  └─OtusRoot-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]

```

8. Скрипты модулей хранятся в каталоге /usr/lib/dracut/modules.d/.
    Для того, чтобы добавить свой модуль, создаем там папку с именем 01test:

```

[root@lvm ~]# mkdir /usr/lib/dracut/modules.d/01test

```

9. В нее поместим два скрипта:

- module-setup.sh - который устанавливает модуль и вызывает скрипт 
- test.sh 2. test.sh - собственно сам вызываемый скрипт, в нём у нас 
рисуется пингвинчик

```
module-setup.sh

#!/bin/bash

check() {
    return 0
}

depends() {
    return 0
}

install() {
    inst_hook cleanup 00 "${moddir}/test.sh"
}

test.sh

#!/bin/bash

exec 0<>/dev/console 1<>/dev/console 2<>/dev/console
cat <<'msgend'

Hello! You are in dracut module!

 ___________________
< I'm dracut module >
 -------------------
   \
    \
        .--.
       |o_o |
       |:_/ |
      //   \ \
     (|     | )
    /'\_   _/`\
    \___)=(___/
msgend
sleep 10
echo " continuing...."


[root@lvm ~]# touch module-setup.sh
[root@lvm ~]# touch test.sh
[root@lvm ~]# nano module-setup.sh
[root@lvm ~]# nano test.sh
[root@lvm ~]# cp module-setup.sh /usr/lib/dracut/modules.d/01test/
[root@lvm ~]# cp test.sh /usr/lib/dracut/modules.d/01test/

[root@lvm ~]# ll /usr/lib/dracut//modules.d/01test/
total 8
-rw-r--r--. 1 root root 123 May 30 13:11 module-setup.sh
-rw-r--r--. 1 root root 323 May 30 13:12 test.sh

```

10. Пересобираем образ initrd

```
[root@lvm ~]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
Executing: /sbin/dracut -f -v /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img 3.10.0-862.2.3.el7.x86_64
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
*** Including module: bash ***
*** Including module: test ***
*** Including module: nss-softokn ***
*** Including module: i18n ***
*** Including module: drm ***
*** Including module: plymouth ***
*** Including module: dm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 60-persistent-storage-dm.rules
Skipping udev rule: 55-dm.rules
*** Including module: kernel-modules ***
Omitting driver floppy
*** Including module: lvm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 56-lvm.rules
Skipping udev rule: 60-persistent-storage-lvm.rules
*** Including module: qemu ***
*** Including module: resume ***
*** Including module: rootfs-block ***
*** Including module: terminfo ***
*** Including module: udev-rules ***
Skipping udev rule: 40-redhat-cpu-hotplug.rules
Skipping udev rule: 91-permissions.rules
*** Including module: biosdevname ***
*** Including module: systemd ***
*** Including module: usrmount ***
*** Including module: base ***
*** Including module: fs-lib ***
*** Including module: shutdown ***
*** Including modules done ***
*** Installing kernel module dependencies and firmware ***
*** Installing kernel module dependencies and firmware done ***
*** Resolving executable dependencies ***
*** Resolving executable dependencies done***
*** Hardlinking files ***
*** Hardlinking files done ***
*** Stripping files ***
*** Stripping files done ***
*** Generating early-microcode cpio image contents ***
*** No early-microcode cpio image needed ***
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***

```

10. Проверим, какие модули у нас добавлены в образ

```

[root@lvm ~]$ lsinitrd -m /boot/initramfs-$(uname -r).img | grep test 
test

```
11. Выключаем при загрузке опции rghb quiet и проверяем что отбратился пингвин.

- скриншот прилагается


# dz-7
