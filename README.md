## Fedora Silverblue
不可变操作系统（只读操作系统、桌面容器操作系统）。

- 安装阶段，`/boot` 分区不能使用 `Btrfs` 文件系统，用 `/var` 代替 `/home` 成为独立用户分区。
- 分区方案：`/boot=15G(Ext4)、/boot/efi=2G(EFI)、swap=16G、/=剩余第一磁盘(Btrfs)、/var=第二磁盘(Btrfs=200G)、/home=剩余第二磁盘(Btrfs)`。
- 核心三大工具：`toolbox` 容器工具 `rpm-ostree` 系统包管理器 `flatpack` 桌面应用容器。
  
```text
一、toolbox，容器工具，生成无GUI的终端容器，与本地系统共享/home目录。
二、rpm-ostree，是唯一可以修改系统的工具，主要用于安装软件包，升级系统和系统回滚（为保证系统清洁，不建议多用）。
三、flatpack，主要用于桌面GUI应用的安装。
```

- 开发环境的搭建方案：在用户环境下安装编译工具，可以和容器终端共享一致的开发环境。

```text
一、在用户根目录创建 (.opt) 目录，把从网上下载的软件包解压在此。
二、分别在 (.bash_profile) 文件和 (.profile) 文件内添加 (PATH) 环境变量。
三、可在用户环境下配置的开发工具（Rust、Go、NodeJS、Flutter、Dart、JDK）。
四、Rust开发环境安装：直接使用官网的一键安装脚本（容器内安装也一样，共享HOME目录）。
```

### 添加PATH环境变量

```sh
export ANDROID_HOME=$HOME/.Android/SDK
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
export GOPATH=$HOME/.golang
export JAVA_HOME=$HOME/.opt/jdk
export PATH=$JAVA_HOME/bin:$PATH
export PATH=$HOME/.opt/go/bin:$PATH
export PATH=$HOME/.opt/node/bin:$PATH
export PATH=$HOME/.opt/yarn/bin:$PATH
export PATH="$HOME/.cargo/bin:$PATH:$HOME/.npm-global/bin:$PATH"
export PATH="$GOPATH/bin:$PATH:$HOME/.opt/flutter/bin:$PATH:$HOME/.opt/dart-sdk/bin:$PATH"
export PATH="$ANDROID_HOME/tools:$PATH:$ANDROID_HOME/platform-tools:$PATH:$ANDROID_HOME/emulator:$PATH"
```

- 安装 `Chrome` 浏览器(本地RPM包)：
  
```sh
rpm-ostree install google-chrome-stable_current_x86_64.rpm
```

- 安装 `Virt-Manager` 虚拟机(本机安装)：

```sh
rpm-ostree install virt-install libvirt-daemon-config-network libvirt-daemon-kvm qemu-kvm virt-manager virt-viewer
```

- 重启电脑：

```sh
systemctl reboot
```

- 容器工具的使用：

```sh
toolbox create     //创建一个默认容器
toolbox enter     //进入容器环境【可使用(dnf)安装软件】
toolbox list     //查看容器列表
toolbox rm 容器名     //删除容器（-f = 删除运行中的容器、-a = 删除所有容器）
toolbox rmi 镜像名     //删除镜像（-f = 删除运行中的镜像、-a = 删除所有镜像）
toolbox run 工具名     //在本地终端运行容器内的应用。
```

- 安装桌面应用（GUI容器）:

在 `https://flathub.org/home` 下载应用市场镜像文件。
直接在GNOME软件中心安装即可。