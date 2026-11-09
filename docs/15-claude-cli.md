# 浏览器说登录成功，Claude CLI 为什么还报 403？

适用范围：macOS、zsh、已安装 Claude Code，桌面应用已经能通过预期代理联网。下面只给终端里的 `claude` 指定入口。

## 这次是怎么查出来的

1. **现象。** 浏览器提示授权成功，回到终端却看到 `Login failed: Request failed with status code 403`。`claude auth status` 显示 `"loggedIn": false`，另一个终端还出现了地区提示。
2. **线索。** 在本机分别测试后发现，默认终端请求和显式使用桌面代理的请求走了不同出口。检查 zsh 启动配置又发现，输入 `claude` 会先触发一个旧别名。它检查的是已停用的端口，连接失败便清除代理变量，再启动 CLI。
3. **修复。** 保留桌面设置，用已经测通的本地 HTTP 端口单独启动 Claude，绕开旧别名。先在当前终端登录，成功后才把同一启动方式写入 zsh 配置。
4. **验收。** 重新加载配置后，命令解析到了新别名，认证状态变为 `"loggedIn": true`，真实模型请求也收到了 `OK`。

浏览器完成授权后，CLI 还要自行联网完成登录。Claude Code 启动时读取代理环境变量。桌面能用，只能说明桌面那条路径可用。这个案例确认了旧端口与当前入口不一致；单看 403 仍无法判断是代理、账号资格还是服务端限制。[Claude Code 官方网络说明](https://code.claude.com/docs/en/network-config)

## 1. 先找入口，别猜端口

在原本报错的终端运行以下命令，输出只在本机查看：

```zsh
type -a claude
whence -p claude
claude --version
scutil --proxy
```

`type -a` 可以看出输入 `claude` 时是否先执行了别名或函数。`whence -p` 查找 PATH 中的可执行文件。如果没有路径，先按[官方入门指南](https://code.claude.com/docs/en/quickstart)检查安装，再排查代理。也可以在本机查看 `~/.zshrc` 里含 `alias claude`、`proxy`、`unset` 的行，留意有没有先探测旧端口或清除变量的逻辑。

在系统代理输出里找已启用的 HTTP/HTTPS 代理地址和端口。如果使用自动配置，或这里没有端口，去当前客户端设置中找“HTTP 代理端口”或“混合端口”。本指南假定入口在本机。本库 SSH 参考路线提供的是 SOCKS5；单有 SOCKS5 时，先确认客户端已有支持 HTTP CONNECT 的本地入口。把 SOCKS5 地址的前缀改成 `http://` 不会转换协议。

## 2. 用一条请求验证这个端口

下面的 `8080` 是示例，请换成上一步查到的值。在同一终端运行：

```zsh
port=8080
proxy="http://127.0.0.1:$port"
curl -q -sS --max-time 8 --noproxy '' --proxy "$proxy" \
  -o /dev/null -w '代理连接 %{http_connect}，目标响应 %{http_code}\n' \
  https://example.com/
```

这条命令丢弃网页正文，不查询公网 IP。出现“代理连接 200，目标响应 200”，说明这个入口能为该请求建立 HTTPS 隧道。连接被拒绝就回去核对端口和监听状态；连接中止则核对端口类型。若需要核验实际出口，可按[仓库验收步骤](06-checkpoints.md)在本地对照，只公开“匹配”或“不匹配”。

## 3. 先临时登录，发一条真实请求

在刚才的终端中运行下面这段。括号让代理变量随本次测试结束而失效。用可执行文件的路径启动，可以绕开已有的同名别名或函数。

```zsh
(
  cli=$(whence -p claude) || exit 1
  export HTTPS_PROXY="$proxy" HTTP_PROXY="$proxy"
  export https_proxy="$proxy" http_proxy="$proxy"
  "$cli" auth login &&
    "$cli" auth status &&
    "$cli" -p '只回复 OK'
)
```

按提示去浏览器授权，再回到发起登录的终端等待命令结束。看到 `"loggedIn": true`，并且最后收到模型回复，才算这次登录和调用都成功。若连接测试通过但登录仍报 403，再核对账号访问资格、服务支持地区，以及本机 `NO_PROXY` / `no_proxy` 是否让相关请求绕开代理。保留现有配置和认证目录；报告问题时只摘录错误类型和登录布尔值。

## 4. 保存给 Claude 专用，下次直接输入 claude

临时验证成功后，在 `~/.zshrc` **最末尾**追加下面一行。把这一行里四处 `8080` 都换成刚测通的端口。已有同名别名可以保留在前面。如果 `type -a claude` 显示多份安装，可把末尾的 `claude` 换成上一步验证过的可执行文件路径。

```zsh
alias claude='env HTTPS_PROXY=http://127.0.0.1:8080 HTTP_PROXY=http://127.0.0.1:8080 https_proxy=http://127.0.0.1:8080 http_proxy=http://127.0.0.1:8080 claude'
```

这个别名让 `env` 带着代理变量启动 Claude。打开一个新终端，逐行运行：

```zsh
which claude
claude auth status
claude -p '只回复 OK'
claude
```

第一行应显示刚写的别名；最后一行进入交互对话。以后在自己的项目目录打开终端，输入 `claude` 即可。需要撤销时，只移除自己追加的那一行，再打开新终端。
