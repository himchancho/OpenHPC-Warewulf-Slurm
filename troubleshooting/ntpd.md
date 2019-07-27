ntpd troubleshooting
====================


ComputeNode들을 정비하다 보면 Headnode의 Slurmctl에서 잡지 못하는 경우가 있다.
```
[sms] sinfo -N
NODELIST   NODES PARTITION STATE
bridge1        1   normal* alloc
bridge2        1   normal* alloc
bridge3        1   normal* down*
bridge4        1   normal* alloc
bridge5        1   normal* alloc
bridge6        1   normal* alloc
bridge7        1   normal* alloc
bridge8        1   normal* alloc
bridge9        1   normal* alloc
bridge10       1   normal* alloc
```
STATE에 *가 있는것은 연결이 안된상태란 뜻이다.
그러므로 
```
[sms] scontrol update nodename=bridge3 state=RESUME
```
이 명령어를 쳐도 돌아오지 않는다.

우선 bridge3에 ssh접속해서 연결상태가 되어 있는지 확인한다.
만약 ssh접속이 된다면
```
[bridge3] scontrol ping
```
을 해주고 Down이 나온것을 확인

```
[sms] vim /var/log/slurmctld.log
```

[2019-07-27T18:43:22.627] ENCODED: Sat Jul 27 18:36:04 2019   
[2019-07-27T18:43:22.627] DECODED: Sat Jul 27 18:43:22 2019   

이러한 로그를 확인할 수 있다.

ntp의 gap이 크지않아 업데이트가 되지 않는 상황이다.
```
[sms] service ntpd stop
[sms] ntpd -g # Allow the first adgustment to be big
[sms] service ntpd start
[sms] service slurmctld restart
[bridge3] service slurmd restart
```
sinfo를 이용해 잘 작동되는지 확인해보면 된다. 


출처 : https://github.com/ciemat-tic/codec/wiki/Slurm-troubleshooting
