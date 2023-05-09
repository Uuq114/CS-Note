[toc]

## 简介

slurm 是 linux 集群的资源管理和调度系统。

SLURM: simple linux utility for resource management

* 官网：https://slurm.schedmd.com/
* cheatsheet：https://slurm.schedmd.com/pdfs/summary.pdf



## 常用命令

* `sinfo`：显示队列、节点状态
* `scontrol`：显示、设定 slurm 作业、队列、节点等状态
* `sacct`：已完成作业的报告
* `squeue`：排队作业的状态
* `sbatch`：提交作业
* `scancel`：删除作业



### `sinfo`

**查看队列**

> 队列的状态有：NoResp DRAIN FAIL FUTURE RESUME POWER_DOWN POWER_UP UNDRAIN

查看队列信息：`sinfo --partition=cpu`

**查看节点**

> 节点的状态有：drain、alloc、idle、down、mix

查看节点信息：`sinfo`

查看可用节点：`sinfo -N --states=idle`

**格式**

可以修改`sinfo`输出的格式：

```bash
sinfo -R --Format='REASON:100,NODELIST:256' --noheader | sort
```



### `scontrol`

**查看、修改作业参数**

查看排队、运行中作业信息：`scontrol show job <job_id>`

用`job_id`暂停作业：`scontrol hold <job_id>`

根据`job_id`继续作业：`scontrol release <job_id>`

添加作业依赖性，即作业的顺序：`scontrol update dependancy=<job_id>`

修改某个作业优先级：`scontrol update job=<...> priority=<...>`

查看某个用户还在排队的作业的优先级：`squeue --partition=a100 --user=<...> --state=PD --noheader --Format=jobid`

**修改队列状态**

调整一些节点的状态（上线、下线）：`scontrol update node=xxx state=drain reason="xxx"`



### `sacct`

默认查看 24h 以内的作业信息：`sacct`

查看详细的信息：`sacct -l`

查看某个用户正在运行的作业：`sacct --user=<...> --state=R`

指定时间后的作业：`sacct -S YYYY-MM-DD`

查看更多信息：`$ sacct --format=jobid,jobname,account,partition,ntasks,alloccpus,elapsed,state,exitcode -j 3224`



### `squeue`

> 作业的状态包括：R、PD、CG（completing）、CD、F、TO、CA
>
> 运行中，排队中，即将完成，已完成、失败、超时、已取消

查看某个队列排队、在运行的作业：`squeue --partition=<...>`

查看某个job：`squeue -j <jobid>`

查看特定节点的作业信息：`squeue -n <HOST>`

查看细节信息：`squeue -l`

查看队列的各作业优先级：`squeue --partition=a100 --Format=jobid,account,username,prioritylong,reasonlist`



### `sbatch`

`sbatch`将作业脚本提交给系统，比如：`sbatch jobscript.slurm`

`jobscript.slurm`脚本里面可以设置一些参数，比如：作业队列、申请节点数目、总进程数、每个节点的进程数、作业完成邮件通知、作业输出 output/error 文件等

```slurm
#!/bin/bash

#SBATCH --job-name=hostname
#SBATCH --partition=cpu
#SBATCH -N 1
#SBATCH --mail-type=end
#SBATCH --mail-user=YOU@EMAIL.COM
#SBATCH --output=%j.out
#SBATCH --error=%j.err

/bin/hostname
```

