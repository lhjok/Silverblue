## Fedora Silverblue
不可变操作系统、只读操作系统、桌面容器操作系统。

- 安装阶段，`/boot` 分区不能使用 `Btrfs` 文件系统。

```sh
 分区方案 (一、优点，可变分区在固态系统盘，程序运行会更快)：
 /boot= 10G(Ext4)、/boot/efi= 2G(EFI)、swap= 16G、
 /= 80G(Btrfs)、/home= 剩余固态磁盘(Btrfs)、用户挂载(~/data)=剩余机械磁盘(Btrfs)。
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

- 编辑 `~/.bash_profile` `~/.bashrc` 设置环境变量：

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

- 本地系统安装NVIDIA显卡驱动：

```sh
# 更新整个系统
$ rpm-ostree upgrade
$ systemctl reboot
# 添加第三方源
$ rpm-ostree install https://download1.rpmfusion.org/free/fedora/\
rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://download1.\
rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
$ systemctl reboot
# 安装NVIDIA显卡驱动
$ rpm-ostree install akmod-nvidia xorg-x11-drv-nvidia
# 如果需要CUDA支持，请运行以下命令。
# $ rpm-ostree install xorg-x11-drv-nvidia-cuda
$ rpm-ostree kargs --append=rd.driver.blacklist=nouveau \
--append=modprobe.blacklist=nouveau --append=nvidia-drm.modeset=1
$ systemctl reboot
```

- 本地系统安装基础开发环境：

```sh
$ rpm-ostree install vim screen cmake3 python2 python2-devel python3-devel gcc-c++ clang \
clang-devel libudev-devel autoconf automake glib-devel gtk3-devel libtool libgccjit \
the_silver_searcher ripgrep fd-find libvterm libvterm-devel openssl openssl-devel aria2 \
perl-core libsoup-devel webkitgtk4-jsc-devel webkit2gtk3-devel expat-devel libtree-sitter
$ rpm-ostree install --apply-live fcitx5-qt-module fcitx5-{gtk2,gtk3,gtk4}
```

- 添加第三方Flatpak源：

```sh
# 执行下面命令后，在软件仓库设置开启第三方Flathub源。
$ flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
$ flatpak remote-add flathub-beta https://flathub.org/beta-repo/flathub-beta.flatpakrepo
```

- 安装各种应用程序：

```sh
$ flatpak install flathub com.google.Chrome
$ flatpak install flathub com.microsoft.Edge
$ flatpak install flathub com.brave.Browser
$ flatpak install flathub com.visualstudio.code
$ flatpak install flathub com.valvesoftware.Steam
$ flatpak install flathub com.qq.QQmusic
$ flatpak install flathub org.telegram.desktop
$ flatpak install flathub tv.kodi.Kodi
$ flatpak install flathub org.gimp.GIMP
$ flatpak install flathub net.xmind.XMind
$ flatpak install flathub com.google.AndroidStudio
$ flatpak install flathub com.wps.Office
$ flatpak install flathub com.discordapp.Discord
$ flatpak install flathub com.github.alecaddd.sequeler
$ flatpak install flathub org.remmina.Remmina
$ flatpak install flathub org.blender.Blender
$ flatpak install flathub io.mpv.Mpv
$ flatpak install flathub com.skype.Client
$ flatpak install flathub com.jetbrains.CLion
$ flatpak install flathub com.jetbrains.GoLand
$ flatpak install flathub com.jetbrains.WebStorm
```

- 安装Fcitx5输入法并开机启动：

```sh
$ flatpak install flathub org.fcitx.Fcitx5
$ flatpak install flathub org.fcitx.Fcitx5.Addon.Rime
$ flatpak install flathub com.dropbox.Client
$ sudo cp /var/lib/flatpak/app/org.fcitx.Fcitx5/current/active/export/share\
/applications/org.fcitx.Fcitx5.desktop /etc/xdg/autostart/
$ sudo cp /var/lib/flatpak/app/com.dropbox.Client/current/active/export/share\
/applications/com.dropbox.Client.desktop /etc/xdg/autostart/
```

- 禁用ibus（因为Fedora中gnome-shell强依赖ibus所以无法直接删除ibus）

```sh
systemctl mask --user org.freedesktop.IBus.session.GNOME.service
systemctl mask --user org.freedesktop.IBus.session.generic.service
mkdir -p .local/share/dbus-1/services
ln -s /dev/null .local/share/dbus-1/services/org.freedesktop.IBus.service
ln -s /dev/null .local/share/dbus-1/services/org.freedesktop.portal.IBus.service
```

- 编辑 `sudo vi /etc/profile` 设置输入法（默认可不用设置）：

```sh
aria2c --enable-rpc --rpc-listen-port=6800 &
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```

- 解决中英文切换冲突，修改Rime的Shift键绑定：

```sh
$ sudo vi ~/.local/share/fcitx5/rime/build/default.yaml
```

```yaml
ascii_composer:
  ... ... ...
  Shift_L: noop
  Shift_R: noop
```

- 本地系统安装 `Virt-Manager` 虚拟机：

```sh
$ rpm-ostree install virt-install libvirt-daemon-config-network \
libvirt-daemon-kvm qemu-kvm virt-manager virt-viewer virglrenderer \
qemu-device-display-virtio-vga qemu-device-display-virtio-vga-gl
```

- 升级和回滚系统：

```sh
$ ostree remote list    //查看远程系统列表
$ ostree remote refs fedora    //选择要升级的系统
$ rpm-ostree rebase fedora:fedora/38/x86_64/silverblue    //执行升级到指定版本
$ rpm-ostree override remove firefox    //删除系统自带的Firefox浏览器
$ rpm-ostree cleanup -b    //清除临时文件
$ rpm-ostree cleanup -p    //删除挂起的部署
$ rpm-ostree cleanup -r    //删除回滚的部署
$ rpm-ostree cleanup -m    //删除缓存元数据
$ rpm-ostree update    //执行系统更新
$ rpm-ostree status    //查看回滚版本和已安装包详情
$ flatpak update    //升级所有Flatpak应用和依赖
$ rpm-ostree rollback    //回滚到上一次到版本
$ systemctl reboot    //重启系统
```

- 本地编译安装NeoVim编辑器：

```sh
$ pip3 install pynvim
$ git clone https://github.com/neovim/neovim.git
$ cd neovim
$ make CMAKE_INSTALL_PREFIX=/var/home/lhjok/.opt/neovim/
$ make install
```

- 容器工具的使用：

```sh
$ toolbox create     //创建一个默认容器
$ toolbox enter     //进入容器环境【可使用(dnf)安装软件】
$ toolbox list     //查看容器列表
$ toolbox rm 容器名     //删除容器（-f = 删除运行中的容器、-a = 删除所有容器）
$ toolbox rmi 镜像名     //删除镜像（-f = 删除运行中的镜像、-a = 删除所有镜像）
$ toolbox run 工具名     //在本地终端运行容器内的应用。
```

#### 配置开发环境

- 安装MySQL和PostgreSQL容器镜像：

```sh
$ podman pull mysql/mysql-server
$ podman pull postgres
```

- 查看已安装的镜像：

```sh
$ podman images
```

- 生成一个MySQL和PostgreSQL实例：

```sh
$ podman run -itd --name=qn_mysql -e MYSQL_ROOT_PASSWORD=password -p \
3306:3306 docker.io/mysql/mysql-server:latest
$ podman run --name qn_postgres -e TZ=PRC -e POSTGRES_USER=root -e \
POSTGRES_DB=qndbs -e POSTGRES_PASSWORD=password -p 5432:5432 -d postgres
```

- 查看实例的安装日志：

```sh
$ podman logs qn_mysql
$ podman logs qn_postgres
```

- 查看所有已安装的实例：

```sh
$ podman ps -a
```

- 进入实例并执行登录命令：

```sh
$ podman exec -it qn_mysql mysql -u root -p
mysql> CREATE DATABASE qndbs;
$ podman exec -it qn_postgres psql -U root qndbs
```

- 安装Redis容器镜像：

```sh
$ podman pull redis
```

- 生成一个Redis实例：

```sh
$ podman run -d --name=qn_redis -p 6379:6379 docker.io/library/redis:latest
```

- 启动和关闭实例：

```sh
# 开启和关闭MySQL实例
$ podman start qn_mysql
$ podman stop qn_mysql
# 开启和关闭PostgreSQL实例
$ podman start qn_postgres
$ podman stop qn_postgres
# 开启和关闭Redis实例
$ podman start qn_redis
$ podman stop qn_redis
```

- 本地连接MySQL容器出现拒绝连接的问题：

```sh
# 运行MySQL实例，并登录MySQL环境。
$ podman exec -it qn_mysql mysql -u root -p
# 查询允许连接的主机及用户信息。
mysql> select Host,User from mysql.user;
```

```sh
# 默认MySQL只支持本机连接数据库。
# 需要把本地Host主机名，改成"%"表示匹配所有Host主机。
mysql> update mysql.user set `Host` = '%' where `Host` = 'localhost' and User = 'root';
# 使上面的改动生效。
mysql> flush privileges;
# 某些客户端连接可能会出问题。
mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```

#### 开机启动MySQL、PostgreSQL和Redis容器

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

- 在 `~/.config/systemd/user` 目录下创建 `qn_postgres.service` 文件：

```sh
[Unit]
Description=Podman Container qn_postgres.service

[Service]
Type=forking
Restart=on-failure
ExecStart=/usr/bin/podman start qn_postgres
ExecStop=/usr/bin/podman stop -t 10 qn_postgres
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
# 开机启动MySQL、PostgreSQL和Redis容器
$ systemctl --user start qn_mysql
$ systemctl --user start qn_postgres
$ systemctl --user start qn_redis
```

- 使用 `Systemd` 开机启动命令（貌似不起作用）：

```sh
$ systemctl --user enable qn_mysql
$ systemctl --user enable qn_postgres
$ systemctl --user enable qn_redis
```

#### 设置VSCode在Flatpak环境下的终端问题

- 把下面代码拷贝到 `settings.json` 设置文件里。

```sh
{
    "editor.accessibilityPageSize": 15,
    "editor.fontSize": 15,
    "scm.inputFontSize": 14,
    "debug.console.fontSize": 15,
    "markdown.preview.fontSize": 15,
    "window.zoomLevel": 0.51,
    "workbench.tree.indent": 20,
    "window.titleBarStyle": "custom",
    "editor.fontFamily": "Consolas, 'Droid Sans Mono', 'monospace', monospace",
    "terminal.integrated.profiles.linux": {
        "ToolBox": {
            "path": "bash",
            "args": ["-c", "flatpak-spawn --host toolbox enter -c fedora-toolbox-38"]
        }
    },
    "terminal.integrated.defaultProfile.linux": "ToolBox",
    "update.mode": "none",
    "git.enableSmartCommit": true,
    "git.autofetch": true,
    "python.terminal.activateEnvInCurrentTerminal": true,
    "python.analysis.extraPaths": [
        "/var/home/lhjok/.local/lib/python3.11/site-packages",
        "/var/home/lhjok/.local/lib/python3.10/site-packages"
    ],
    "[python]": {
        "editor.formatOnType": true
    },
    "rust-analyzer.inlayHints.typeHints.enable": false,
    "rust-analyzer.inlayHints.parameterHints.enable": false,
    "rust-analyzer.inlayHints.chainingHints.enable": false,
    "rust-analyzer.inlayHints.closingBraceHints.enable": false,
    "editor.semanticTokenColorCustomizations": {
        "enabled": true,
        "rules": {
            "*.mutable": {
                "underline": false,
            }
        }
    },
    "editor.inlineSuggest.enabled": true,
    "editor.unicodeHighlight.includeStrings": false,
    "move-analyzer.server.path": "/var/home/lhjok/.cargo/bin/move-analyzer",
    "emmet.includeLanguages": {
        "rust": "html",
    },
    "go.gopath": "/var/home/lhjok/.golang",
    "go.goroot": "/var/home/lhjok/.opt/go",
}
```

#### 设置JetBrains全家桶Fcitx不跟随光标的问题

- 在Flatpak环境下编辑启动文件，修改成自己编译的JDK文件目录。

```sh
# 修改CLION的启动文件
$ cd /var/lib/flatpak/app/com.jetbrains.CLion
$ sudo vi current/active/files/extra/clion/bin/clion.sh
export CLION_JDK=/var/opt/images/jdk
# 修改GOLAND的启动文件
$ cd /var/lib/flatpak/app/com.jetbrains.GoLand
$ sudo vi current/active/files/bin/goland.sh
export GOLAND_JDK=/var/opt/images/jdk
# 修改WebStorm的启动文件
$ cd /var/lib/flatpak/app/com.jetbrains.WebStorm
$ sudo vi current/active/files/extra/webstorm/bin/webstorm.sh
export WEBIDE_JDK=/var/opt/images/jdk
# 修改IDEA的启动文件
$ cd /var/lib/flatpak/app/com.jetbrains.IntelliJ-IDEA-Ultimate
$ sudo vi current/active/files/extra/idea-IU/bin/idea.sh
export IDEA_JDK=/var/opt/images/jdk
# 修改PyCharm的启动文件
$ cd /var/lib/flatpak/app/com.jetbrains.PyCharm-Professional
$ sudo vi current/active/files/extra/bin/pycharm.sh
export PYCHARM_JDK=/var/opt/images/jdk
# 修改Android Studio的启动文件
$ cd /var/lib/flatpak/app/com.google.AndroidStudio
$ sudo vi current/active/files/extra/android-studio/bin/studio.sh
export STUDIO_JDK=/var/opt/images/jdk
```

#### 设置Virt-Manager桥接网络

- 在Virt-Manager虚拟机内使用和本地一样的网段。

```sh
$ nmcli connection add ifname vnet0 type bridge con-name vnet0 connection.zone trusted
$ nmcli connection add type bridge-slave ifname enp0s31f6 master vnet0
$ nmcli connection modify vnet0 bridge.stp yes
$ nmcli connection modify enp0s31f6 autoconnect no
$ nmcli connection down enp0s31f6
$ nmcli connection up id vnet0
```
