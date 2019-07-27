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
UUID, mount location, file system, option(defaults로 하면 무난) 0 0
의 형식으로 입력해주고 재부팅을 한다.

ComputeNode에 HeadNode의 드라이브를 mount하는 방법
------------------------------------------------

