# OpenHPC-Warewulf-Slurm
Using OpenHPC-Warewulf-Slurm, make High Performance Computer


## 0. H/W Requirements for this recipe
1. Head Node (DELL R720)  
  1-1. ssd (리뷰안 dx2200 * 2 , 리뷰안 960x)
2. Compute Node * 10 (Quanta Computer Windmill)  
  2-1. Infiniband card
3. IP router
4. Infiniband Switch

## 1. Installation Centos7.6
1. Install CentOS 7.6 on Head Node
centos7.6을 usb를 이용하여 부팅하면 다음과 같은 화면이 나타난다.
<img src = "./img/centos_install.png" width="800" height="600">
INSTALLATION SOURCE에서 설치할 디스크와 파티션을 잘 나누어주고
SOFTWARE SELECTION에서 GUI만 선택하고 (추가 애드온 없이) DONE한다

## 2. Installation OpenHPC
### 2-1 . Export Environment Variables
여기서 사용할 환경 변수들에 대한 설명이다
줄그어진 변수들은 설명서에는 있지만 여기서는 사용하지 않는 것들이다.
• ${sms name} # Hostname for SMS server  
• ${sms ip} # Internal IP address on SMS server   
• ${sms eth internal} # Internal Ethernet interface on SMS   
• ${eth provision} # Provisioning interface for computes   
• ${internal netmask} # Subnet netmask for internal network   
• ${ntp server} # Local ntp server for time synchronization   
~~• ${bmc username} # BMC username for use by IPMI ~~
~~• ${bmc password} # BMC password for use by IPMI ~~
• ${num computes} # Total # of desired compute nodes   
• ${c ip[0]}, ${c ip[1]}, ... # Desired compute node addresses   
~~• ${c bmc[0]}, ${c bmc[1]}, ... # BMC addresses for computes ~~
• ${c mac[0]}, ${c mac[1]}, ... # MAC addresses for computes   
• ${c name[0]}, ${c name[1]}, ... # Host names for computes   
• ${compute regex} # Regex matching all compute node names (e.g. “c*”)   
• ${compute prefix} # Preﬁx for compute node names (e.g. “c”) Optional:   
~~• ${sysmgmtd host} # BeeGFS System Management host name ~~
~~• ${mgs fs name} # Lustre MGS mount name ~~
• ${sms ipoib} # IPoIB address for SMS server  
• ${ipoib netmask} # Subnet netmask for internal IPoIB  
• ${c ipoib[0]}, ${c ipoib[1]}, ... # IPoIB addresses for computes  
• ${kargs} # Kernel boot arguments  
~~• ${nagios web password} # Nagios web access password~~

이런 변수들은 매번 치는게 어려우니 텍스트파일에 저장해놓고 필요할 때 마다 복사해서 export하도록 한다. 


