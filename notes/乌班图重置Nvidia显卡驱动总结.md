---
date: 2021-7-3
categories: Daily_Life
tags:
  - Ubuntu
  - Nvidia
  - 总结
comments: true
title: 乌班图重置Nvidia显卡驱动总结
---

这已经是数不清第几次的重置显卡驱动了，这次出问题起因就是先前为了安装一个封装版QQ，其使用了高版本的程序库（基于Ubuntu20开发的），所以我1804装不上，结果我手贱用 aptitude 强制安装了相关程序库，结果导致开机之后黑屏了，分析应该是显卡驱动完蛋了，所以重置了显卡驱动，最终回归正常。

<!--more-->

### 可以确定的经验

- 升级内核（或者安装某些软件时升级了某些库）后进入不了系统、黑屏不要着急，更不需要重装系统！！只是黑屏、卡机、或者卡开机画面（必须通过引导），都是显卡出了问题，只需要重装显卡驱动即可。

- 进不去系统如何修复呢？使用 Recovery 模式！在Grub引导界面，选择 Advance 进入后选择 内核的recovery选项，进入recovery模式内，可完成旧驱动卸载、新驱动安装等一系列操作。如果网络不通，选择菜单中的 Network 打开网络即可。这样我们就能在 Recovery 模式下直接从网络下载新的驱动安装包。

### 完全卸载旧驱动

```
sudo apt-get --purge remove nvidia*
sudo apt autoremove
sudo apt-get --purge remove "*cublas*" "cuda*"
sudo apt-get --purge remove "*nvidia*"  //有点重复，不影响
```

此时，系统内应该已经没有Nvidia显卡，`nvidia-smi` 显示为空。并且此时重启开机，电脑应该可以进入桌面，因为这时 Ubuntu 会使用默认的 Nouveau 显卡驱动来驱动更新。

值得注意的是：Nouveau并不是集显驱动，它的定位是 Ubuntu 默认的显卡驱动，不管有没有集显，当系统中不存在英伟达显卡时，Ubuntu 就会用 Nouveau 驱动显示。

### 常见的显卡安装方法及问题

- 本人机器上，进入桌面后，使用系统App（软件更新）进行Nvidia显卡安装，安装后无法进入系统。
- 本人机器上，直接使用Cuda默认绑定的显卡安装包，可完美安装显卡驱动。
- 本人机器上，低版本的Nvidia-410（+Cuda10.0）经测试无法正常安装，但450（+Cuda11）版本以上正常安装。

### 必须禁用Nouveau

进入Nvidia驱动安装界面时 Nouveau 必须在禁用状态，`lsmod | grep nouveau` 应该无输出（如果有，则表明Nouveau依然被载入且运行，此时无法通过Nvidia驱动安装程序的检查，从而导致安装进程终止）

如何禁用？

- 首先可以在 `/etc/modprobe.d/blacklist.conf` 中添加 `blacklist nouveau` 和 `options nouveau modeset=0` 两行，然后执行 `sudo update-initramfs -u`。（注意：此步骤可省略。当第一次运行Nvidia驱动安装程序时，如果Nouveau并未被禁用，则安装程序会在上述目录自动生成一个conf文件，内容和我们自己写的一样。并且会提示我们reboot，以让写入的conf文件生效）
- 其次，在开机后的grub页面，`quiet splash` 后面紧跟插入 `nomodset`，再重新进入启动流程。
- 再次，手动关闭桌面显示。`service lightdm stop` 或者 `service gdm stop`，停用桌面服务。（注意：此时要 Ctrl+Alt+F3 进入tty来执行后续操作及安装过程）
- 最后，验证的方法就是 `lsmod | grep nouveau` 没有任何输出！

### BIOS禁用 SecureBoot

NVIDIA 由于由于Ubuntu的内核编译默认设置了 CONFIG_MODULE_SIG 为真，在支持UEFI的设备上打开Secure Boot 后，Ubuntu对于添加到内核的模块更加保守, 需要持有签名才能添加到模块中。这里我们直接进入BIOS关闭 SecureBoot 即可。

### Cuda绑定安装显卡驱动

对于执行 Cuda 绑定的驱动程序来说，无需附加诸如 `-no-x-check` 之类的参数，这些参数只有在直接通过 Nvidia 驱动安装程序安装时才可以追加。我们这里是 Cuda 的安装中附带安装 Nvidia 驱动！

**注意：**

- **Nvidia-470 版本及以上，不包括i386的程序库支持**，也就是说某些使用32位程序库的第三方软件（比如CNCNet），无法在Nvidia-470驱动上工作，会报错：`libGL error: No matching fbConfigs or visuals found`。这时只能从新安装低版本的驱动，比如我现在使用Nvidia-450，CNCNet完美运行。

- 安装过程中出现错误不要紧，留意错误下方的日志路径，在Cuda安装过程中，问题几乎就出在显卡驱动的安装上，而Nvidia驱动安装出现的错误会在 `/var/log/nvidia-installer.log` 中，查看会发现错误非常单一，基本上就是 `Nouveau并未被禁止`，或者是 `nvidia-drm正在运行`。前者的解决方案在上面，对于后者，通常是 Nvidia 驱动没卸载干净所致，所以要充分的卸载。重复以下过程：

```
sudo apt-get --purge remove nvidia*
sudo apt autoremove
sudo apt-get --purge remove "*cublas*" "cuda*"
sudo apt-get --purge remove "*nvidia*"  //有点重复，不影响
```

如果仍然存在 `nvidia-drm` 错误，那么执行以下命令（此时应该在停用 lightdm 或 gdm 之后的 tty 中运行这些命令）

```
systemctl isolate multi-user.target
modprobe -r nvidia-drm
```

第一条命令执行完毕后可能会退出当前的tty进入黑屏，此时只需重新 Ctrl+Alt+F3 进入tty即可。

### 正常流程

如果一切正常，在 Recovery 模式中或者是进入桌面后的 tty 中，直接下载并执行安装脚本即可。例如：

```
wget http://developer.download.nvidia.com/compute/cuda/11.0.2/local_installers/cuda_11.0.2_450.51.05_linux.run
sudo sh cuda_11.0.2_450.51.05_linux.run
// 过程中要同意安装绑定的Nvidia显卡驱动
```


### 参考资料

- How to disable Nouveau kernel driver https://askubuntu.com/questions/841876/how-to-disable-nouveau-kernel-driver
- System Isolation Failed https://bbs.archlinux.org/viewtopic.php?id=228445
- How to unload kernel module 'nvidia-drm'? https://unix.stackexchange.com/questions/440840/how-to-unload-kernel-module-nvidia-drm
- Ubuntu uninstall the Nvidia driver and install the latest driver https://www.programmersought.com/article/38077690789/
- libGL error: No matching fbConfigs or visuals found libGL error: failed to load driver: swrast https://askubuntu.com/questions/834254/steam-libgl-error-no-matching-fbconfigs-or-visuals-found-libgl-error-failed-t/903488#903488
- Nvidia drivers without 32 bit library https://github.com/ValveSoftware/steam-for-linux/issues/7901
