Auto Mount
==========

HeadNode에 새로운 드라이브를 설치하는 방법
---------------------------------------


```
[sms] vim /etc/fstab
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=ffa0b4b4-aa17-48f7-8637-60eded285c74 /boot                   xfs     defaults        0 0
UUID=72E7-9A3D          /boot/efi               vfat    umask=0077,shortname=winnt 0 0
/dev/mapper/centos-home /home                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
```

fstab은 리눅스가 부팅시 mount하는 정보를 담는 파일이다.
위의 상태에서 두개의 디스크를 추가해보겠다.

```
[sms] vim /etc/fstab
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=ffa0b4b4-aa17-48f7-8637-60eded285c74 /boot                   xfs     defaults        0 0
UUID=72E7-9A3D          /boot/efi               vfat    umask=0077,shortname=winnt 0 0
/dev/mapper/centos-home /home                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
UUID=c1e36153-e18b-4115-97de-2c8f7a225104 /tank/SFS ext4 defaults 0 0
UUID=df2723ce-bdd7-48f4-bcf8-607eef880aa0 /tank/SLS ext4 defaults 0 0
```
UUID, mount location, file system, option(defaults로 하면 무난), 0, 0
의 형식으로 입력해주고 재부팅을 한다.

ComputeNode에 Infiniband를 이용한 nfs mount를 하는 방법
------------------------------------------------------

ComputeNode에 fstab을 이용하여 하는 방법도 있지만 이 방법보다는 안정적인 rc.local을 이용한 방법을 추천한다.

우선 rc.local을 열어보면
```
[bridge*] vim /etc/rc.d/rc.local
#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
#
# It is highly advisable to create own systemd services or udev rules
# to run scripts during boot instead of using this file.
#
# In contrast to previous versions due to parallel execution during boot
# this script will NOT be run after all other services.
#
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.

touch /var/lock/subsys/local
```
설명을 자세히 읽어보면 마지막에 chmod로 rc.local이 executable해야 이 파일이 실행된다고 적혀있다.
```
[bridge*] chmod +x /etc/rc.d/rc.local
```
이제 mount를 해준다.
```
[bridge*] vim /etc/rc.d/rc.local
#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
#
# It is highly advisable to create own systemd services or udev rules
# to run scripts during boot instead of using this file.
#
# In contrast to previous versions due to parallel execution during boot
# this script will NOT be run after all other services.
#
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.

#touch /var/lock/subsys/local
modprobe xprtrdma
mount -o rdma,port=20049 192.168.100.100:/tank/SFS /tank/SFS
mount -o rdma,port=20049 192.168.100.100:/tank/SLS /tank/SLS
```
이제 부팅시마다 rc.local이 실행되어 SFS와 SLS를 마운트해준다.





