#Попасть в систему без пароля несколькими способами

#1. Способ заходим в загрузчик до старта системы (перед тем, как начнется загрузка нужно успеть нажать на клашиву e)
#после чего добавить к строке linux16 init=/bin/sh и удалить console=tty0,console=ttyS0,115200n8

           Ctrl+X


#Далее перемонтируем корень

           mount -o remount,rw /


#далее меняем пароль командой

           passwd


#проверяем включен для SELUX 
  
           cat /etc/selinux/config


#если SELINUX=enforcing, то заменяем или на permissive или disabled и перезагружаемся


           vi /etc/selinux/config

           reboot

#2. Способ заходим в загрузчик до старта системы (перед тем, как начнется загрузка нужно успеть нажать на клашиву e)
#после чего добавить к строке linux16 rd.break и удалить console=tty0,console=ttyS0,115200n8

           Ctrl+X
    

#Далее перемонтируем корень

           mount -o remount,rw /sysroot

           chroot /sysroot

           passwd root

     
#После смены пароля необходимо создать скрытый файл .autorelabel в /, выполнив

            touch /.autorelabel


#После чего можно перезагружаться и заходить в систему с новым паролем.

           reboot


#3.Способ. Чтобы после сброса root-пароля изменения были сохранены, необходимо заменить ro на rw (read-write) — режим чтение-запись.
#заходим в загрузчик до старта системы (перед тем, как начнется загрузка нужно успеть нажать на клашиву e)
#после чего изменить в строке linux16 ro на rw init=/sysroot/bin/sh и удалить console=tty0,console=ttyS0,115200n8

           Ctrl+X

#Для установки нового root-пароля введите команду

           chroot /sysroot

           passwd root

#После этого, обновляем параметры SELinux командой

           touch /.autorelabel


           exit

           reboot



#Установить систему с LVM, после чего переименовать VG

#Первым делом посмотрим на текущее состояние системы

            vgs


#Далее можем поменять название VG

            vgrename VolGroup00 OtusRoot


#Смотрим новое название 

           ls -l /dev/mapper


#Далее будем править файлы, чтобы в дальнейшем мы могли загрузиться.(/etc/fstab, /etc/default/grub,  /boot/grub2/grub.cfg)
#заходим в эти файлы  и меняем старое название на новое
#работаем через nano    

           yum install nano.x86_64

           nano /etc/fstab

           nano /etc/default/grub

           nano /boot/grub2/grub.cfg


#Далее пересоздаем initrd image

           mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)


#Перезагружаем ПК, теперь у нас система загружается с новым названием VG, проверяем:

           vgs


#Скрипты модулей хранятся в каталоге /usr/lib/dracut/modules.d/. Для того чтобы добавить свой модуль создаем там папку с именем 01test.

            mkdir /usr/lib/dracut/modules.d/01test

#Переходим в директорию

            cd /usr/lib/dracut/modules.d/01test/


#Далее создаем там 2 скрипта - module-setup.sh и test.sh

           touch module-setup.sh

           touch module-test.sh


#Прописываем в module-setup.sh

           #!/bin/bash

           check() {
           return 0
       }

           depends() {
           return 0
       }

           install() {
           inst_hook cleanup 00 "${moddir}/module-test.sh"
       }


#Прописываем в module-test.sh

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


#делаем их исполняемыми

           chmod +x module-setup.sh

           chmod +x module-test.sh

#Далее выполняем команду

            dracut -f -v

#Можно посмотреть какие модули загружены в образ

            lsinitrd -m /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img | grep test

#После чего можно пойти двумя путями для проверки:
#1.Перезагрузиться и руками выключить опции rghb и quiet и увидеть вывод
#2.Либо отредактировать grub.cfg убрав эти опции

#Редактируем grub.cfg

           nano /boot/grub2/grub.cfg

           reboot







