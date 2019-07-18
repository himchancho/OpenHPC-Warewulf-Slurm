주로 설명이 위에 있고 그림이나 코드가 아래에 

# OpenHPC-Warewulf-Slurm
Using OpenHPC-Warewulf-Slurm, make High Performance Computer


## 0. H/W Requirements for this recipe
1. Head Node (DELL R720)  
  1-1. SSD (리뷰안 dx2200 * 2 , 리뷰안 960x * 2)
2. Compute Node * 10 (Quanta Computer Windmill)  
  2-1. Infiniband card  
  2-2. SSD 
3. IP router (iptime)
4. Infiniband Switch (mellanox)

## 1. Installation Centos7.6
1. Install CentOS 7.6 on Head Node
centos7.6을 usb를 이용하여 부팅하면 다음과 같은 화면이 나타난다.  
<img src = "./img/centos_install.png" width="600" height="450">  

INSTALLATION SOURCE에서 설치할 디스크와 파티션을 잘 나누어주고  
SOFTWARE SELECTION에서 GUI만 선택하고 (추가 애드온 없이) DONE한다  

## 2. Installation OpenHPC
### 2-1 . Export Environment Variables
여기서 사용할 환경 변수들에 대한 설명이다  
~~줄그어진~~ 변수들은 설명서에는 있지만 여기서는 사용하지 않는 것들이다.  

• ${sms name} # Hostname for SMS server  
• ${sms ip} # Internal IP address on SMS server   
• ${sms eth internal} # Internal Ethernet interface on SMS   
• ${eth provision} # Provisioning interface for computes   
• ${internal netmask} # Subnet netmask for internal network   
• ${ntp server} # Local ntp server for time synchronization   
• ~~${bmc username} # BMC username for use by IPMI~~  
• ~~${bmc password} # BMC password for use by IPMI~~  
• ${num computes} # Total # of desired compute nodes   
• ${c ip[0]}, ${c ip[1]}, ... # Desired compute node addresses   
• ${c bmc[0]}, ${c bmc[1]}, ... # BMC addresses for computes  
• ${c mac[0]}, ${c mac[1]}, ... # MAC addresses for computes  
• ${c name[0]}, ${c name[1]}, ... # Host names for computes   
• ${compute regex} # Regex matching all compute node names (e.g. “c*”)   
• ${compute prefix} # Preﬁx for compute node names (e.g. “c”) Optional  
• ~~${sysmgmtd host} # BeeGFS System Management host name~~  
• ~~${mgs fs name} # Lustre MGS mount name~~  
• ${sms ipoib} # IPoIB address for SMS server  
• ${ipoib netmask} # Subnet netmask for internal IPoIB  
• ${c ipoib[0]}, ${c ipoib[1]}, ... # IPoIB addresses for computes  
• ${kargs} # Kernel boot arguments  
• ~~${nagios web password} # Nagios web access password~~  

이런 변수들은 매번 치는게 어려우니 텍스트파일에 저장해놓고 필요할 때 마다 복사해서 export하도록 한다. 

### 2-2. Route setting
**가장 앞서 공유기의 DHCP를 비활성화 한다**  
앞으로 pxe부팅을 하게 될 것인데 그에 앞서 라우팅 세팅을 한다.
<img src = "./img/route.png">  
이 세팅을 하는 이유는 각각의 computenode가 pxe boot 할 때 gateway에 있는 서버에 접속하여 부팅이미지를 다운받게 되는데 그 gateway를 헤드노드로 지정할 수 있도록 해야한다.
```
route -add -net 192.168.0.0 netmask 255.255.255.0 dev em2
```
이 코드가 아니더라도 GUI의 setting에서 network설정을 통해 route를 설정하는 것도 좋다

### 2-3. Install Base Operating System
호스트를 등록하고 방화벽을 해제한다.
```
[sms]# echo ${sms_ip} ${sms_name} >> /etc/hosts
[sms]# systemctl disable firewalld 
[sms]# systemctl stop firewalld
```

### 2-4. Install OpenHPC Components

OpenHPC 설치
```
[sms]# yum install http://build.openhpc.community/OpenHPC:/1.3/CentOS_7/x86_64/ohpc-release-1.3-1.el7.x86_64.rpm
```
Add provisioning services on master node
```
[sms]# yum -y install ohpc-base
[sms]# yum -y install ohpc-warewul
```
NTP setup
```
[sms]# systemctl enable ntpd.service
[sms]# echo "server ${ntp_server}" >> /etc/ntp.conf 
[sms]# systemctl restart ntpd
```
Add resource management services on master node
여기서 나오는 slurm.conf가 나중에 slurm을 세팅하는 중요한 파일이 된다. 지금은 넘어가도록 한다  
```
[sms]# yum -y install ohpc-slurm-server
[sms]# perl -pi -e "s/ControlMachine=\S+/ControlMachine=${sms_name}/" /etc/slurm/slurm.conf
```
Optionally add InﬁniBand support services on master node  
말그대로 인피니밴드가 있다면 설치하면 된다
```
[sms]# yum -y groupinstall "InfiniBand Support" 
[sms]# yum -y install infinipath-psm
[sms]# systemctl start rdma
```
인피니밴드를 설치하였다면 opensm도 설치해주자
```
[sms]# yum -y install opensm
[sms]# service opensm start
```
인피니밴드 네트워크 세팅
```
[sms]# cp /opt/ohpc/pub/examples/network/centos/ifcfg-ib0 /etc/sysconfig/network-scripts
[sms]# perl -pi -e "s/master_ipoib/${sms_ipoib}/" /etc/sysconfig/network-scripts/ifcfg-ib0 
[sms]# perl -pi -e "s/ipoib_netmask/${ipoib_netmask}/" /etc/sysconfig/network-scripts/ifcfg-ib0
[sms]# ifup ib0
```
Complete basic Warewulf setup for master node
```
[sms]# perl -pi -e "s/device = eth1/device = ${sms_eth_internal}/" /etc/warewulf/provision.conf
[sms]# perl -pi -e "s/^\s+disable\s+= yes/ disable = no/" /etc/xinetd.d/tftp
[sms]# ifconfig ${sms_eth_internal} ${sms_ip} netmask ${internal_netmask} up
[sms]# systemctl restart xinetd
[sms]# systemctl enable mariadb.service
[sms]# systemctl restart mariadb
[sms]# systemctl enable httpd.service
[sms]# systemctl restart httpd
[sms]# systemctl enable dhcpd.service
```
### 2-5. Deﬁne compute image for provisioning (ComputeNode image setting)  
CHROOT는 자주 사용하므로 저장해놓고 복사 붙여넣기 하여 쉽게 사용하도록 하자  
#####2-1 env-setup에 CHROOT를 추가하였음
 wwmkchroot을 이용해 ComputeNode에 provisioning할 초기 이미지를 생성한다  
 CHROOT의 경로에 있는 폴더가 ComputeNode의 root폴더가 된다고 생각하면 된다  
```
[sms]# export CHROOT=/opt/ohpc/admin/images/centos7.6
[sms]# wwmkchroot centos-7 $CHROOT
```
Add OpenHPC components
```
[sms]# yum -y --installroot=$CHROOT install ohpc-base-compute
[sms]# cp -p /etc/resolv.conf $CHROOT/etc/resolv.conf

[sms]# yum -y --installroot=$CHROOT install ohpc-slurm-client
[sms]# yum -y --installroot=$CHROOT install ntp
[sms]# yum -y --installroot=$CHROOT install kernel
[sms]# yum -y --installroot=$CHROOT install lmod-ohpc
```
Customize system conﬁguration
```
[sms]# wwinit database
[sms]# wwinit ssh_keys
```
fstab은 linux에서 고정 마운트를 할수 있도록 하는 파일인데 후에 쓰일 것이다.
여기서는 ComputeNode의 /home 디렉토리에 HeadNode의 /home 디렉토리를 마운트하는 설정이다
```
[sms]# echo "${sms_ip}:/home /home nfs nfsvers=3,nodev,nosuid 0 0" >> $CHROOT/etc/fstab
[sms]# echo "${sms_ip}:/opt/ohpc/pub /opt/ohpc/pub nfs nfsvers=3,nodev 0 0" >> $CHROOT/etc/fstab
```
ComputeNode의 /etc/exports를 설정하면 nfs를 통해 자신의 디렉토리를 밖으로 export해줄 수 있다.  
나중에 공유할 폴더를 더 추가하고 싶다면 /etc/exports를 참조하면 된다.  
```
[sms]# echo "/home *(rw,no_subtree_check,fsid=10,no_root_squash)" >> /etc/exports 
[sms]# echo "/opt/ohpc/pub *(ro,no_subtree_check,fsid=11)" >> /etc/exports
[sms]# exportfs -a
[sms]# systemctl restart nfs-server 
[sms]# systemctl enable nfs-server
```
ComputeNode에 ntp 설정
```
[sms]# chroot $CHROOT systemctl enable ntpd 
[sms]# echo "server ${sms_ip}" >> $CHROOT/etc/ntp.conf
```
Enable InﬁniBand drivers  
여기서는 opensm을 설치할 필요가 없다 opensm은 host에만 설치한다.
```
[sms]# yum -y --installroot=$CHROOT groupinstall "InfiniBand Support"
[sms]# yum -y --installroot=$CHROOT install infinipath-psm
[sms]# chroot $CHROOT systemctl enable rdma
```
Increase locked memory limits
```
[sms]# perl -pi -e 's/# End of file/\* soft memlock unlimited\n$&/s' /etc/security/limits.conf 
[sms]# perl -pi -e 's/# End of file/\* hard memlock unlimited\n$&/s' /etc/security/limits.conf
[sms]# perl -pi -e 's/# End of file/\* soft memlock unlimited\n$&/s' $CHROOT/etc/security/limits.conf 
[sms]# perl -pi -e 's/# End of file/\* hard memlock unlimited\n$&/s' $CHROOT/etc/security/limits.conf
```
Enable ssh control via resource manage  
pam을 설정하게 되면 ComputeNode에는 HeadNode의 root계정만 ssh접속 할 수 있게 된다.
```
[sms]# echo "account required pam_slurm.so" >> $CHROOT/etc/pam.d/sshd
```
Add Ganglia monitoring 
```
[sms]# yum -y install ohpc-ganglia
[sms]# yum -y --installroot=$CHROOT install ganglia-gmond-ohpc
[sms]# cp /opt/ohpc/pub/examples/ganglia/gmond.conf /etc/ganglia/gmond.conf 
[sms]# perl -pi -e "s/<sms>/${sms_name}/" /etc/ganglia/gmond.conf
[sms]# cp /etc/ganglia/gmond.conf $CHROOT/etc/ganglia/gmond.conf 
[sms]# echo "gridname MySite" >> /etc/ganglia/gmetad.conf
[sms]# systemctl enable gmond
[sms]# systemctl enable gmetad 
[sms]# systemctl start gmond 
[sms]# systemctl start gmetad 
[sms]# chroot $CHROOT systemctl enable gmond
[sms]# systemctl try-restart httpd
```
Import ﬁles
```
[sms]# wwsh file import /etc/passwd  
[sms]# wwsh file import /etc/group  
[sms]# wwsh file import /etc/shadow

[sms]# wwsh file import /etc/slurm/slurm.conf  
[sms]# wwsh file import /etc/munge/munge.key

[sms]# wwsh file import /opt/ohpc/pub/examples/network/centos/ifcfg-ib0.ww  
[sms]# wwsh -y file set ifcfg-ib0.ww --path=/etc/sysconfig/network-scripts/ifcfg-ib0
```

### 2-6. Finalizing provisioning conﬁguration

Assemble bootstrap image
```
[sms]# export WW_CONF=/etc/warewulf/bootstrap.conf 
[sms]# echo "drivers += updates/kernel/" >> $WW_CONF
[sms]# echo "drivers += overlay" >> $WW_CONF
[sms]# wwbootstrap `uname -r`
```
Assemble Virtual Node File System (VNFS) image
```
[sms]# wwvnfs --chroot $CHROOT
```
Register nodes for provisioning
```
[sms]# echo "GATEWAYDEV=${eth_provision}" > /tmp/network.$$  
[sms]# wwsh -y file import /tmp/network.$$ --name network  
[sms]# wwsh -y file set network --path /etc/sysconfig/network --mode=0644 --uid=0  
[sms]# for ((i=0; i<$num_computes; i++)) ; do  
          wwsh -y node new ${c_name[i]} --ipaddr=${c_ip[i]} --hwaddr=${c_mac[i]} -D ${eth_provision}  
       done
```
만약 자신이 쓰는 ethernet card 이름이 eth0이 아니라면 다음을 실행한다
```
[sms]# export kargs="${kargs} net.ifnames=1,biosdevname=1"
[sms]# wwsh provision set --postnetdown=1 "${compute_regex}"
```

```
[sms]# wwsh -y provision set "${compute_regex}" --vnfs=centos7.6 --bootstrap=`uname -r` \  
  --files=dynamic_hosts,passwd,group,shadow,slurm.conf,munge.key,network
  
[sms]# for ((i=0; i<$num_computes; i++)) ; do  
    wwsh -y node set ${c_name[$i]} -D ib0 --ipaddr=${c_ipoib[$i]} --netmask=${ipoib_netmask}  
    done
[sms]# wwsh -y provision set "${compute_regex}" --fileadd=ifcfg-ib0.ww

[sms]# systemctl restart dhcpd
[sms]# wwsh pxe update
```

Optional kernel arguments

```
[sms]# export kargs="${kargs} namespace.unpriv_enable=1"
[sms]# echo "user.max_user_namespaces=15076" >> $CHROOT/etc/sysctl.conf 
[sms]# wwvnfs --chroot $CHROOT

[sms]# wwsh -y provision set "${compute_regex}" --console=ttyS1,115200
[sms]# wwsh -y provision set "${compute_regex}" --kargs="${kargs}"
```  
Optionally conﬁgure stateful provisioning
UEFI부팅으로 세팅할 것이다
후에 메뉴얼에 나와있는대로 bootlocal을 normal로 바꿔서는 안된다.
```
[sms]# yum -y --installroot=$CHROOT install grub2-efi grub2-efi-modules 
[sms]# wwvnfs --chroot $CHROOT 
[sms]# cp /etc/warewulf/filesystem/examples/efi_example.cmds /etc/warewulf/filesystem/efi.cmds 
[sms]# wwsh provision set --filesystem=efi "${compute_regex}"
[sms]# wwsh provision set --bootloader=sda "${compute_regex}"
```
**여기까지 했다면 ComputeNode에서 PXE부팅은 성공해야 한다.**


## 3. Install OpenHPC Development Components

Development Tools
```
[sms]# yum -y install ohpc-autotools
[sms]# yum -y install EasyBuild-ohpc 
[sms]# yum -y install hwloc-ohpc  
[sms]# yum -y install spack-ohpc  
[sms]# yum -y install valgrind-ohpc
```
Compilers  
```
[sms]# yum -y install gnu8-compilers-ohpc
[sms]# yum -y install llvm5-compilers-ohpc
```
MPI Stacks  
여기선 openmpi를 사용한다
```
[sms]# yum -y install openmpi3-gnu8-ohpc mpich-gnu8-ohpc
```
Performance Tools
```
[sms]# yum -y install ohpc-gnu8-perf-tools
[sms]# yum -y install ohpc-gnu8-geopm
```
Setup default development environment
```
[sms]# yum -y install lmod-defaults-gnu8-openmpi3-ohpc
```
3rd Party Libraries and Tools
```
[sms]# yum search petsc-gnu7 ohpc 
Loaded plugins: fastestmirror Loading mirror speeds from cached hostfile 
=========================== N/S matched: petsc-gnu7, ohpc =========================== 
petsc-gnu7-impi-ohpc.x86_64 : Portable Extensible Toolkit for Scientific Computation 
petsc-gnu7-mpich-ohpc.x86_64 : Portable Extensible Toolkit for Scientific Computation 
petsc-gnu7-mvapich2-ohpc.x86_64 : Portable Extensible Toolkit for Scientific Computation 
petsc-gnu7-openmpi3-ohpc.x86_64 : Portable Extensible Toolkit for Scientific Computation
```
```
[sms]# yum -y install ohpc-gnu8-serial-libs 
[sms]# yum -y install ohpc-gnu8-io-libs 
[sms]# yum -y install ohpc-gnu8-python-libs 
[sms]# yum -y install ohpc-gnu8-runtimes
```
```
[sms]# yum -y install ohpc-gnu8-mpich-parallel-libs 
[sms]# yum -y install ohpc-gnu8-openmpi3-parallel-libs
```

## 4. Resource Manager Startup
필자가 만든 ComputeNode는 10개이므로 메뉴얼에서 조금 수정하였다
```
[sms]# systemctl enable munge 
[sms]# systemctl enable slurmctld 
[sms]# systemctl start munge 
[sms]# systemctl start slurmctld
[sms]# pdsh -w $compute_prefix[1-10] systemctl start slurmd
[sms]# pdsh -w c1 "/usr/sbin/nhc-genconf -H '*' -c -" | dshbak -c
```

## 5. Run a Test Job
```
[sms]# useradd -m test
[sms]# wwsh file resync passwd shadow group
```
Interactive execution
Hello, World가 꼭 저렇게 나오는건 아니다. 자기 ComputeNode의 개수에 따라서 달라진다.
```
[sms]# su - test
[test@sms ~]$ mpicc -O3 /opt/ohpc/pub/examples/mpi/hello.c
[test@sms ~]$ srun -n 8 -N 2 --pty /bin/bash
[test@c1 ~]$ prun ./a.out
[prun] Master compute host = c1 [prun] Resource manager = slurm 
[prun] Launch cmd = mpiexec.hydra -bootstrap slurm ./a.out
Hello, world (8 procs total) 
--> Process # 0 of 8 is alive. -> c1 
--> Process # 4 of 8 is alive. -> c2 
--> Process # 1 of 8 is alive. -> c1 
--> Process # 5 of 8 is alive. -> c2 
--> Process # 2 of 8 is alive. -> c1 
--> Process # 6 of 8 is alive. -> c2 
--> Process # 3 of 8 is alive. -> c1 
--> Process # 7 of 8 is alive. -> c2
```

**여기까지 완성하였다면 기본적인 구조는 모두 완성한 셈이다!!**





