# 可检查的 TCP 参考路线：SSH 转发到 ISP

目的：尽量少加组件，验证“电脑→加密 VPS→ISP”，保留现有网络。这是手动参考方案，未在你的设备上预先验收。

## 路径要对

```text
测试应用 → 127.0.0.1:17891 → SSH 加密通道 → VPS → ISP SOCKS5 端点 → HTTPS 网站
```

这里使用 **ssh -L 转发到 ISP SOCKS5 端口**。单独使用 ssh -D 得到的是从 VPS 直出的 SOCKS，不会自动多一跳 ISP。

SSH 不给 VPS→ISP 段加密；普通 SOCKS5 也没有该保证。此路只转发 TCP，不能声称 UDP/QUIC 已覆盖。[OpenSSH 手册](https://man.openbsd.org/ssh)

## A. 确认服务器可管理

Agent 先检查系统、SSH 版本、现有账号和业务。新服务器从供应商控制台核对 SSH 主机公钥指纹，第一次连接按可信指纹确认，不设置 StrictHostKeyChecking=no。保留已登录管理会话和供应商控制台；先验证普通管理用户与公钥登录，再讨论收紧 SSH。别直接禁止所有密码或改防火墙导致自己进不去。

如果只有 root，先在供应商控制台按该发行版官方管理文档创建普通管理用户、配置公钥，验证第二个会话；此过程由 Agent 针对系统给出具体差异，不执行未知发行版的通用安装脚本。长期隧道宜用最小权限专用用户并限制转发目标；先核服务器实际 OpenSSH 版本与 PermitOpen 等选项，再单独验收。本库示例不自动修改 sshd。

## B. 在 VPS 单独验证 ISP

在 ISP 后台选 SOCKS5，记真实端点和端口，不能照抄示例端口。若用白名单认证，只增补 VPS 的实际出站 IP。若用密码认证，把凭据放本机/VPS 的受限 curl 配置，别贴聊天或写到命令历史。

在 VPS 的受限目录创建配置，参考 [curl 示例](../examples/curl-probe.example.conf)，将 proxy 改成订单中的 ISP 端点；仅需认证时填 proxy-user。由用户本地填写，Agent 不输出值。目录 0700，文件 0600；Windows 使用相应 ACL。

```sh
curl -q --config "$HOME/.config/vps-isp-private/isp-probe.conf"
```

配置中的测试 URL 为公开 HTTPS IP 回显服务，会看到该次请求的出口。返回值须在本地和订单期望 ISP 地址核对；不只看退出码。失败先查白名单、端口、认证、供应商状态，不跳去切本地系统代理。

## C. 本地创建 SSH 配置候选

将 [SSH 模板](../examples/ssh-config.example) 复制到仓库外，例如 `$HOME/.config/vps-isp-private/ssh.conf`。本地替换 VPS_HOST、SSH_USER、KEY_FILE、ISP_HOST、ISP_PORT 和已核验的 SSH_PORT；模板含占位符，不能直接运行。主机指纹须已通过可信渠道核对并保存。

确认 17891 未被占用，模板只监听 127.0.0.1。前台启动：

```sh
ssh -F "$HOME/.config/vps-isp-private/ssh.conf" -N isp-relay
```

它会持续占用窗口是正常的，不要因此认为卡死。ExitOnForwardFailure 只检查转发监听等建立问题，**不保证 ISP 可达**，仍需下一项请求验证。

## D. 在另一窗口显式探测

将 curl 示例复制为本地 `local-probe.conf`，proxy 保持 `socks5h://127.0.0.1:17891`；密码认证仍在此配置填写 ISP 凭据，SSH 不替你完成 ISP 认证。

```sh
curl -q --config "$HOME/.config/vps-isp-private/local-probe.conf"
```

`socks5h` 将目标域名交给 SOCKS 代理解析。它只影响这个请求，不能代替整机 DNS 验收。[curl 手册](https://curl.se/docs/manpage.html)

期望为指定 ISP，若为 VPS 或本地地址即失败。再用第二个独立 HTTPS 回显服务交叉检查，不上传代理凭据。先保存脱敏结果再停止测试。

## E. 可选客户端候选

[Mihomo 示例](../examples/mihomo.example.yaml) 用本地转发作为唯一出站，TUN 关闭，候选端口 17890，避免默认覆盖旧入口。按实际版本用客户端或核心的配置测试功能检查；导入独立配置，不覆盖旧订阅。只有需要时才在本地加入 username/password 字段。

示例无 DIRECT 后备，但不能阻止应用忽略系统代理；DNS 沿用系统设置，未声称防 DNS 泄漏；禁用模板 IPv6 也不是系统级 IPv6 封锁。[Mihomo SOCKS 文档](https://wiki.metacubex.one/en/config/proxies/socks/)

## F. 退出与负测试

只对显式测试请求做局部负测试：停止前台 SSH（Ctrl+C），重发同样代理请求，预期失败；期间不要让敏感应用自行尝试其他路径。这个结果证明该代理请求在隧道关闭时不可用，**不证明系统级停连**。

再启动隧道后复核出口。若曾设置系统代理，先退出受保护应用，按恢复卡恢复原值，再停新增组件。不得全局清空防火墙或删除原 VPN。
