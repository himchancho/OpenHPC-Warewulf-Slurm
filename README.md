# OpenHPC-Warewulf-Slurm
Using OpenHPC-Warewulf-Slurm, make High Performance Computer


## H/W Requirements for this recipe
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
