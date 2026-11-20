# 路由分流（待验，先不部署）

2026-11-20 定下的原则：路由器到位，并把现有 VPS→ISP 做成网络层透明代理之后，目标不再是「全屋所有流量都走 ISP」。很多国内站点在这条出口下打不开，必须分流。

现网配置不动。路由验收并实际部署之前，不把这套分流装上路由器，也不在本机开 TUN，不安装 Antify 来预演。本文只记结论、原因和边界，没有可执行的分流规则。

分流不能把机房 IP 或广播 IP 变成干净家宽，也不代替地区和身份资格。出口 IP 不代替资格。见 [安全与账号](12-security-and-accounts.md)。

## 用词

用户说的「内网」是家庭宽带，也就是国内直连。用户说的「外网」是已经买下的 VPS→ISP 指定出口。这里不是局域网和公网的那一对意思。局域网、Tailscale 远程路径是另一件事，验收时不要把它们吸进 ISP 代理。

## 分流原则

不限于 Claude。ChatGPT、OpenAI 和其他境外服务，也走同一条 ISP 出口。只让 Claude 走 ISP、别的境外服务走家宽，会把同一台机器拆成两个出口。

最终路径可能会采用国内白名单：国内走家庭宽带直连，境外走这条 ISP 出口。「可能会采用」是方向，还没有锁定某一份规则集。

这是「国内直连，其余境外流量走同一条 ISP」。不是只把 Claude 官方域名钉到代理上。只钉官方域名会漏掉遥测和相关请求，那些请求若从家宽出去，就变成 ISP 和家宽两个出口。

社区清单只用来看补齐性和容易漏的域名。不原样导入 one-exit、v2fly、Loyalsoldier、blackmatrix7、ip.net.coffee。收录不等于采用。夙愿学长条目里的 claude-cleanup 已经收录，边界仍是不执行，这里不重复收。来源见 [社区参考](13-sources.md#社区参考)。

## 验收还要看的事

验收通过之前不部署。通过之后要核对四件事：

1. 境外流量的出口是不是所购的 ISP。
2. 这条出口断开时，境外流量不自动回落到家庭宽带。这是断网保护，也叫 fail-closed。
3. DNS、WebRTC、IPv6 有没有泄漏。
4. 局域网和 Tailscale 远程路径不进 ISP 代理。

验收之后，不要在路由透明代理上面再叠本机 TUN 或 Antify。

## 待验时再核对的技术点

下面四条是以后验收时的提示，不是现在改配置。

1. 官方 Claude Code 企业网络配置列出了 `http-intake.logs.us5.datadoghq.com` 和 `browser-intake-us5-datadoghq.com`：[network-config](https://code.claude.com/docs/en/network-config)。后者中间是连字符，主机名是 `browser-intake-us5-datadoghq.com`，不是 `datadoghq.com` 下面的子域。`DOMAIN-SUFFIX,datadoghq.com` 抓不到它。追单个域名不是本库的实现方案；要的是境外流量同一条出口，不是一份钉死的域名表。
2. NTP 只对 UTC 时钟。它不返回本机时区，也不改本机时区。把 NTP 说成时区风控开关是错的，不采纳。
3. Anthropic 的 IPv6 段 `2607:6bc0`：one-exit 建议拒绝，理由是避免住宅代理 502；ip.net.coffee 建议走代理。两份建议相反，留给路由验收时再测。现在不选边。
4. 把支付、Persona 等核验域名对齐到同一条出口，不是官方政策。本库不采用这类步骤。

Magpie 把 VPS 上的 Claude CLI 传回本机，不是已经定下的 Mini 远程方案三。方案三见 [Mini 远程](16-mini-remote.md)。干净 VPS 通常仍是机房 IP，不等于干净家宽。不部署 Magpie，不把网关暴露到公网。

本文是 2026-11-20 的原则记录。落地时再按当时的官方网络配置核对域名，不以本文代替原文。
