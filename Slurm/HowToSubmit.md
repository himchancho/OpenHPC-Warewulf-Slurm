# How to submit a Job to Slurm

다음과 같은 스크립트 파일을 작성한다.
```[링크] : https://slurm.schedmd.com/sbatch.html "Slurm B
[sms] vim job_script

#!/bin/bash
#SBATCH -N 10
#SBATCH -n 200
salloc mpirun --hostfile /hostfile -np 200 interFoam -parallel > job_log
```
옵션은 다양한 방법으로 만들 수 있다. ([링크] 참조)

```
[sms] sbatch ./job_script
```
스크립트파일을 sbatch 명령어를 이용하여 제출한다.


[hostfile 예시](./hostfile)  

[링크]: https://slurm.schedmd.com/sbatch.html "Slurm Batch"


