# VPS → ISP：让你和 Agent 都能照着走的上网链路指南

**你只需要会用 Codex。先弄清买什么，再让 Agent 一步一步带你配置。**

> 公开流程版 v0.1 · 中文优先 · 2026-10-25 核对供应商公开资料。提供采购、配置参考与验收流程；不是已经验证所有设备的一键安装器。

```mermaid
flowchart LR
    A[你的电脑或手机] -->|加密入口| B[VPS 中转服务器]
    B -->|向 ISP 代理发起连接| C[固定 ISP 出口]
    C --> D[允许访问的互联网服务]
```

**VPS 负责中转，ISP 负责最终出口。只买 VPS，网站看到的通常还是机房 IP。**

## 你现在该点哪里

| 你目前的情况 | 下一步 |
|---|---|
| 完全不懂这些词 | [三分钟看懂](docs/01-basics.md) |
| 还没买任何东西 | [买什么、怎么买、别买错](docs/02-buying.md) |
| 已有 VPS 和 ISP | [从这里开始配置](docs/03-quickstart.md) |
| 想把整件事交给 Codex | 复制下面这段话 |
| 想生成客户端配置或订阅 | [配置与订阅的区别](docs/05-subscriptions.md) |
| 连不上、变慢、出口不对 | [按层排查](docs/08-troubleshooting.md) |
| 准备切换系统代理或测试断网 | **先读 [离线恢复](docs/07-recovery.md)** |

## 复制给你的 Agent

```text
请读 https://github.com/jingzuozoe-bot/vps-isp-guide 的 README.md、AGENTS.md、
docs/03-quickstart.md、docs/06-checkpoints.md 和 docs/09-platforms.md。
我的目标是按本库检查点建立“设备→加密 VPS→固定 ISP→互联网”的链路。
先只读盘点已有服务、设备、客户端版本、现网与回退方式，不要让我重复购买。
从第一个未完成的检查点开始；每步先用白话解释目的、影响、预期结果和回退。
凭据只在我本地受限文件中填写，不要求贴进聊天、Issue 或在线转换器。
普通代理可用与故障停连分别验收；没有证据就标未测。不要直接安装防火墙或锁应用。
请先告诉我当前阶段和下一项动作，并继续完成已授权的准备工作。
```

Agent 无法打开 GitHub？下载仓库 ZIP，解压后让它读本地 README.md 和 AGENTS.md。无需先学 Git。

## 一眼看懂你需要买什么

| 东西 | 首次建议 | 为什么 |
|---|---|---|
| VPS，例如搬瓦工 | 一台能从你网络稳定访问的 Linux KVM VPS；按线路、机房和流量挑选 | 中转不是算力比赛 |
| ISP，例如 IPRoyal | **ISP / Static Residential / Dedicated，先 1 个 IP、短周期** | 固定出口，避免误买轮换流量池 |
| 客户端 | 能导入对应协议、来自官方发行渠道的客户端 | 不同软件的配置不能混用 |
| 域名、订阅转换网站、大量 IP | 基础 SSH 参考路线无需购买 | 先跑通再增加组件 |

具体套餐、付款检查表和供应商陷阱见 [采购指南](docs/02-buying.md)。没有返佣链接。

## 按这个顺序走

**盘点 → 核订单 → 保存回退 → VPS 到 ISP → 本地到 VPS → 应用接入 → 分项验收 → 日常维护。**

| 层级 | 能说什么 | 不能据此说什么 |
|---|---|---|
| 采购完成 | 资源已开通 | 链路已经可用 |
| 普通代理通过 | 指定请求走了指定 ISP | 全机覆盖、DNS/IPv6/UDP 无旁路 |
| 故障停连通过 | 特定设备、版本、场景有负测试证据 | 所有设备永久安全 |

本库首条参考路线是 **macOS/Linux 电脑 + OpenSSH 本地转发 + ISP SOCKS5**，仅 TCP。手机、Windows、热点以及系统级停连须单独适配，见 [支持范围](docs/09-platforms.md)。

## 全部内容

[原理](docs/01-basics.md) · [采购](docs/02-buying.md) · [开工](docs/03-quickstart.md) · [参考配置](docs/04-reference-route.md) · [订阅](docs/05-subscriptions.md) · [验收](docs/06-checkpoints.md) · [恢复](docs/07-recovery.md) · [排错](docs/08-troubleshooting.md) · [设备支持](docs/09-platforms.md) · [手机路线](docs/14-mobile-roadmap.md) · [失败教训](docs/10-lessons.md) · [维护与费用](docs/11-maintenance.md) · [安全与账号](docs/12-security-and-accounts.md) · [官方来源](docs/13-sources.md) · [Agent 交接](templates/handoff.md)

欢迎补充可复现、已脱敏的设备适配记录。默认 [MIT 许可证](LICENSE)，第三方软件与商标适用各自条款；本库与供应商无隶属关系。
