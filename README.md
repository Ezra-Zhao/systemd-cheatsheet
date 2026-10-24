# systemd 命令速查表 · systemd Cheatsheet

[![Stars](https://img.shields.io/github/stars/Ezra-Zhao/systemd-cheatsheet?style=social)](https://github.com/Ezra-Zhao/systemd-cheatsheet/stargazers) [![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE) ![Languages](https://img.shields.io/badge/languages-3-blue)

> 🌍 [Español](README.es.md) · [Português](README.pt.md)

一份**一页纸**的 systemd 速查表：30 条真实可用的用法——`systemctl` 服务管理、`journalctl` 日志、unit 文件、定时器、系统状态。每个条目只讲三件事——**它是干什么的、一条能直接复制粘贴的命令、适用场景标签**。不编造参数：所有命令与选项均对照本机 systemd 255 的 `--help` 输出逐条核对，两个示例 unit 用 `systemd-analyze verify` 实际验证过、日历表达式用 `systemd-analyze calendar` 实际跑过。多数管理命令需要 `sudo`；还有一个前提——容器等非 systemd 引导的环境里 `systemctl` 直接不可用，先确认你的环境再往下看。

## 面向谁

运维工程师、后端开发者、SRE，以及任何"服务起不来只会重启服务器"的人。如果 `enable` 和 `start` 的区别、日志怎么按时间段捞、定时器怎么代替 cron 这几件事总让你现查，这张表能让你一次记牢。

## 怎么用

1. 直接看 **[CHEATSHEET.md](CHEATSHEET.md)** ——速查表正文，英文命令全球开发者都无压力。
2. 本地要有一台以 systemd 引导的 Linux（绝大多数主流发行版都是）；容器和 WSL 默认环境里多数命令跑不了，拿虚拟机或云主机练最稳妥。
3. 按"服务管理 → 日志 → unit 文件 → 定时器 → 系统状态"的顺序，每个条目亲手敲一遍，一小时就顺手了。

## 先跑起来（30 秒验证环境）

```bash
systemctl --version                # 打印版本（如 systemd 255），说明工具在
systemctl is-system-running          # 看整体状态，可能的输出：
#   running   = 全部起来了，正常
#   degraded  = 起来了，但有单元失败（用 systemctl --failed 看是谁）
#   starting  = 还在启动中；maintenance = 维护/救援模式；offline = 管理器没在跑
systemd-analyze                      # 总启动耗时——这条在本机就能跑，不用 sudo
ps -p 1 -o comm=                     # 打印 systemd 才说明是 systemd 引导；容器里通常是别的名字
```

## 一个能直接用的 service unit 示例

```ini
[Unit]
Description=MyApp demo service
After=network.target
Wants=network.target

[Service]
Type=simple
ExecStart=/usr/bin/sleep infinity
Restart=on-failure
User=www-data

[Install]
WantedBy=multi-user.target
```

把 `ExecStart` 换成你自己的程序、存为 `/etc/systemd/system/myapp.service`，再 `sudo systemctl daemon-reload && sudo systemctl enable --now myapp` 就跑起来了。这个示例与正文里的 unit 剖析是同一个文件，已用 `systemd-analyze verify` 验证通过。

## 收录了什么

| 分类 | 内容 |
|---|---|
| 🎛️ 服务管理 systemctl | status 看状态 / start·stop / restart 与 reload 的区别 / enable 只管开机自启 / --now 一步到位 / is-* 状态查询 / mask 彻底禁用 / daemon-reload（8 条） |
| 🔍 日志 journalctl | -u 按服务看 / -f 实时跟踪 / -b 按开机次捞 / --since 时间窗 / -p 优先级 / 磁盘占用与 vacuum 清理（6 条） |
| 📄 unit 文件 | cat 看合并结果 / 存放位置与优先级 / .service 逐行剖析 / edit 写 drop-in / show 查属性 / list-dependencies 依赖树 / --failed 与 reset-failed（7 条） |
| ⏰ 定时器 Timer | list-timers 总览 / .timer 文件剖析 / calendar 验表达式 / 启用 timer 的坑 / 相对型触发（5 条） |
| 🧰 系统状态 | analyze·blame 启动耗时 / set-default 切目标 / reboot·poweroff 与 isolate / hostnamectl 三件套（4 条） |

官方文档只链 https://www.freedesktop.org/software/systemd/man/latest/ ，所有命令均对照 systemd 255 的 help 输出核实过语法，示例 unit 与日历表达式都实际运行验证过。

## 为什么是这 30 条

- **服务管理占日常九成**：status / start / enable / journal 是每天都摸的，先把这组练到不用想。
- **几个坑先说清**：`enable` 只改开机链接、不会启动当前服务；`mask` 之后连手动 `start` 都不行；`--vacuum-*` 删掉的日志找不回来；服务器上误设 `set-default` 或随手 `poweroff`，代价都是要跑机房或开控制台救。
- **unit 要会看也会改**：`cat` 看合并后的真相、`edit` 写 drop-in 覆盖——不直接改 `/usr/lib` 里的厂商文件，升级不被冲掉。
- **定时器替代 cron**：`OnCalendar` + `Persistent=true`，关机错过的任务开机补跑；cron 做不到这一点。

## 欢迎 PR

发现命令写错了、参数过时了、翻译不到位——欢迎提 Pull Request。新增条目请保持统一格式（一句话说明 / 代码块 / 标签），**只收真实可用的命令，不确定的宁可不收**。

## 📚 更多速查表

同一作者的其他速查表，同样有中/西/葡三语版本：

- [FFmpeg 命令速查表](https://github.com/Ezra-Zhao/ffmpeg-cheatsheet) — 28 条音视频处理命令
- [Git 实战速查表](https://github.com/Ezra-Zhao/git-cheatsheet) — 30 条日常+救命命令
- [Docker 命令速查表](https://github.com/Ezra-Zhao/docker-cheatsheet) — 28 条容器命令
- [curl 命令速查表](https://github.com/Ezra-Zhao/curl-cheatsheet) — 29 条 API 调试命令
- [正则表达式速查表](https://github.com/Ezra-Zhao/regex-cheatsheet) — 30 条实战正则模式
- [jq 命令速查表](https://github.com/Ezra-Zhao/jq-cheatsheet) — 30 条 JSON 处理 filter
- [终端神器速查表](https://github.com/Ezra-Zhao/terminal-superpowers-cheatsheet) — 14 个终端效率工具
- [Adobe 免费替代速查表](https://github.com/Ezra-Zhao/adobe-free-alternatives-cheatsheet) — 16 个 Adobe 免费替代品
- [退订省钱速查表](https://github.com/Ezra-Zhao/desubscribe-cheatsheet) — 10 组退订替代，年省约 $1,260
- [kubectl 命令速查表](https://github.com/Ezra-Zhao/kubectl-cheatsheet) — Kubernetes 常用命令速查
- [SQL 查询速查表](https://github.com/Ezra-Zhao/sql-cheatsheet) — 常用 SQL 查询与函数速查
- [SSH 命令速查表](https://github.com/Ezra-Zhao/ssh-cheatsheet) — 28 条 SSH 命令

## License

MIT © 2026 Guangyi Zhao，详见 [LICENSE](LICENSE)。
