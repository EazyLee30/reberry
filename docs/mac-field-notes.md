# Mac 现场记录：SSH、BerryCore 与 Term49

记录日期：2026-10-11。宿主为 Apple Silicon（arm64）、macOS 27.0；这些信息在整理时读取。本次开始时，设备拥有者已重新刷机、完成 root 并打开 SSH。**Mac 上没有执行镜像构建、USB 发现或刷写**，不能据此认定 Apple Silicon 可以运行 Windows autoloader。

本文仅记录本次执行与观察。地址、私钥路径、应用目录后缀均使用占位符；不包含密码、私钥、设备照片或原始日志。

## 01 / 从 Mac 连上 QNX

初次连接报 `no matching host key type found`，设备提供 `ssh-rsa`。仅对此连接启用旧主机密钥算法后，密码登录成功；随后使用拥有者提供的本地私钥完成密钥登录。私钥权限原为 640，OpenSSH 拒绝使用，改为 600 后通过。

```sh
chmod 600 "$HOME/.ssh/bb10_lab"
ssh -i "$HOME/.ssh/bb10_lab" \
  -o HostKeyAlgorithms=+ssh-rsa \
  -o PubkeyAcceptedAlgorithms=+ssh-rsa \
  -o ConnectTimeout=10 root@'<BB10_IP>'
```

本次初次接受新主机密钥使用了 `StrictHostKeyChecking=accept-new`，没有独立核对指纹；后续复现应先核对设备指纹。没有为所有主机全局降低 SSH 配置，也没有上传私钥。

远程 `uname -a` 返回 QNX 8.0.0 / armle。原系统 `id` 不在预期路径，不能仅以此否定 root。空间检查显示用户文件系统约有 10 GB 可用。

## 02 / 下载、校验与传输 BerryCore

从 [BerryCore 官方 v0.90.1 发布](https://github.com/sw7ft/BerryCore/releases/tag/v0.90.1) 下载 `berrycore.zip` 和 `install.sh`。压缩包为 315,029,377 字节；Mac 上计算 SHA-256，与 GitHub 发布 API 的 digest 一致：

```text
berrycore.zip  170dd86b3ef12485540fa0d79ef19ef186442dc91201db82782fb8d7bbcfaa6a
install.sh    452d91947da6cd8e178772dc62f50d9f07670f5e930f73389473cb02a16d8eba
```

通过 SSH 上传到手机的 downloads 暂存目录。使用 `scp -O` 选择传统 SCP 协议；此处没有测试默认 SFTP 是否可用。Wi-Fi 传输约 1 MB/s，这只是本次观察，不是性能承诺。

```sh
scp -O -i "$HOME/.ssh/bb10_lab" \
  -o HostKeyAlgorithms=+ssh-rsa \
  -o PubkeyAcceptedAlgorithms=+ssh-rsa \
  berrycore.zip install.sh root@'<BB10_IP>':/accounts/1000/shared/downloads/berrycore-setup/
```

安装目标为 `/accounts/1000/shared/misc/berrycore`，创建了兼容旧工具的 `misc/clitools` 链接。安装器备份了 root 的 `.profile`，写入新配置、终端主题及 Documents 中的目录指南。没有更换 SSH 服务或重刷系统。

## 03 / 解压成功，不等于程序可以运行

`sh install.sh`、加载 `env.sh`、部分二进制均报 `Operation not permitted`。增加执行权限、复制脚本到 `/root` 都未解决。将已校验脚本内容送入系统 shell 可以推进主包解压，但加载环境时仍受阻。

Python 3.2 的 `pbpkgadd` 也无法读取运行；最终用系统 `unzip` 逐个展开内置 package，并完成安装器的后续配置。**这是一条现场恢复路径，不是官方安装器完整无误执行的证明。**

发布包内 `packages/gcc.zip` 实为 134 字节 Git LFS 占位文件，不是 ZIP，已跳过。没有安装或验证 GCC；其他包展开完毕并不意味着全部工具可运行。

发现 `/proc/boot/pathtrust` 后，对具体文件设置信任：

```sh
# root shell 中；前缀 ! 表示单个文件，不能省略成整文件系统授权。
/proc/boot/pathtrust '!/accounts/1000/shared/misc/berrycore/env.sh'
```

随后 qpkg help/catalog、Git 2.47.2、curl 8.12.1、nano 7.2、Vim 9.1、zsh 5.0.2、jq 1.6 实际运行通过；curl 对 GitHub HTTPS 请求返回 200。新版 `ls` 和 Bash 仍报错。保留新版 `ls` 为 `ls.berrycore-original`，将 `bin/ls` 链接到 `/bin/ls`，环境中的 ls alias 也切回系统命令。Bash 未修复。

## 04 / 终端自动加载与 SSH 短命令

用 `pidin` 读取 Term49 shell 环境，确认 HOME 是应用自己的 `data` 目录，**不是 `/root`**。在实际发现的目录写入 `.profile`：

```sh
export BERRYCORE_QUIET=1
export BERRYCORE_MOTD_ENABLED=0
export BERRYCORE_SETUP_AUTO=0
. /accounts/1000/shared/misc/berrycore/env.sh
. /accounts/1000/shared/misc/berrycore/ssh-shortcuts.sh
```

应用目录示例为 `/accounts/1000/appdata/<TERM49_PACKAGE>/data`。写入时使用设备上实际发现的所有者和组，并设置该配置文件的信任。系统 ksh 登录测试确认 NATIVE_TOOLS、qpkg 路径与 Git 可用；Term49 自带 mksh 的远程直接执行被拒绝，因此不能把这次 ksh 测试写成完整应用启动测试。

加入 `sshon` 与 `sshstatus` 两个 shell 函数，调用现有 `/var/rootdata/sshd-password` 和 `/var/etc/ssh/sshd_config`。启动前先用 `sshd-password -t` 检查配置，再检测现有 daemon，避免重复启动。

早期普通用户执行 `pidin ar` 会刷出 `/proc/.../as: Operation not permitted`。后续将检查与启动放进 `/bin/__root`，抑制进程扫描的 stderr，并改用系统 `/usr/bin/grep` 与 shell `print`。远程检查确认能输出 SSH running；**尚未验证重启后普通 Term49 用户首次执行整条启动路径**。进程存在也不等于认证成功：本次曾遇到 22 端口接受后立即关闭连接，后来实际 SSH 登录恢复。

## 05 / 欢迎消息、Python 和网页目录

远程 MOTD 请求返回 `404: Not Found`，已把 env.sh 的 MOTD 默认值设为关闭。`setup` 自动后台任务起初因缺少 python3 退出。

通过 `qpkg install python3` 安装 Python 3.11.10，并设置 wrapper、解释器及扩展库的文件信任。`python3 --version` 通过，但 `import math` 仍报 `ImportError: Not permitted`。增加库执行权限和调整 LD_LIBRARY_PATH 也未解决。

`setup start` 曾短暂显示成功，随后本地 HTTP 检查返回 000，`setup status` 为 stopped，日志指向 Python 扩展加载失败。`python3 -m ensurepip` 同样失败；pip 未初始化成功。已关闭 setup 默认自动启动，避免反复报错。**Python 本体存在，不等于 Python 应用环境完整可用。**

## 06 / 重启让信任状态丢失

拥有者重启后再次报告拒绝执行。Mac 重新连接并检查，确认 `env.sh` 已变成 untrusted：前面单次设置的文件信任没有跨重启保留。

随后新增 `/var/rootdata/berrycore-trust.sh`，对 root/Term49 profile、env.sh、SSH 函数、BerryCore bin/lib 和 Python 扩展逐文件恢复信任。在**本次实际发现的** Android `native/system/xbin/btool` root 启动脚本末尾加入调用：先信任 helper，再用系统 shell 执行，将日志写入 `/tmp/berrycore-trust.log`。修改前在 `/var/rootdata/backup/` 保存 btool 备份；检查两份脚本语法。

这是设备相关的 root 启动链修改，不能把应用后缀硬编码到别的设备，也不要直接重跑整个 btool（它还会处理 SSH、系统服务和安装任务）。本次没有把整个用户文件系统标为可信。

当前手动恢复后 env.sh 为 trusted，Git 与 SSH 状态函数通过。**开机 hook 尚未经过再次重启回归验证**；也未证明这些设置能承受 root payload 重装、应用目录变化或系统升级。下一步应重启后检查日志、Term49 自动加载、sshon 与新的实际 SSH 认证。

## 状态清单

| 项目 | 本次最后确认状态 |
| --- | --- |
| Mac → QNX 密码/密钥 SSH | 实际登录并执行命令通过 |
| BerryCore 基础工具 | qpkg、Git、curl、nano、Vim、zsh、jq 通过 |
| Term49 自动配置 | 已写入；ksh 登录验证通过；重启回归待测 |
| SSH 短命令 | 已写入；当前远程状态/重复启动检查通过 |
| 重启恢复文件信任 | 开机 hook 已写入、语法检查通过；重启回归待测 |
| GCC / Bash | GCC 占位包跳过；Bash 未解决 |
| Python / pip / setup | Python 版本通过；扩展加载、pip、网页功能未解决 |
| Mac 构建/刷写 | 本次未执行 |

[Windows 记录](windows-field-notes.md) · [Mac 后续验证](mac-handoff.md) · [致谢](credits.md)
