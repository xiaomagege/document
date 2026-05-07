# Linux网络与远程运维命令：nc命令详解

`nc` 也叫 netcat，是 Linux 中非常实用的网络工具。它可以用于端口连通性测试、简单 TCP/UDP 通信、临时监听端口、发送数据等场景。

在排查“服务器能不能连到某个端口”时，`nc -vz host port` 是非常高频的命令。

---

## 一、`nc` 是什么？

`nc` 是 netcat 的常见命令名，被称为网络工具中的瑞士军刀。

它主要用于：

- 测试 TCP 端口是否可连接
- 测试 UDP 端口
- 临时监听本地端口
- 手工发送简单请求
- 在两台机器之间传输简单数据
- 排查防火墙、安全组、服务监听问题

不同系统中的 `nc` 版本可能不同，参数也可能略有差异，例如 OpenBSD netcat、GNU netcat、Nmap 的 `ncat`。日常使用前可以查看：

```bash
nc -h
```

---

## 二、常见使用语法与重点参数

```bash
nc [选项] 主机 端口
nc [选项] -l 端口
```

常用参数说明：

- `-v`：显示详细信息
- `-z`：只扫描端口，不发送数据
- `-u`：使用 UDP
- `-l`：监听模式
- `-p 端口`：指定本地端口，具体支持依版本而定
- `-w 秒`：设置超时时间
- `-n`：不做 DNS 解析

常见示例：

```bash
nc -vz example.com 80
nc -vz 127.0.0.1 22
nc -vzu 8.8.8.8 53
nc -l 8080
```

---

## 三、常用命令与使用场景

### 1. 测试 TCP 端口连通性

```bash
nc -vz example.com 80
```

含义：

- `-v`：显示详细输出
- `-z`：只测试连接，不发送数据

如果连接成功，说明从当前机器到目标主机的该 TCP 端口可达。

### 2. 测试本机端口是否监听

```bash
nc -vz 127.0.0.1 8080
```

适合确认本机服务端口是否能连接。

也可以配合 `ss` 查看监听：

```bash
ss -lntp | grep ':8080'
```

### 3. 设置超时时间

```bash
nc -vz -w 3 example.com 443
```

`-w 3` 表示最多等待 3 秒。

脚本中建议设置超时，避免长时间阻塞。

### 4. 测试多个端口

```bash
nc -vz example.com 80 443
```

部分版本支持一次测试多个端口。

也可以使用 shell 循环：

```bash
for p in 80 443 8080; do nc -vz -w 2 example.com $p; done
```

### 5. 测试 UDP 端口

```bash
nc -vzu 8.8.8.8 53
```

`-u` 表示使用 UDP。

UDP 无连接，结果不一定像 TCP 那样明确。即使显示成功，也要结合应用协议进一步验证。

### 6. 临时监听端口

```bash
nc -l 8080
```

在本机监听 8080 端口。

另一个终端可以连接：

```bash
nc 127.0.0.1 8080
```

两个终端之间可以输入文本进行简单通信。

### 7. 手工发送 HTTP 请求

```bash
printf 'GET / HTTP/1.1\r\nHost: example.com\r\n\r\n' | nc example.com 80
```

可以观察 HTTP 服务的原始响应。

日常调试 HTTP 更推荐使用 `curl`，但 `nc` 有助于理解底层文本协议。

### 8. 简单传输文件

接收端：

```bash
nc -l 9000 > file.txt
```

发送端：

```bash
nc 接收端IP 9000 < file.txt
```

这种方式适合临时测试，不适合替代 `scp`、`rsync` 等正式传输工具。

---

## 四、常见问题与建议

### 1. `nc -vz` 成功说明什么？

说明 TCP 三次握手层面可以连接到目标端口。

但这不代表应用协议一定正常。例如 443 端口能连通，不代表 HTTPS 证书、路径、鉴权都正常。

后续可以继续用：

```bash
curl -v https://目标
```

### 2. `ping` 通但 `nc` 不通是什么原因？

可能原因包括：

- 目标端口没有监听
- 防火墙或安全组拦截端口
- 服务只监听本地地址
- 中间网络策略阻断 TCP

先在服务端查看：

```bash
ss -lntp
```

再从客户端使用：

```bash
nc -vz host port
```

### 3. UDP 测试为什么不可靠？

UDP 没有连接建立过程，很多服务也不会对空包响应。

所以 `nc -u` 的结果只能作为参考，最好结合具体协议工具验证。

### 4. 生产环境使用 `nc` 有什么建议？

建议：

- 端口探测使用 `-w` 设置超时
- 只在明确授权范围内做端口测试
- 文件传输优先使用 `scp` 或 `rsync`
- HTTP/API 调试优先使用 `curl`
- 注意不同 `nc` 版本参数差异

---

## 五、常用命令速查

```bash
nc -vz example.com 80
nc -vz 127.0.0.1 8080
nc -vz -w 3 example.com 443
for p in 80 443 8080; do nc -vz -w 2 example.com $p; done
nc -vzu 8.8.8.8 53
nc -l 8080
nc 127.0.0.1 8080
printf 'GET / HTTP/1.1\r\nHost: example.com\r\n\r\n' | nc example.com 80
nc -l 9000 > file.txt
nc 接收端IP 9000 < file.txt
ss -lntp | grep ':8080'
curl -v https://目标
```

---

## 六、总结

`nc` 是非常实用的网络排查工具。建议优先掌握：

```bash
nc -vz host port
nc -vz -w 3 host port
nc -l port
nc -vzu host port
```

排查服务访问问题时，`ping` 只能看 ICMP 连通性，`nc` 可以进一步验证 TCP/UDP 端口是否可达。再结合 `ss` 和 `curl`，基本可以覆盖大部分服务连通性排查场景。