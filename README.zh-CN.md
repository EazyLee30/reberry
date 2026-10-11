<p align="center"><img src="assets/banner.svg" alt="BlackBerry Revival — The keyboard remembers" width="100%"></p>

<p align="center">
<img alt="Device" src="https://img.shields.io/badge/BlackBerry-Classic_%2F_Q20-252522?style=flat-square">
<img alt="System" src="https://img.shields.io/badge/BB10-10.3.3.3216-75404b?style=flat-square">
<img alt="Host" src="https://img.shields.io/badge/Field_notes-Windows_%2B_macOS-b39360?style=flat-square">
<img alt="Skill" src="https://img.shields.io/badge/Agent_skill-included-494e42?style=flat-square">
</p>

<p align="center"><a href="README.md">English</a> · <a href="README.zh-CN.md">简体中文</a> · <a href="README.zh-TW.md">繁體中文</a></p>

# reberry

**键盘还记得。我们让系统重新回应。**

一台 BlackBerry Classic / Q20 的恢复手记：从干净刷机，到 root、SSH，再到 Android 设置终于越过进度条。这里留下 Windows 与 Mac 上实际做过的操作、失败的线索与可交给下一位 agent 的经验。

[阅读杂志版博客 →](https://eazylee.xyz/reberry/zh-CN/) · [完整操作记录](docs/windows-field-notes.md) · [复用 skill](skills/blackberry-bb10-recovery/SKILL.md) · [Mac 实测：SSH、BerryCore、Term49](docs/mac-field-notes.md) · [Mac 接棒](docs/mac-handoff.md)

## v3 资源来源

本次使用者从 [BerryLife 的 Pre-rooted Autoloaders v3 页面](https://bberrylife.github.io/Pages/V3root.html) 获取 v3 工具与刷机资源。该页提供 v3 工具、原厂 ROM 的入口，并注明原作者 Oleksandr。请通过来源页面获取资源，本仓库不托管固件。

## 本次走通的路径

```text
匹配设备的 Stock 10.3.3.3216
  → 同源 AutoRoot v3 用户文件系统
  → 配套 radio
  → 首次启动与开发模式
  → Term49 / root / 自有 SSH 密钥
  → 独立 SSH 服务与可选密码认证
  → Android 配置权限修复
  → upd 身份提交 PPS 安装任务
  → 确认 success 后清理传入的 BAR
```

| 检查项 | 本次证据 |
| --- | --- |
| Stock、root v3、radio | 三次刷写均完成；root 更新日志完成 |
| SSH | 独立 22 端口可执行远程命令；密码认证也完成一次实际登录 |
| Android Runtime | 系统服务启动，使用者确认 Settings 能进入页面 |
| BAR 批量安装 | 八个任务均返回 `progress::100`、`result::success` |
| 游戏运行 | 尚未逐个打开验证，安装成功不等于运行兼容 |
| 网易云登录 | 旧版 APK 可安装；密码登录的服务器错误未解决 |

## 三个最值得带走的细节

1. 本次 v3 脚本检查 **`/var/rootdata/use.sshd`**。不要照抄 `use.ssh`，先读你下载版本的启动脚本。
2. root 不代表安装器的访问角色正确。**`/bin/on -u upd /bin/sh`** 能进入安装器需要的身份。
3. Android 日志中的 **`skipping insecure file`** 有明确方向：本次将四个配置文件从 777 改成 644、保留所有者后恢复启动。不要递归更改整个 `/apps`。

## 直接复制给 agent

```text
请读取 https://github.com/EazyLee30/reberry ，先阅读 README.md
和 skills/blackberry-bb10-recovery/SKILL.md，再根据任务阅读对应参考文件。
我拥有这台 BlackBerry 10 设备，请结合我的电脑系统、设备型号、固件版本和实际日志协助恢复。
先发现工具与当前状态；不要默认重刷。刷写前确认我已授权清空数据，以及 stock 与 root 镜像同源。
使用我自己生成的 SSH 密钥，凭据仅本地使用。不要发布密码、私钥、设备标识或未脱敏日志。
按证据区分端口监听、身份认证、root、Android 服务启动、安装成功和应用运行。
完成后记录实际结果与未解决问题；如果是 Mac，只记录真正执行过的 Mac 操作，不照搬 Windows 成功结论。
```

博客语言：[English](https://eazylee.xyz/reberry/) · [简体中文](https://eazylee.xyz/reberry/zh-CN/) · [繁體中文](https://eazylee.xyz/reberry/zh-TW/)。

把仓库中的 `skills/blackberry-bb10-recovery` 文件夹复制到你所用 agent 的技能目录即可；Codex 通常使用 `~/.codex/skills/`。也可以直接让 agent 阅读文件，无须安装。

## 阅读顺序

- [Mac：SSH、BerryCore、Term49 实测](docs/mac-field-notes.md)
- [分享预览与响应式布局](docs/site-sharing.md)

- [Windows 实战：刷写、SSH、Runtime、PPS](docs/windows-field-notes.md)
- [密码 SSH 的旧系统兼容分支](skills/blackberry-bb10-recovery/references/password-ssh.md)
- [Mac 后续贡献说明与复制 prompt](docs/mac-handoff.md)
- [致谢与资料来源](docs/credits.md)

## 隐私与边界

仓库仅包含重新撰写的文字、原创 SVG/PNG 和静态网页。固件、游戏、BAR/APK、SSH 私钥、设备截图、完整日志和账户文件均未上传。示例地址和路径是占位符。这里是单台设备的现场记录，不承诺所有固件或旧应用均可运行。复刷会清空数据，应由设备拥有者明确决定；本次使用者已选择不备份。

默认优先使用自有密钥认证。密码分支记录了历史系统的 DES 兼容限制，仅供理解问题；请读其限制，避免把旧 root SSH 暴露到互联网。

## 致敬

感谢 [BerryCore / sw7ft](https://github.com/sw7ft/BerryCore)、[BerryLife](https://bberrylife.github.io/)、[bb10root](https://github.com/bb10root)、[BerryFarm / Term49](https://github.com/BerryFarm/Term49) 及 BB10 社区的持续工作。感谢 OpenAI 的 ChatGPT 与 Codex 在资料整理、命令分析和执行协作中的帮助。工具、平台与社区作者保有各自的贡献；这份记录没有把他们的成果归为我们的发明。

原创文档与网页采用 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)。BlackBerry 名称属于相应权利人，本项目为社区记录，与官方无隶属关系。
