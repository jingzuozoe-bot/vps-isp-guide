# 官方来源与时效

核验日：2026-10-25。供应商可随时更新商品、库存与后台；公开页面不是最终订单。只作必要摘要，不转载整篇文档。历史项目中的旧报价不作为当前购买依据。

| 来源 | 支持什么判断 |
|---|---|
| [搬瓦工商品页](https://bandwagonhost.com/vps-hosting.php) | KVM、自管理、控制台与可用系统 |
| [搬瓦工商品目录 JSON](https://bandwagonhost.com/order/catalog.json) | Agent 查当前目录入口；最终仍核结算 |
| [搬瓦工服务条款](https://bandwagonhost.com/terms-of-service.php) | 自管理、退款前提、全部服务终止和数据删除、禁止开放代理 |
| [IPRoyal ISP 价格](https://iproyal.com/pricing/static-residential-proxies/) | 周期与公开起价；不能代替整期订单总额 |
| [IPRoyal ISP 产品文档](https://docs.iproyal.com/proxies/isp) | 独享、协议、认证方式 |
| [IPRoyal ISP 购买和管理](https://iproyal.com/quick-start-guides/static-residential-proxies/) | 采购、协议、白名单、续费与公平使用 |
| [IPRoyal 私有/共享 FAQ](https://help.iproyal.com/en/articles/7222591-are-iproyal-proxies-private-or-shared) | 区分 ISP 产品与住宅轮换池 |
| [OpenSSH ssh 手册](https://man.openbsd.org/ssh) | -L/-D 区别、绑定地址、转发行为 |
| [curl 官方手册](https://curl.se/docs/manpage.html) | 配置文件、SOCKS 远端解析、超时 |
| [Mihomo SOCKS 配置](https://wiki.metacubex.one/en/config/proxies/socks/) | 字段参考，不等于设备实测 |

采购建议和起步硬件规格是本库的工程判断，非商家承诺。文档审阅、合成示例检查、真实设备实测是不同证据等级。

新增供应商前必须补官方商品、条款、计费、认证和续费来源；没核过就写待核，不把论坛促销帖当官方保证。

## 社区参考

本段与上面的官方来源表分开。这里只保存社区经验帖和社区工具仓库的原文链接，不把它们当作官方客户端或供应商文档，也不据此改采购、验收或路由。

| 项目 | 内容 |
|---|---|
| 原文链接 | https://x.com/gkxspace/status/2101993381303820704 |
| 作者 | @gkxspace（余温） |
| 是什么 | X Article / 长文社区经验帖，题为「被 Claude 封了 N 次之后，我终于搞定了全套海外基础设施（出海必备保姆级教程，2026.11.10）」 |
| 收录 | 推文发布于 2026-09-21；帖内称持续更新，最近注明更新 2026-11-10。2026-11-11 按仓库所有者要求收录此链接 |

| 项目 | 内容 |
|---|---|
| 原文链接 | https://x.com/gengdaJ/status/2108363527971041650 |
| 作者 | @gengdaJ（逸尘） |
| 是什么 | X Article，题为「任何人都能复刻的Claude注册、订阅和过KYC防封号教程」 |
| 收录 | 推文发布于 2026-10-09；帖内注明更新 2026-11-10。2026-11-11 按仓库所有者要求收录此链接 |

| 项目 | 内容 |
|---|---|
| 原文链接 | https://x.com/PanXing0827/status/2120662744047444400 |
| 作者 | @PanXing0827（王小丑） |
| 是什么 | X 长帖（note post），题为「目前最稳的一套 Claude 防封号方案」。从设备环境、时区、网络、代理、手机号和邮箱、支付方式、使用行为七个方面写社区经验，不是官方文档 |
| 收录 | 推文发布于 2026-11-11 23:39 UTC（台北时间 2026-11-12 07:39）。2026-11-12 按仓库所有者要求收录此链接 |

下表只列第二条帖子提到的相关公开链接。身份验证以 Anthropic 官方帮助为准。

| 相关链接 | 是什么 |
|---|---|
| https://x.com/gkxspace/status/2101993381303820704 | 本节已收录的 @gkxspace（余温）社区帖；第二条帖子引用了它 |
| https://github.com/ryuu-restart/agent-route | 社区路由/分流规则仓库，不是官方客户端或供应商文档 |
| https://support.claude.com/en/articles/14328960-identity-verification-on-claude | Anthropic 官方帮助：Identity verification on Claude |
| https://support.claude.com/zh-CN/articles/14328960-claude-%E4%B8%8A%E7%9A%84%E8%BA%AB%E4%BB%BD%E9%AA%8C%E8%AF%81 | 同一官方帮助的中文页：Claude 上的身份验证 |

下面这条是社区工具仓库，单独收录，不属于上表。

| 项目 | 内容 |
|---|---|
| 原文链接 | https://github.com/YOURSJEYO/claude-switcher |
| 作者 | YOURSJEYO |
| 是什么 | 社区开源的本机 Fail-Closed 账号环境管理器。用途是启动 Claude 前检查代理或 TUN 是否在跑、隧道断开时停掉 Claude 桌面版和 CLI，避免流量泄漏；也可多账号隔离。它不是软路由，也不改善出口名声。许可证是 AGPL-3.0 |
| 收录 | 2026-11-12 按仓库所有者要求收录此链接 |

这些帖子是关于海外环境，以及 Claude 注册、订阅、身份验证和账号使用环境的社区写法；Claude Switcher 是社区工具仓库。它们都不是 Anthropic、IPRoyal 或其他供应商的官方文档，也不是官方客户端。本库只保留链接作参考，不转载正文。收录链接不等于推荐现在安装，也不等于采用该项目或这些写法。出口 IP 不代替目标服务的地区、身份、支付和用途资格；本库不采用其中关于地区、身份、支付伪装或规避限制的步骤。见 [安全与账号](12-security-and-accounts.md)。
