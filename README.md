## Fedora Silverblue
不可变操作系统、只读操作系统、桌面容器操作系统。

- 安装阶段，`/boot` 分区不能使用 `Btrfs` 文件系统。

```sh
 分区方案 (一、优点，可变分区在固态系统盘，程序运行会更快)：
 /boot= 15G(Ext4)、/boot/efi= 2G(EFI)、swap= 16G、
 /= 剩余固态磁盘(Btrfs)、/home= 全部机械磁盘(Btrfs)。
```

```sh
 分区方案 (二、优点，可变分区挂在机械硬盘，可提高固态硬盘寿命)：
 /boot= 15G(Ext4)、/boot/efi= 2G(EFI)、swap= 16G、
 /= 剩余固态磁盘(Btrfs)、/var= 机械磁盘(Btrfs=200G)、/home= 剩余机械磁盘(Btrfs)。
```

- 核心三大工具：`toolbox` 容器工具 `rpm-ostree` 系统包管理器 `flatpack` 桌面应用容器。

```text
 一、toolbox，容器工具，生成无GUI的终端容器，与本地系统共享/home目录。
 二、rpm-ostree，是唯一可以修改系统的工具，主要用于安装软件包，升级系统和系统回滚。
 三、flatpack，主要用于桌面GUI应用的安装（使用GNOME软件中心安装）。
```

- 开发环境的搭建方案：在用户环境下安装编译工具，可以和容器终端共享一致的开发环境。

```text
 一、在用户根目录创建 (.opt) 目录，把从网上下载的软件包解压在此。
 二、分别在 (.bash_profile) 文件和 (.profile) 文件内添加 (PATH) 环境变量。
 三、可在用户环境下配置的开发工具（Rust、Go、NodeJS、Flutter、Dart、JDK）。
 四、Rust开发环境安装：直接使用官网的一键安装脚本（容器内安装也一样，共享HOME目录）。
```

#### 添加PATH环境变量

- 编辑 `~/.profile` `~/.bashrc` 设置环境变量：

```sh
export ANDROID_HOME=$HOME/.Android/SDK
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
export GOPATH=$HOME/.golang
export JAVA_HOME=$HOME/.opt/jdk
export PATH=$JAVA_HOME/bin:$PATH
export PATH=$HOME/.opt/go/bin:$PATH
export PATH=$HOME/.opt/node/bin:$PATH
export PATH=$HOME/.opt/neovim/bin:$PATH
export PATH="$HOME/.cargo/bin:$PATH:$HOME/.npm-global/bin:$PATH"
export PATH="$GOPATH/bin:$PATH:$HOME/.opt/flutter/bin:$PATH:$HOME/.opt/dart-sdk/bin:$PATH"
export PATH="$ANDROID_HOME/tools:$PATH:$ANDROID_HOME/platform-tools:$PATH"
export PATH="$ANDROID_HOME/emulator:$PATH"
```

- 本地系统安装输入法和基础开发环境：

```sh
rpm-ostree install fcitx5 fcitx5-autostart fcitx5-configtool fcitx5-gtk fcitx5-qt python2 \
fcitx5-qt-module fcitx5-rime fcitx5-chinese-addons cmake3 python2-devel python3-devel gcc-c++ \
clang clang-devel libudev-devel autoconf automake glib-devel gtk3-devel libtool libgccjit \
the_silver_searcher fd-find libvterm libvterm-devel
```

- 编辑 `sudo vi /etc/profile` `~/.profile` `~/.bash_profile` 设置输入法（默认可不用设置）：

```sh
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```

- 解决中英文切换冲突，修改Rime的Shift键绑定：

```sh
sudo vi ~/.local/share/fcitx5/rime/build/default.yaml
```

```yaml
ascii_composer:
  ... ... ...
  Shift_L: noop
  Shift_R: noop
```

- 本地系统安装 `Virt-Manager` 虚拟机：

```sh
rpm-ostree install virt-install libvirt-daemon-config-network \
libvirt-daemon-kvm qemu-kvm virt-manager virt-viewer
```

- 回滚系统和重启电脑：

```sh
rpm-ostree rollback
systemctl reboot
```

- 添加第三方Flatpak源：

```sh
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
flatpak remote-add flathub-beta https://flathub.org/beta-repo/flathub-beta.flatpakrepo
```

- 安装Chrome和Edge浏览器：

```sh
flatpak install com.google.Chrome
flatpak install com.microsoft.Edge
```

- 本地编译安装NeoVim编辑器：

```sh
pip3 install pynvim
git clone https://github.com/neovim/neovim.git
cd neovim
make CMAKE_INSTALL_PREFIX=/var/home/lhjok/.opt/neovim/
make install
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

#### 配置开发环境

- 安装MySQL容器镜像：

```sh
podman pull mysql/mysql-server
```

- 查看已安装的镜像：

```sh
podman images
```

- 生成一个MySQL实例：

```sh
podman run -itd --name=qn_mysql -e MYSQL_ROOT_PASSWORD=password -p \
3306:3306 docker.io/mysql/mysql-server:latest
```

- 查看实例的安装日志：

```sh
podman logs qn_mysql
```

- 查看所有已安装的实例：

```sh
podman ps -a
```

- 进入实例并执行登录命令：

```sh
podman exec -it qn_mysql mysql -u root -p
mysql> CREATE DATABASE qnDis;
```

- 安装Redis容器镜像：

```sh
podman pull redis
```

- 生成一个Redis实例：

```sh
podman run -d --name=qn_redis -p 6379:6379 docker.io/library/redis:latest
```

- 启动和关闭实例：

```sh
# 开启和关闭MySQL实例
podman start qn_mysql
podman stop qn_mysql
# 开启和关闭Redis实例
podman start qn_redis
podman stop qn_redis
```

- 本地连接MySQL容器出现拒绝连接的问题：

```sh
# 运行MySQL实例，并登录MySQL环境。
podman exec -it qn_mysql mysql -u root -p
# 查询允许连接的主机及用户信息。
select Host,User from mysql.user;
```

```sh
# 默认MySQL只支持本机连接数据库。
# 需要把本地Host主机名，改成"%"表示匹配所有Host主机。
update mysql.user set `Host` = '%' where `Host` = 'localhost' and User = 'root';
# 使上面的改动生效。
flush privileges;
# 某些客户端连接可能会出问题。
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```

#### 开机启动MySQL和Redis容器

- 在 `~/.config/systemd/user` 目录下创建 `qn_mysql.service` 文件：

```sh
[Unit]
Description=Podman Container qn_mysql.service

[Service]
Type=forking
Restart=on-failure
ExecStart=/usr/bin/podman start qn_mysql
ExecStop=/usr/bin/podman stop -t 10 qn_mysql
KillMode=none

[Install]
WantedBy=multi-user.target
```

- 在 `~/.config/systemd/user` 目录下创建 `qn_redis.service` 文件：

```sh
[Unit]
Description=Podman Container qn_redis.service

[Service]
Type=forking
Restart=on-failure
ExecStart=/usr/bin/podman start qn_redis
ExecStop=/usr/bin/podman stop -t 10 qn_redis
KillMode=none

[Install]
WantedBy=multi-user.target
```

- 在 `~/.bash_profile` 文件添加启动脚本：

```sh
# 开机启动MySQL和Redis容器
systemctl --user start qn_mysql
systemctl --user start qn_redis
```

- 使用 `Systemd` 开机启动命令（貌似不起作用）：

```sh
systemctl --user enable qn_mysql
systemctl --user enable qn_redis
```

#### 设置VSCode在Flatpak环境下的终端问题

- 把下面代码拷贝到 `settings.json` 设置文件里。

```sh
"terminal.integrated.profiles.linux": {
    "ToolBox": {
        "path": "bash",
        "args": ["-c", "flatpak-spawn --host toolbox enter -c fedora-toolbox-33"]
    }
},
"terminal.integrated.defaultProfile.linux": "ToolBox",
"terminal.integrated.automationShell.linux": "ToolBox",
```

#### 设置JetBrains全家桶Fcitx不跟随光标的问题

- 在Flatpak环境下编辑启动文件，修改成自己编译的JDK文件目录。

```sh
# 修改CLION的启动文件
cd /var/lib/flatpak/app/com.jetbrains.CLion
sudo vi current/active/files/extra/clion/bin/clion.sh
export CLION_JDK=/var/opt/images/jdk
# 修改GOLAND的启动文件
cd /var/lib/flatpak/app/com.jetbrains.GoLand
sudo vi current/active/files/bin/goland.sh
export GOLAND_JDK=/var/opt/images/jdk
# 修改WebStorm的启动文件
cd /var/lib/flatpak/app/com.jetbrains.WebStorm
sudo vi current/active/files/extra/webstorm/bin/webstorm.sh
export WEBIDE_JDK=/var/opt/images/jdk
# 修改IDEA的启动文件
cd /var/lib/flatpak/app/com.jetbrains.IntelliJ-IDEA-Ultimate
sudo vi current/active/files/extra/idea-IU/bin/idea.sh
export IDEA_JDK=/var/opt/images/jdk
# 修改Android Studio的启动文件
cd /var/lib/flatpak/app/com.google.AndroidStudio
sudo vi current/active/files/extra/android-studio/bin/studio.sh
export STUDIO_JDK=/var/opt/images/jdk
```
