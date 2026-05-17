# 31｜网络连通与端口排查｜接口到底通没通？curl 是排查 HTTP 问题的第一把刀

HTTP 问题最容易卡在一种很模糊的状态里：页面打不开、接口超时、健康检查失败，但你一时说不清是 DNS、网络、证书、重定向，还是应用自己返回错了。

`curl` 的价值就在于，它能把这类模糊问题迅速拆成几个具体问题：

- URL 能不能访问
- 返回的状态码是什么
- 重定向到哪里了
- 响应头和耗时是什么
- TLS 或证书有没有问题

如果只想先记 3 条：

- 想看响应头和状态码，先用 `curl -I URL`
- 想看详细过程，直接用 `curl -v URL`
- 想做探活或脚本判断，优先用 `curl -s -o /dev/null -w`

## 一、什么时候先用 `curl`

这些场景都应该先想到 `curl`：

- 接口到底通没通还不确定
- 健康检查失败
- 页面打不开，但你想先排除 HTTP 层问题
- 想确认状态码、重定向、响应头
- 想从服务器上直接测某个内部 URL

`curl` 最适合回答的是：

**HTTP 这一层到底发生了什么？**

## 二、先记住这几条命令

```bash
curl -I https://example.com
curl -v https://example.com
curl -L https://example.com
curl -s -o /dev/null -w '%{http_code}\n' https://example.com
curl -o /dev/null -s -w 'code=%{http_code} time=%{time_total}\n' https://example.com
curl -H 'Authorization: Bearer token' https://api.example.com/user
```

它们分别适合：

- `-I`：只看响应头
- `-v`：看连接、TLS、请求头、响应头细节
- `-L`：跟随重定向
- `-w`：自定义输出状态码和耗时
- `-H`：带认证头或其他请求头

## 三、接口排查时最常见的 4 步

### 1. 先看能不能返回 HTTP 头

```bash
curl -I https://example.com
```

这一步最适合快速确认：

- 有没有响应
- 状态码是多少
- 是不是 301 / 302 跳转
- 服务器类型、缓存头是否异常

### 2. 如果还不清楚，再看详细过程

```bash
curl -v https://example.com
```

这一步适合继续判断：

- DNS 解析是否正常
- TCP/TLS 建连是否成功
- 证书有没有问题
- 请求是不是被代理、重定向或拒绝

### 3. 如果有跳转，记得跟随

```bash
curl -L https://example.com
```

很多地址看似“访问失败”，其实只是没有跟随 301 / 302。

### 4. 如果是探活或脚本判断，只保留关键结果

```bash
curl -s -o /dev/null -w '%{http_code}\n' https://example.com
```

或者同时看耗时：

```bash
curl -o /dev/null -s -w 'code=%{http_code} time=%{time_total}\n' https://example.com
```

这类写法很适合健康检查和简单巡检脚本。

## 四、最常见的 4 个判断场景

### 1. 返回 200，但页面依然不对

这通常说明网络层没问题，应该继续往：

- 应用逻辑
- 页面内容
- 认证鉴权

方向排查。

### 2. 返回 301 / 302

这通常不是“不可达”，而是跳转。

用 `-L` 继续跟，确认最终落点。

### 3. 连不上或者卡很久

这时优先用 `-v` 看是：

- DNS 卡住
- TCP 建连失败
- TLS 握手失败
- 上游服务本身不响应

### 4. 端口能连，但接口还是异常

这时 `nc` 和 `ss` 只能说明端口层面没问题，HTTP 层细节还得靠 `curl`。

## 五、和 `ss`、`nc` 怎么分工

- `ss`：看本机监听和连接状态
- `nc`：测端口可达
- `curl`：看 HTTP/HTTPS 层真实返回

一个常见排查顺序是：

1. 服务端 `ss -lntp`
2. 客户端 `nc -vz host port`
3. 然后 `curl -I` 或 `curl -v`

这样能把“服务没监听”“端口不通”“HTTP 返回异常”三层分开。

## 六、最常见的误区

### 1. 一上来就只看响应体

很多时候真正有价值的是状态码、响应头、重定向和 TLS 过程，而不是页面正文。

### 2. 明明是跳转，却忘了 `-L`

这会让很多本来正常的地址看起来像“没返回对”。

### 3. 长期依赖 `-k`

```bash
curl -k https://...
```

只适合临时排查。生产环境里长期忽略证书问题，只会把真实风险藏起来。

## 七、结论

`curl` 是排查 HTTP 问题时最直接的一把刀。

先用 `-I` 看头，再用 `-v` 看过程，需要脚本化时再用 `-w` 提取状态码和耗时，这样最省力。
