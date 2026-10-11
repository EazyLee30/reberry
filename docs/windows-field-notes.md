# Windows 现场记录：让 Classic 重新回应

记录日期：2026-10-11。设备：BlackBerry Classic / Q20。宿主：Windows + PowerShell。全部命令中的路径、地址与账户密钥均使用示例。读者需要自己发现设备地址、脚本版本和应用目录。

## 01 / 先把底座刷对

原先状态是 v3 root 后 SSH 不稳定、Android 应用停在加载页。设备拥有者授权清空、不备份。本次使用适配 Classic 的 Z30 STA100 系列 **10.3.3.3216** 原厂 autoloader，再使用同一份文件生成 v3 镜像。这个包名是本次所用来源，不应单凭文件名判断未知下载安全或硬件兼容。

Windows 准备驱动、可靠 USB 连接、对应 firmware、AutoRoot v3、BB10 MultiTool。检查下载来源和发布说明，并读取 `root_v3.ps1` 的参数：

```powershell
Get-Help .\root_v3.ps1 -Full
Get-Content .\patch.txt
```

在构建前，本次启用了以下 patch 行。版本变化时请先核对启动脚本确实读取这些标志：

```sh
touch /var/rootdata/use.root
touch /var/rootdata/use.sshd
push "patchdata/var/etc/ssh/authorized_keys2" "/var/etc/ssh/authorized_keys2"
chmod 644 "/var/etc/ssh/authorized_keys2"
chown 0:0 "/var/etc/ssh/authorized_keys2"
```

密钥用自己新建的，不沿用工具包中的共享私钥。以下在 Windows 生成一对带口令的 RSA 密钥，将 **`.pub`** 复制进 payload；私钥始终留在电脑：

```powershell
ssh-keygen -t rsa -b 2048 -f "$env:USERPROFILE\.ssh\bb10_lab"
Copy-Item -LiteralPath "$env:USERPROFILE\.ssh\bb10_lab.pub" `
  -Destination '.\patchdata\var\etc\ssh\authorized_keys2'
.\root_v3.ps1 -InputExe 'C:\BB10-Lab\stock-3216.exe' `
  -WorkDir 'C:\BB10-Lab\build'
```

先运行 **stock** autoloader，确认完成；再运行生成的 **root_v3**，完成后运行其 **radio**。分别等待工具结束，不并行、不在刷写中拔线。v3 的 root 包是用户文件系统，不是完整原厂固件；底层必须与构建来源匹配。本次三次均完成，随后 `/tmp/root_update.log` 显示 root 更新完成。

首次开机先到桌面，连电、连 Wi-Fi、打开开发模式；允许所需的第三方安装。给后台初始化留时间，结合日志判断完成，不能仅凭桌面出现或固定 20 分钟认定所有服务健康。

## 02 / 开发模式和 SSH 是两件事

开发接口可用不意味着 TCP 22 已监听。USB 网络地址在本次不可达，改用手机设置里实际显示的 Wi-Fi 地址。电脑先查端口：

```powershell
Test-NetConnection -ComputerName '<Q20_IP>' -Port 22
.\bb10mt.exe connect --help
```

`bb10mt connect` 在本次早期提供了随连接进程存活的 SSH 通路。它需要设备密码与公钥；按实际版本帮助传参，不在文档、历史记录或公开日志里保存密码。不要把这条临时通路当作开机自启服务。

若已有 Term49 / Term49CX，可在终端中进入 root，手动启动原生服务：

```sh
__root
touch /var/rootdata/use.sshd
/usr/sbin/sshd -e -f /var/etc/ssh/sshd_config -h /var/etc/ssh/ssh_host_rsa_key
```

`-e` 将错误写到终端；需要保持前台诊断时加 `-D`。不要重复启动多个争抢 22 端口的服务。若主机密钥不存在，先检查 `ssh-keygen` 是否可用，再用该设备上的工具生成，不从另一台设备复制主机私钥。

这次遇到的几种错误很直接：

| 报错 | 本次原因 |
| --- | --- |
| `Not a directory` | 将二进制写成 `/usr/sbin/sshd/`，多了末尾斜杠 |
| `No such file or directory` | 路径误写为 `/usr/sbin/sbin` |
| `Bad configuration option: AuthorizedKeyFile` | 正确名称是 **`AuthorizedKeysFile`** |
| `no hostkeys available` | 服务未加载有效主机密钥，需检查路径和权限 |

这台设备的 key 登录示例：

```powershell
ssh -o HostKeyAlgorithms=+ssh-rsa `
    -o PubkeyAcceptedAlgorithms=+ssh-rsa `
    -o MACs=+hmac-sha1 `
    -i "$env:USERPROFILE\.ssh\bb10_lab" root@'<Q20_IP>'
```

兼容选项仅给这一台旧设备使用，不要全局降低其他 SSH 连接配置。第一次连接核对主机指纹，不使用 `StrictHostKeyChecking=no`。

本机部分常见 Unix 命令不在预期路径；`/bin/id`、`/bin/printf` 不可用不能证明 root 失败。用真实命令执行和受限位置写入的权限证据判断。本次已完成 root SSH 远程操作。后来安装阶段设备已重新启动过仍能连接，但没有做一套受控的重启回归检查。

## 03 / 密码登录是独立的兼容问题

打开 `PasswordAuthentication` 不等于认证实现能够验证当前账户数据库。这台原生 OpenSSH 6.2 没有走通所需 root 密码认证。最终从 BerryCore 包中提取 **原始 OpenSSH 7.1p2** 程序，独立运行；没有把完整 BerryCore 环境先装到手机上。

核心配置使用 `AuthorizedKeysFile`、`PermitRootLogin=yes`、`PasswordAuthentication=yes`、`PermitEmptyPasswords=no`、独立 HostKey 与 PidFile。本次还使用 `UsePrivilegeSeparation=no`。路径信任、QNX 账户 overlay 与哈希格式必须分别处理。详细步骤在 [密码 SSH 参考](../skills/blackberry-bb10-recovery/references/password-ssh.md)。

实测替代服务支持传统 DES crypt，MD5 格式没有成功。最终密码登录通过。这不是建议在新系统使用弱哈希：DES 只考虑密码前八个字节，旧 root 服务只宜在可信局域网实验，密钥登录优先。

## 04 / Android 进度条背后的权限

Settings BAR 安装后一直留在加载页。通过 `pidin ar` 与 `slog2info` 检查，最初存在 Android init/resmgr 等进程，却没有正常的 zygote/system_server。关键日志摘要：

```text
init: skipping insecure file '.../native/default.cfg'
init: skipping insecure file '.../native/init.cfg'
init: skipping insecure file '/system/build.cfg'
shrimp: Startup timed out. Android file system not ready or Android failed to launch.
```

本次 v3 patch 中包含递归 777；对应配置文件的可写权限过宽。先发现应用目录，不硬编码设备专属后缀：

```sh
/bin/ls -d /apps/sys.android.*/native
```

确认唯一目标后，以实际完整路径代替下方 `<ANDROID_NATIVE>`，检查再仅修正四个文件：

```sh
/bin/ls -l <ANDROID_NATIVE>/default.cfg <ANDROID_NATIVE>/init.cfg <ANDROID_NATIVE>/system/build.cfg <ANDROID_NATIVE>/autolaunch.cfg
/bin/chmod 644 <ANDROID_NATIVE>/default.cfg <ANDROID_NATIVE>/init.cfg <ANDROID_NATIVE>/system/build.cfg <ANDROID_NATIVE>/autolaunch.cfg
```

保留原有所有者，不递归 chown/chmod 全系统。我们停止了日志定位的 Android init 与 shell 进程，让使用者重新打开 Settings；随后 zygote、installd、SystemServer 出现，系统扫描、证书收集和 dexopt 开始运行。首次再次打开仍可能早于应用内部安装完成，应等安装队列完成，再关闭卡片、重新打开。最终使用者确认已经进入设置页面。

本机 build.cfg 显示 **Android 4.3 / API 18 / armeabi-v7a**。root 不会把 Runtime 升级成现代 Android。APK 判断需读取真实包的 minSdk 和 native ABI，不能仅看版本号。网易云旧版可以安装但云端密码登录仍报服务器错误，不能把旧服务认证失败写成 Runtime 已坏。

## 05 / Windows 上传，QNX 安装器接棒

`scp` 从 Windows 上传 BAR；远端使用简短 ASCII 文件名，避开中文路径处理差异。例如远端暂存为 `/var/rootdata/staged-reader.bar`。安装由 PPS 请求触发：**root SSH 里的默认身份不等于 upd 安装器身份**。

以下是手机侧示例，确认文件已上传并对 upd 可读后执行。任务名每次唯一：

```sh
/bin/chmod 644 /var/rootdata/staged-reader.bar
/bin/on -u upd /bin/sh -c 'echo action::install > /pps/system/installer/upd/current/job.reader-example.bar; echo package_location::/var/rootdata/staged-reader.bar >> /pps/system/installer/upd/current/job.reader-example.bar; echo extras:: >> /pps/system/installer/upd/current/job.reader-example.bar; /bin/chmod 660 /pps/system/installer/upd/current/job.reader-example.bar'
/bin/on -u upd /bin/cat /pps/system/installer/upd/current/job.reader-example.bar
```

等待 `progress::100` **与** `result::success`；Android 包还可能经历 `apk::go` → `apk::done`。别把 PPS 文件已写入当作安装成功。失败时保存脱敏错误摘要，先处理错误，不循环盲目重装。大量包上传与解压争用存储，Asphalt 的大包在本次耗时数分钟。

### 八个实际完成的任务

| 文件标签 | 安装器识别名称 / 版本 | 结果 |
| --- | --- | --- |
| playepub_BB10 | Playepub Book Reader 3.2.1.1 | success |
| PlantsVsZombies | Plants Vs Zombies 1.3.0.1307 | success |
| Cut-the-Rope-2_1.6.3 | Cut the Rope 2 / 1.22.52124.0 | success |
| Asphalt_6_HD | Asphalt 6 / 1.0.2.2 | success |
| Angry-Birds_1.4.1 | Angry Birds **Star Wars** / 1.0.1410.0 | success |
| Angry-Birds_5.0.2 | Angry Birds / 1.0.5020.0 | success |
| 2048_2_0_16 | 2048 / 2.0.16.707 | success |
| Minion-Rush_3.3.0p | Minion Rush / 1.0.33026.0 | success |

此前 BAR Installer、Term49CX、Settings 也已安装。上述只代表安装器完成，未逐个验证游戏启动、画面比例、触控和网络服务。

## 06 / 可以删安装包，但要删对

成功后删除自己上传的暂存 BAR，不会卸载应用。此次清理了十三份已知成功的暂存文件，释放约 **780 MiB**。保留 AutoRoot 的 `Term49.bar`、`getroot.bar` 等维护 payload；不清扫 `/apps`、应用数据或未知系统暂存文件。

```sh
# 仅在对应任务 result::success 后，删除已知上传的这一份
/bin/rm /var/rootdata/staged-reader.bar
df -h /var/rootdata
```

## 留给下一位操作者

最有价值的是把证据链留下：刷写完成 → root 更新完成 → 端口监听 → 认证成功 → Android 系统服务正常 → 安装器成功 → 应用可用。它们是不同的状态。下一篇 Mac 记录会由实际执行者补上；目前没有将未知的 Mac 步骤冒充为成功经验。

[资料与致谢](credits.md) · [复用 skill](../skills/blackberry-bb10-recovery/SKILL.md)
