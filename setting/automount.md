Auto Mount and NFS with Infiniband
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



HeadNode에 Infinibnand를 이용한 nfs export하는 방법
--------------------------------------------------
다음과 같은 설정을 통해 부팅시 nfs를 설정할 수 있다.
```
[sms] vim /etc/exports
/home *(rw,no_subtree_check,fsid=10,no_root_squash)
/opt/ohpc/pub *(ro,no_subtree_check,fsid=11)
/tank/SFS *(rw,no_subtree_check,fsid=12,no_root_squash)
/tank/SLS *(rw,no_subtree_check,fsid=13,no_root_squash)
```
아래 /tank/SFS행과 /tank/SLS행을 추가해준다.
```
[sms] export -a
```
export를 해준다.



```
[sms] vim /etc/rc.d/rc.local
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
service opensm start # infiniband start
modprobe svcrdma # rdma service module
service nfs start # nfs start 
echo "rdma 20049" > /proc/fs/nfsd/portlist # 20049 port for rdma

[sms] chmod +x /etc/rc.d/rc.local # rc.local이 부팅시 실행될수있도록 권한변경
```





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


```
[bridge*] chmod +x /etc/rc.d/rc.local # rc.local이 부팅시 실행될수있도록 권한변경
```
다음과 같은 코드로 변경하여 mount를 해준다
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

참고로 마운트하기 전에 폴더는 미리 만들어놓아야 한다

**xprtrdma가 없는 모듈이라고 나올 수 있다. 이것은 처음에 warewulf에서 커널버전이 headnode와 같지 않아서이다. HeadNode의 `uname -r`을 확인하고 그 버전에 맞는 커널을 warewulf에서 설치해주어야 한다.**
**$CHROOT에 있는 파일들을 모두 제거하고 커널을 HeadNode버전에 맞게 설치하는 것을 추천한다**

