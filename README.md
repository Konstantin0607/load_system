#������� � ������� ��� ������ ����������� ���������

#1. ������ ������� � ��������� �� ������ ������� (����� ���, ��� �������� �������� ����� ������ ������ �� ������� e)
#����� ���� �������� � ������ linux16 init=/bin/sh � ������� console=tty0,console=ttyS0,115200n8

           Ctrl+X


#����� ������������� ������

           mount -o remount,rw /


#����� ������ ������ ��������

           passwd


#��������� ������� ��� SELUX 
  
           cat /etc/selinux/config


#���� SELINUX=enforcing, �� �������� ��� �� permissive ��� disabled � ���������������


           vi /etc/selinux/config

           reboot

#2. ������ ������� � ��������� �� ������ ������� (����� ���, ��� �������� �������� ����� ������ ������ �� ������� e)
#����� ���� �������� � ������ linux16 rd.break � ������� console=tty0,console=ttyS0,115200n8

           Ctrl+X
    

#����� ������������� ������

           mount -o remount,rw /sysroot

           chroot /sysroot

           passwd root

     
#����� ����� ������ ���������� ������� ������� ���� .autorelabel � /, ��������

            touch /.autorelabel


#����� ���� ����� ��������������� � �������� � ������� � ����� �������.

           reboot


#3.������. ����� ����� ������ root-������ ��������� ���� ���������, ���������� �������� ro �� rw (read-write) � ����� ������-������.
#������� � ��������� �� ������ ������� (����� ���, ��� �������� �������� ����� ������ ������ �� ������� e)
#����� ���� �������� � ������ linux16 ro �� rw init=/sysroot/bin/sh � ������� console=tty0,console=ttyS0,115200n8

           Ctrl+X

#��� ��������� ������ root-������ ������� �������

           chroot /sysroot

           passwd root

#����� �����, ��������� ��������� SELinux ��������

           touch /.autorelabel


           exit

           reboot



#���������� ������� � LVM, ����� ���� ������������� VG

#������ ����� ��������� �� ������� ��������� �������

            vgs


#����� ����� �������� �������� VG

            vgrename VolGroup00 OtusRoot


#������� ����� �������� 

           ls -l /dev/mapper


#����� ����� ������� �����, ����� � ���������� �� ����� �����������.(/etc/fstab, /etc/default/grub,  /boot/grub2/grub.cfg)
#������� � ��� �����  � ������ ������ �������� �� �����
#�������� ����� nano    

           yum install nano.x86_64

           nano /etc/fstab

           nano /etc/default/grub

           nano /boot/grub2/grub.cfg


#����� ����������� initrd image

           mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)


#������������� ��, ������ � ��� ������� ����������� � ����� ��������� VG, ���������:

           vgs


#������� ������� �������� � �������� /usr/lib/dracut/modules.d/. ��� ���� ����� �������� ���� ������ ������� ��� ����� � ������ 01test.

            mkdir /usr/lib/dracut/modules.d/01test

#��������� � ����������

            cd /usr/lib/dracut/modules.d/01test/


#����� ������� ��� 2 ������� - module-setup.sh � test.sh

           touch module-setup.sh

           touch module-test.sh


#����������� � module-setup.sh

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


#����������� � module-test.sh

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


#������ �� ������������

           chmod +x module-setup.sh

           chmod +x module-test.sh

#����� ��������� �������

            dracut -f -v

#����� ���������� ����� ������ ��������� � �����

            lsinitrd -m /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img | grep test

#����� ���� ����� ����� ����� ������ ��� ��������:
#1.��������������� � ������ ��������� ����� rghb � quiet � ������� �����
#2.���� ��������������� grub.cfg ����� ��� �����

#����������� grub.cfg

           nano /boot/grub2/grub.cfg

           reboot







