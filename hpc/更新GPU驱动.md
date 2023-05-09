查看 CUDA Toolkit 和 GPU Driver 版本对应表：

https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html



找到对应版本的 CUDA Toolkit：

https://developer.nvidia.cn/cuda-12-0-1-download-archive?target_os=Linux&target_arch=x86_64&Distribution=RHEL&target_version=8&target_type=runfile_local

这里有几种下载方式，我选择了`.run`的方法，里面有驱动的版本

```bash
wget https://developer.download.nvidia.com/compute/cuda/12.0.1/local_installers/cuda_12.0.1_525.85.12_linux.run
sudo sh cuda_12.0.1_525.85.12_linux.run
```

执行`.run`时可能出现说内核模块 nvidia 已经被加载，且被进程使用的情况：

> ERROR: An NVIDIA kernel module 'nvidia' appears to already be loaded in your kernel. This may be because it is in use (for example, by an X server, a CUDA program, or the NVIDIA Persistence Daemon), but this may also happen if your kernel was configured without support for module unloading. Please be sure to exit any programs that may be using the GPU(s) before attempting to upgrade your driver. If no GPU-based programs are running, you know that your kernel supports module unloading, and you still receive this message, then an error may have occured that has corrupted an NVIDIA kernel module's usage count, for which the simplest remedy is to reboot your computer.

```bash
lsmod | grep nvidia
lsof /dev/nvidia*
kill <proc_id>
```



更新CUDA 再 reboot 之后，可能 nvidia-persistenced service 没了，`nvidia-smi`

命令里面 Persistence-M （全名Persistence Mode）显示为off，正常情况下应该是on，表示驱动即使没有程序需要也常驻待机，可以保证高速响应。

正常情况下安装驱动时会生成`nvidia-persistenced`服务来维持Persistence Mode，如果发现该服务缺失，可以从`/usr/share/doc/NVIDIA_GLX-1.0/sample/nvidia-persistenced-init.tar.bz2`中找到安装脚本，手动生成服务。

```bash
tar -xjf <...>.tar.bz2
./install.sh
sudo systemctl enable nvidia-persistenced
sudo systemctl start nvidia-persistenced
```





可能是 `nvidia-persistenced`的 Github：

https://github.com/NVIDIA/nvidia-persistenced



更新驱动 reboot 之后，可能机器上的`slurmd`进程没了，这时需要重启

```bash
systemctl status slurmd
systemctl restart slurmd
```

重启可能出错，显示`slurmd.service: Failed with result 'exit-code'`，这时查看`slurmd`日志，显示是 MIG 没有添加设备，所以再添加设备：

```bash
sudo nvidia-smi mig -cgi 0 -C
```

 