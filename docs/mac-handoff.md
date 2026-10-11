# Mac 接棒：后续验证与构建/刷写

已补充 [Mac 实测记录](mac-field-notes.md)：SSH、BerryCore 安装、Term49 自动加载、SSH 短命令及重启信任恢复的现场处理。开机 hook 尚待重启回归，Python 扩展、pip、setup 和 Bash 问题未解决。Mac 构建与刷写未执行。

此前发布了 Windows 现场记录。Mac，尤其 Apple Silicon 上的操作，需要后续执行者以自己的日志补充。不要把构建支持误写成 Windows autoloader 可原生运行，也不要把未执行的刷写写成成功。

```text
请继续 https://github.com/EazyLee30/reberry 的 Mac 章节。
先读 README.md、docs/windows-field-notes.md 与复用 skill。
确认我的 macOS 版本、芯片架构、连接方式以及现有设备状态。
读取下载版本的 root_v3.sh 与 bb10mt 帮助，记录实际执行的镜像构建、设备连接和 SSH 操作。
未经授权不要重刷当前已恢复的设备。不要把 Windows autoloader 的成功推断为 Apple Silicon 可直接刷写。
补充 docs/mac-field-notes.md，写下环境、命令（凭据占位）、日志摘要、成功证据、失败和限制。
同步更新博客的 Mac 接棒段落及 README 链接。沿用奶油白、墨黑、莓红的杂志风格。
发布前移除私钥、密码、个人 IP、设备 PIN/序列号、个人路径和原始日志。
```

建议交付：构建耗时与工具版本、可用架构、USB/Wi-Fi 发现方式、SSH 兼容选项，以及未能在 Mac 完成的环节。只有真实执行并有证据的部分才标记为完成。
