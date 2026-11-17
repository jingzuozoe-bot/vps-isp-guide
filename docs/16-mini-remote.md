# Mini 远程操作 Claude Code

定案是方案三。Mini 安装 Claude 桌面端，登录独立新账号，工作文件放在 Mini。人在 Air 或外出时，用远程桌面点 Mini 上已经打开的那个窗口。Claude 进程、文件和模型请求都留在 Mini，从 Mini 所在网络出去。Air 继续用现有 $20 账号自己的桌面端，不在 Air 上登录 Mini 的账号。

路由器做成整屋 VPS→ISP 透明代理、并且验收通过之前，不部署本文任何远程或 Claude 配置。Air 维持现在的系统代理 `127.0.0.1:7890`，TUN 保持关闭。本文是 2026-11-17 的方案记录和边界，不是可以现在照着做的操作手册。

出口 IP 不代替官方资格。远程只解决人在哪台机器上点击，并减少流量漏回家庭线路。它不把 ISP 出口变成「干净」，也不保证账号不弹核验。见 [安全与账号](12-security-and-accounts.md)。

## 背景与目标（截止 2026-11-17）

用户已经买了全球海外版路由器，准备把现有 VPS→ISP 做成整屋透明代理。验收完成前，Air 的网络配置不动：系统代理仍是 `127.0.0.1:7890`，TUN 未开。

Mini 以前连过普通 VPN，没有跑过 Claude，按新环境对待。计划把它当成长期开机的工作机，用来剪视频和跑重任务，订阅打算用 $100 或 $200。Air 保留现有 $20。两台机器用两个 Claude 账号。要隔离，是因为 Claude 环境不稳定，一个账号出问题不能把另一台一起卷进去。

人可以在 Air 前，也可以在外面。点击发生在 Air（或手机）的图形界面上。真正的 Claude 进程、工作文件和 API 请求必须在 Mini 上，并走 Mini 当时所在网络的出口。Mini 不是 Air 的远程硬盘。也不做桥接，不让 Mini 上的 Claude 回头操作 Air。

判断出口只看一件事：发出模型请求的，是正在跑 Claude 进程的那台机器。进程在 Mini，请求就从 Mini 出。进程在 Air，请求就从 Air 出。远程桌面搬过去的是画面和键鼠，不把进程搬回 Air。

## 讨论先后顺序（必须保留，供后人看懂为什么这么改）

1. **2026-11-14：家里一台干净电脑，怎样让 Air 和手机远程用。** 初次结论是 Claude Code 只装、只跑在 Mini，文件也放 Mini；Air 和手机只当遥控器。当时主推官方 Remote Control，加上 SSH 或 Desktop SSH；屏幕共享只作为看桌面的备选；网易 UU、SSH 加 tmux 再作备选。出口原则在这一步就定了：请求从跑着 `claude` 进程的那台机器出去。Cursor 自带 AI 走 Remote SSH 时仍从本机出网，本机在这个场景里是 Air。依据是 [Cursor 网络说明](https://cursor.com/help/troubleshooting/network)：Remote SSH 扩展的 AI 功能在本地机器上运行，向远端要文件；AI 请求从本地机器发往 Cursor 的服务器，不从远端主机发出。所以不能把「Cursor 远程连上 Mini」当成「Claude 或 Cursor 的模型请求已经从 Mini 出去」。

2. **2026-11-17：对照林森的长文。** 帖子是 [《全网最稳定使用 Claude 的办法》](https://x.com/ChappelDae55911/status/2118615751435358466)，社区来源里已有这条链接。相同的只有一层：手机当入口，Claude 跑在一台常开的机器上。他的常开机器是温哥华机房 VPS，入口是挂到公网的网页工作台，再用 SSH 桥接去操作本机。这里不照搬。能执行命令的网页工作台不挂公网。Claude 和文件都在 Mini，不需要桥接回 Air。「手机不开梯子也能用」只说明控制通道能不能连上，和账号资格无关。人在外面用官方 App 或 Remote Control 时，会在手机当时的网络上登录那个账号；这一条后面直接决定了方案二不能当隔离主路。

3. **同日：社区帖只作来源，验收加泄漏检查。** 起风了的汇总帖，以及其中引用的丁小屁（@dingxiaopi）、苍何、玉龙帖，收进了本 PR 的社区来源。路由验收要加测 DNS、WebRTC、IPv6 泄漏。不导入这些帖里的分流规则，也不采用其中的防封做法。分流会把一部分流量送回直连，和「整屋走指定 ISP 出口」不是同一条路线。

4. **用户要 Mini 的全方案。** 当时收成四层：先验收路由；Mini 当唯一干活的机器；再做远程；明确有一批不做的事。订阅顺序当时仍写成：先尝试把 Air 的 $20 升到 $100，升不成就再配 Mini。这一句后来作废，见第 7 步。

5. **Mini 跑 CLI 还是桌面端。** 当时建议 CLI。理由是 Remote Control 从 CLI 启动，SSH 加 tmux 断线后进程还在，常开机器不必保持桌面登录。图形界面准备用 Air 上的 Desktop SSH 连过去看。这一步把「Mini 只跑 CLI」说死了，第 7 步修正。

6. **要不要按林森做：CLI 在 Mini，Air 上开工作台再桥接。** 答案是不用。Claude 和文件都在 Mini，桥接没有对象。CloudCLI / Claude Code UI 如果以后真的要用，只能开在 Tailscale 内网里，不能挂公网。当时仍把官方 Remote Control 放在网页工作台前面。

7. **需求收束，并改掉两处旧结论。** 用户确认：两个账号；Mini 要干净，并上高额订阅；人习惯在 Claude Code 里点图形界面；人在哪里，都要能在 Air 上点界面，实际跑的是 Mini 上的 Claude。从这里把候选收成下面三条。同时修正第 5 步：Mini 可以安装桌面端，不是只能跑 CLI。第 4 步的「先升 Air，再配 Mini」改为：路由验收通过后，直接配 Mini 的干净环境。Air 的 $20 留在 Air。

8. **Desktop SSH 会不会仍是 Air 本机的 Claude；方案一和方案二是不是同一件事；Remote Control 在 Mini 上是 CLI 还是桌面端。** Desktop SSH 的官方形态是外壳在 Air、引擎在 Mini：进程、文件和模型请求在 Mini，不是 Air 本机 Claude 把 Mini 当硬盘。Remote Control 可以从 CLI 的 `claude remote-control` 启动，也可以从桌面端的 `/remote-control` 启动。方案一和方案二的机制不同，共同点只有一个：Air 或手机要登录 Mini 那个账号。

9. **按提示词核对官方文档。** 用户让其他 AI 核对，三份结果一致：三条都能让进程、文件和模型请求从 Mini 出去。方案一和方案二会在 Air 上登录 Mini 的账号。只有方案三不会。Cursor 自带 AI 仍从 Air 出，以第 1 步的 Cursor 官方说明为准。落地前要再对一次当时的官方文档，三份旧核对不能代替当天的页面。

10. **两个账号要隔离时，三条的账号边界。** 方案一不是「两台电脑只能共登一个号」。它是：Air 上那个连到 Mini 的 Desktop 窗口，必须登录 Mini 的账号，凭据在 Air 侧续期。方案二的控制端必须登录同一个账号。只有方案三能让 Mini 的账号不出现在 Air 上。

11. **定案。** 用户确认：要账号隔离，又要有成熟的远程桌面软件，只能选方案三。Mini 直接打开 Claude 桌面端，选 Mini 本地环境。Air 用远程桌面去点那个窗口。

## 三条方案对照

### 方案一：Air 安装 Claude Desktop，用官方 SSH 远程环境连 Mini

官方 SSH 会话是外壳在 Air、引擎在 Mini。Air 上看到的是 Desktop 的界面，读写的文件和拉起的进程在 Mini，模型请求也从 Mini 发出。

优：官方原生图形界面。进程、文件、模型请求都在 Mini。这不是「Air 本机的 Claude 把 Mini 当成一块远程硬盘」。

劣：Air 上的这个 Desktop 窗口必须登录 Mini 的账号，凭据由 Air 侧续期。Air 断开或睡眠之后，远端任务可能因为登录过期停掉。Mini 的账号会出现在 Air 上，两个账号隔离不成立。

不选。用户要的是两个账号彻底分开。

### 方案二：Mini 开 Remote Control，Air 用浏览器或手机 App 接

Mini 上可以用 CLI 的 `claude remote-control`，也可以用桌面端的 `/remote-control`。Air 打开浏览器里的 claude.ai/code，或手机 Claude App 的 Code 页，去接这个会话。

优：官方的浏览器和手机入口。执行和模型请求在 Mini。Mini 不必为这条路开放入站端口。

劣：控制端必须登录同一个账号，Air 或手机上会出现 Mini 的账号。信令经过 Anthropic 的服务器，会话记录会暂存。有些命令只能在本机终端里用。Code 页上云端会话和远程会话排在一起，点错会跑到云上，而不是 Mini。

不选，原因和方案一相同：Mini 的账号会被登到 Air 或手机。以后如果接受「操作端登录同一个号」，它可以当外出时补一句的入口，不当隔离主路。

### 方案三（已定）：Mini 开 Claude 桌面端，Air 和手机用远程桌面去点

Mini 安装并打开 Claude 桌面端，环境选 Mini 本地。Air 用 macOS 自带的屏幕共享去看、去点这个窗口。手机用 Screens 或 Jump Desktop。连接只走局域网或 Tailscale。不用 Tailscale Funnel，不把 SSH 或 VNC 暴露到公网。

优：账号只登录在 Mini。Air 不必打开 Claude，也不登录那个新账号。进程、文件、请求都在 Mini。远程桌面是已经成熟的软件。人和坐在 Mini 前操作，是同一回事：点的是 Mini 上的窗口，跑的是 Mini 上的进程。

劣：这不是 Claude 的官方远程能力，Anthropic 没有专门认证「桌面端加屏幕共享」这一组合。流畅程度取决于屏幕共享。Mini 的桌面要保持登录、机器常开、不睡眠，否则窗口不在，远程过去没有可点的会话。

选定它，是因为它同时满足三件事：两个账号隔离，人在 Air 上点图形界面，进程和请求在 Mini。前两条方案都做不到第一件。远程桌面软件已经有现成产品，不必再造一条网页工作台。

## 路由验收后的技术路线

下面五条是给以后 Codex 的落地约束。路由器透明代理验收通过之前，一条都不执行。这里没有安装命令，也没有配置片段。

1. 先把路由器配成整屋 VPS→ISP 透明代理。Mini 本机不再开代理、不再开 TUN，不和路由叠在一起。验收要看到四件事：出口是所购的 ISP；线路断了必须断网，不回落到家庭宽带；DNS、WebRTC、IPv6 不泄漏；远程通道（局域网或 Tailscale）不进 ISP 代理环。远程通道如果也被透明代理吸进去，人在外面会和 Mini 互相绕进代理，画面和键鼠一起断。

2. Mini 常开、不睡眠，桌面保持登录。只在 Mini 上安装并登录 Claude 桌面端，用新账号。Claude 的案卷和工作目录放 Mini；正在剪的工程放 Air，交接见下文「机器分工、文件交接与订阅边界」。第一次登录 Claude 之前，上一条的出口验收必须全部通过。出口还没验收就登录，等于把新账号的第一次请求交到一条还没证明的线路上。

3. Air 用 macOS 系统屏幕共享。手机用 Screens 或 Jump Desktop。连接走局域网或 Tailscale。Air 上已经有 Tailscale，不要假定 Mini 已经安装。不用 Funnel，不把 SSH 或 VNC 映射到公网。公网入口等于把 Mini 的桌面暴露到互联网。

4. Air 现有 $20 账号只用 Air 自己的桌面端。不在 Air 本地再登录 Mini 那个号。一旦在 Air 上登录，方案三的隔离就退回方案一。

5. 不做这些事：导入分流规则；把网页工作台挂到公网；桥接回 Air；在 Air 上跑 Claude、只把 Mini 当硬盘；把 Cursor 自带 AI 当成已经走了 Mini 的出口；在 Mini 上再开 TUN，和路由透明代理叠在一起。

## 机器分工、文件交接与订阅边界（2026-11-17）

两台机器不是所有东西都割开。分开的是三条边界。

账号：A（现有 $20）只在 Air。B（计划 $100 或 $200）只在 Mini。Air 用远程桌面去点 Mini 上已经登录 B 的桌面端窗口。Air 本机的 Claude 不登录 B。

执行：Claude Code 在 Mini。剪辑在 Air。

资料：按需交接，不做两台全盘互相同步。两台可以同时各干各的。同一份正在改的文件只允许一台机器动。

### 硬件怎么分

Air 是 32G 内存、1T 内置盘，做前台。正在剪的工程放内置硬盘，不把外接盘当成活跃工程盘。预览、改稿、外出，以及 A 账号的短对话，都在 Air。

Mini 是 16G 内存、2T 外接盘，做常开后端。B 账号的 Claude 桌面端、长文档、批量整理、素材和成片归档，放在 Mini。

口诀：Air 剪、交互、出活；Mini 跑 Claude、存货、跑长任务。

Mini 只有 16G。不要在它上面同时开剪辑软件和大浏览器。外接盘是高速 SSD 时，才可以当工作盘；机械盘只用来归档。2T 不能是唯一副本，重要案件和成片要另备一份。这是初步分工，不是性能测试结论。大批 OCR、浏览器和数据库同时跑的时候，再看会不会卡。外接 SSD 不增加内存。

### 文件分三层

1. 正在剪的工程只放 Air。定稿之后单向归档到 Mini。不做双向实时同步，避免工程锁和素材脱链。成品可以用 rsync 或 Syncthing 单向送到 Mini。
2. 给 Claude 用的案卷、文案、输入和输出，优先用 Syncthing，走 Tailscale 内网对传。目的是隐私：证据不默认上传公网。写入约定是 Claude 工作目录只允许 Mini 写。不要两台同时改同一个文件。同步排除 Claude 会话缓存、MCP 缓存，以及大剪辑工程目录。目录名可以约定为 `01_Claude_Input`、`02_Claude_Output`、`03_Archive`。这三个名字只是约定，落地时再配，现在不建同步任务。
3. 飞书只做协作基座：任务、进度、哪台机器正在改、当前版本在哪、对外交付。不要把 Claude 工作目录挂成飞书云盘。客户材料的原件上不上云，另作决定，不跟这一层绑在一起。

### 诉讼材料怎么交

原件保留，不改。Mini 保存原件，并在旁边生成派生文件。交出去之前写明当前编辑位置。交到 Air 修改之后，以 Air 上的工作稿为准。定稿后交回 Mini，标明日期和版本。飞书记下状态和文件位置。另外单独备份一份。

剪辑用同一套规则。当前工程和这次要用的素材放在 Air 本地。做完后，工程、素材和成片一起归档到 Mini。日常剪辑不要建立在 Air 每次经网络去读 Mini 上的素材。人在外面时，这条更不能当工作方式。

### 订阅停在哪一条线上

订阅权益在 Claude 账号上，不在 Apple ID 上，也不在某一台 Mac 上。Mini 的系统 Apple ID 不必为了订阅再单独搞一套。

路由验收通过之前，不要在 Mini 上为 B 开通新订阅。到了那一步，固定一种付款方式，账单在 Mini 上管理。不要在 Air 上登录 B 去改付款。用网页还是 Apple、用哪张卡，验收时再按当时能稳定用的那一条当面决定。这里不记礼品卡、换区或社区支付渠道，也不写开通步骤。

## 资料链接

只放正文链接，没有分享参数。官方页面会改版。落地时按当时页面再核对一次，不以本文转述代替原文。

官方：

- https://code.claude.com/docs/en/desktop
- https://code.claude.com/docs/en/desktop#ssh-sessions
- https://code.claude.com/docs/en/desktop#files-and-processes-on-the-remote-machine
- https://code.claude.com/docs/en/desktop#remote-machine-requirements-for-ssh-sessions
- https://code.claude.com/docs/en/desktop#keep-a-task-running-while-desktop-is-disconnected
- https://code.claude.com/docs/en/desktop-quickstart
- https://code.claude.com/docs/en/remote-control
- https://code.claude.com/docs/en/mobile
- https://support.claude.com/en/articles/14552382-your-first-day-in-claude-code
- https://claude.com/docs/third-party/claude-desktop/ssh-remote-sessions （企业托管版的补充说明。消费版的细节不以这一页为全部依据。）
- https://cursor.com/help/troubleshooting/network
- Apple 屏幕共享：https://support.apple.com/guide/mac-help/share-the-screen-of-another-mac-mh14066/mac 与 https://support.apple.com/guide/mac-help/turn-screen-sharing-on-or-off-mh11848/mac
- https://blog.edovia.com/en/screens-5.10-high-performance-screen-sharing
- https://help.edovia.com/en/screens-5/getting-started/connecting
- https://www.jumpdesktop.com/
- https://docs.jumpdesktop.com/connect/
- Tailscale：https://tailscale.com/docs/features/tailscale-ssh 与 https://tailscale.com/docs/features/tailscale-funnel （不用 Funnel 暴露 SSH 或 VNC。）

社区对照只记架构，不采用防封教程。已经收进 [社区参考](13-sources.md#社区参考) 的，这里只交叉引用：

- 林森 https://x.com/ChappelDae55911/status/2118615751435358466
- CloudCLI / Claude Code UI https://github.com/siteboon/claudecodeui
- 卡兹克的网易 UU 方案：https://www.uisdc.com/agent-anywhere 、https://uuyc.163.com/

本文是 2026-11-17 的方案记录与边界，不是验收通过之后的逐步配置手册。落地时再按当时官方文档核对一次版本变更。
