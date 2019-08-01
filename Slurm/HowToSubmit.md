# How to submit a Job to Slurm

다음과 같은 스크립트 파일을 작성한다.
```[링크] : https://slurm.schedmd.com/sbatch.html "Slurm B
[sms] vim job_script

#!/bin/bash
#SBATCH -N 10
#SBATCH -n 200
salloc mpirun --hostfile ./hostfile -np 200 interFoam -parallel > job_log
```
옵션은 다양한 방법으로 만들 수 있다. ([링크] 참조)

sbatch option
#주석처리를 하여도 앞에 SBATCH가 있으면 slurm이 그 줄을 읽어서 설정에 적용함
#SBATCH -N : 사용할 ComputeNode의 개수 (max:10)
#SBATCH -n : 사용할 총 프로세서의 개수 (max:200)
#SBATCH -p : 파티션 이름 (여기서는 우선순위에 따른 mid와 high)

mpirun option
--hostfile : 호스트(ComputeNode)의 이름이 담겨있는 파일
-np : 총 프로세서의 개수


```
[sms] sbatch ./job_script
```
스크립트파일을 sbatch 명령어를 이용하여 제출한다.


[hostfile 예시](./hostfile)  

[링크]: https://slurm.schedmd.com/sbatch.html "Slurm Batch"


