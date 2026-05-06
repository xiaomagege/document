# linux常用命令：curl命令详解

`curl` 是 Linux 中最常用的网络请求工具之一。它可以访问 HTTP/HTTPS 接口、下载文件、测试请求头、调试 API、验证服务连通性，是运维、开发、测试都高频使用的命令。

相比浏览器，`curl` 更适合在服务器终端中快速验证接口行为和网络问题。

---

## 一、`curl` 是什么？

`curl` 用于通过 URL 和服务器传输数据，支持 HTTP、HTTPS、FTP 等多种协议。

它主要用于：

- 请求 HTTP/HTTPS 接口
- 下载文件
- 查看响应头
- 发送 GET、POST、PUT、DELETE 请求
- 携带请求头、Cookie、Token
- 调试接口状态码和耗时
- 测试服务连通性

日常最常见的使用场景是访问 Web 接口：

```bash
curl https://example.com
```

---

## 二、常见使用语法与重点参数

```bash
curl [选项] URL
```

常用参数说明：

- `-I`：只查看响应头
- `-i`：显示响应头和响应体
- `-L`：跟随重定向
- `-o 文件`：保存为指定文件名
- `-O`：使用远程文件名保存
- `-X 方法`：指定请求方法
- `-H 头`：添加请求头
- `-d 数据`：发送请求体数据
- `-u 用户:密码`：使用基础认证
- `-k`：忽略 HTTPS 证书校验
- `-s`：静默模式
- `-v`：显示详细请求过程
- `-w 格式`：自定义输出统计信息

常见示例：

```bash
curl https://example.com
curl -I https://example.com
curl -L https://example.com
curl -o index.html https://example.com
curl -X POST -H 'Content-Type: application/json' -d '{"name":"test"}' http://127.0.0.1:8080/api
```

---

## 三、常用命令与使用场景

### 1. 请求一个 URL

```bash
curl https://example.com
```

默认会把响应体输出到终端。

如果返回的是 HTML、JSON 或纯文本，会直接显示内容。

### 2. 查看响应头

```bash
curl -I https://example.com
```

适合查看状态码、服务器类型、缓存、重定向、Content-Type 等信息。

如果既要响应头又要响应体：

```bash
curl -i https://example.com
```

### 3. 跟随重定向

```bash
curl -L https://example.com
```

很多 URL 会返回 301 或 302 跳转，`-L` 可以自动跟随重定向。

### 4. 下载文件

```bash
curl -o app.tar.gz https://example.com/app.tar.gz
```

保存为指定文件名。

使用远程文件名保存：

```bash
curl -O https://example.com/app.tar.gz
```

### 5. 发送 POST 请求

```bash
curl -X POST http://127.0.0.1:8080/api
```

如果要发送 JSON：

```bash
curl -X POST \
  -H 'Content-Type: application/json' \
  -d '{"name":"test"}' \
  http://127.0.0.1:8080/api
```

### 6. 添加请求头

```bash
curl -H 'Authorization: Bearer token_value' https://api.example.com/user
```

常用于携带 Token、设置 Content-Type、模拟客户端请求等场景。

### 7. 查看 HTTP 状态码

```bash
curl -s -o /dev/null -w '%{http_code}\n' https://example.com
```

只输出状态码，适合脚本探活。

### 8. 查看请求耗时

```bash
curl -o /dev/null -s -w 'time_total=%{time_total}\n' https://example.com
```

也可以同时输出状态码和总耗时：

```bash
curl -o /dev/null -s -w 'code=%{http_code} time=%{time_total}\n' https://example.com
```

### 9. 显示详细请求过程

```bash
curl -v https://example.com
```

`-v` 会显示 DNS、连接、TLS、请求头、响应头等调试信息。

排查 HTTPS、代理、重定向和连接异常时非常有用。

### 10. 忽略 HTTPS 证书校验

```bash
curl -k https://self-signed.example.com
```

只建议用于测试环境或临时排查。

生产环境不要长期依赖 `-k`，应修复证书链问题。

---

## 四、常见问题与建议

### 1. `curl` 访问没有输出怎么办？

可能原因：

- 接口本身没有响应体
- 请求被重定向但没有加 `-L`
- 请求超时或连接失败
- 被服务端拒绝

可以使用：

```bash
curl -v URL
curl -I URL
```

先看连接和响应头。

### 2. 如何只判断服务是否正常？

可以只看状态码：

```bash
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8080/health
```

脚本中可以根据状态码判断是否健康。

### 3. `curl -X POST` 一定需要吗？

当使用 `-d` 发送数据时，`curl` 通常会自动使用 POST。

下面写法通常也可以：

```bash
curl -H 'Content-Type: application/json' -d '{"name":"test"}' http://127.0.0.1:8080/api
```

但为了可读性，很多人仍然会显式加 `-X POST`。

### 4. 生产环境使用 `curl` 有什么建议？

建议：

- 探活脚本使用 `-s -o /dev/null -w` 输出关键指标
- 调试问题使用 `-v`，不要长期在脚本中保留敏感输出
- Token、密码不要直接写进可共享脚本
- 下载文件后校验大小或哈希
- 不要在生产中长期使用 `-k` 忽略证书问题

---

## 五、常用命令速查

```bash
curl https://example.com
curl -I https://example.com
curl -i https://example.com
curl -L https://example.com
curl -o app.tar.gz https://example.com/app.tar.gz
curl -O https://example.com/app.tar.gz
curl -X POST http://127.0.0.1:8080/api
curl -X POST -H 'Content-Type: application/json' -d '{"name":"test"}' http://127.0.0.1:8080/api
curl -H 'Authorization: Bearer token_value' https://api.example.com/user
curl -s -o /dev/null -w '%{http_code}\n' https://example.com
curl -o /dev/null -s -w 'code=%{http_code} time=%{time_total}\n' https://example.com
curl -v https://example.com
curl -k https://self-signed.example.com
```

---

## 六、总结

`curl` 是 Linux 下调试 HTTP/HTTPS 服务和接口的核心工具。建议优先掌握：

```bash
curl URL
curl -I URL
curl -L URL
curl -o file URL
curl -s -o /dev/null -w '%{http_code}\n' URL
```

日常排查中，先用 `curl -I` 看响应头和状态码，再用 `curl -v` 看详细连接过程，可以快速判断问题在网络、证书、重定向、服务端还是接口本身。