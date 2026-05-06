# linux常用命令：traceroute命令详解

当网络访问慢、跨机房访问异常、某个公网地址无法到达时，只知道 `ping` 通不通还不够。我们还需要知道数据包经过了哪些路由节点，在哪一跳开始延迟升高或丢包。

`traceroute` 就是用于追踪网络路径的常用命令。它可以帮助我们观察从本机到目标主机之间经过的路由跳数和每一跳的响应时间。

---

## 一、`traceroute` 是什么？

`traceroute` 用于显示数据包从本机到目标主机经过的路由路径。

它主要用于：

- 查看访问目标经过哪些网络节点
- 判断网络延迟从哪一跳开始升高
- 排查跨网段、跨运营商、跨地域访问问题
- 判断目标不可达大致卡在哪个位置
- 辅助分析公网链路质量

需要注意的是，`traceroute` 结果只能作为线索。中间节点不响应并不一定代表网络中断，很多路由器会限制或丢弃探测报文。

---

## 二、常见使用语法与重点参数

```bash
traceroute [选项] 目标主机
```

常用参数说明：

- `-n`：不解析域名，直接显示 IP，速度更快
- `-m 跳数`：设置最大跳数
- `-q 次数`：每一跳发送的探测包数量
- `-w 秒`：等待响应超时时间
- `-I`：使用 ICMP 探测
- `-T`：使用 TCP 探测
- `-p 端口`：指定目标端口，常和 `-T` 配合

常见示例：

```bash
traceroute example.com
traceroute -n example.com
traceroute -n -m 20 example.com
traceroute -T -p 443 example.com
```

---

## 三、常用命令与使用场景

### 1. 查看到目标的路由路径

```bash
traceroute example.com
```

会显示每一跳的路由节点和响应时间。

每一行通常代表一跳，后面显示多次探测的耗时。

### 2. 不解析域名，加快输出

```bash
traceroute -n example.com
```

`-n` 会直接显示 IP，不做反向 DNS 解析。

排查问题时通常建议加 `-n`，输出更快也更稳定。

### 3. 限制最大跳数

```bash
traceroute -n -m 20 example.com
```

最多追踪 20 跳。

默认最大跳数通常已经够用，但在一些复杂网络中可以调整。

### 4. 调整每跳探测次数

```bash
traceroute -n -q 1 example.com
```

每跳只发 1 个探测包，输出更快。

如果需要更稳定的观察，可以增加次数：

```bash
traceroute -n -q 5 example.com
```

### 5. 使用 ICMP 探测

```bash
traceroute -I example.com
```

有些环境中默认 UDP 探测可能被拦截，使用 ICMP 可能更接近 `ping` 的路径表现。

### 6. 使用 TCP 探测端口

```bash
traceroute -T -p 443 example.com
```

使用 TCP 探测目标 443 端口。

当排查 HTTPS 访问问题时，这种方式可能比默认探测更贴近真实业务流量。

### 7. 调整等待时间

```bash
traceroute -n -w 2 example.com
```

每次探测最多等待 2 秒。

网络较慢时可以适当增加，想快速输出时可以适当减少。

---

## 四、常见问题与建议

### 1. 输出中的 `* * *` 是什么意思？

表示这一跳没有收到响应。

常见原因包括：

- 路由器不回复探测报文
- 防火墙丢弃相关报文
- 网络设备限速 ICMP/UDP/TCP 探测
- 该跳确实存在丢包或不可达

如果后续跳数还能继续显示，说明该跳不响应不一定是故障点。

### 2. 中间某一跳延迟高，是不是一定有问题？

不一定。

一些路由器会降低对探测报文的响应优先级，导致这一跳显示延迟高，但后续跳延迟正常。

更应该关注：

- 延迟是否从某一跳开始持续升高
- 后续所有跳是否都受影响
- 目标端是否也有高延迟或丢包

### 3. `ping` 通但 `traceroute` 不完整怎么办？

这很常见。不同网络设备对 ICMP、UDP、TCP 探测处理方式不同。

可以尝试：

```bash
traceroute -I 目标
traceroute -T -p 443 目标
```

### 4. 生产环境排查网络问题建议怎么做？

建议组合使用：

```bash
ping -c 4 目标
traceroute -n 目标
traceroute -T -p 端口 目标
curl -v URL
nc -vz 目标 端口
```

从连通性、路径、端口、应用响应逐层判断。

---

## 五、常用命令速查

```bash
traceroute example.com
traceroute -n example.com
traceroute -n -m 20 example.com
traceroute -n -q 1 example.com
traceroute -n -q 5 example.com
traceroute -I example.com
traceroute -T -p 443 example.com
traceroute -n -w 2 example.com
ping -c 4 example.com
curl -v https://example.com
nc -vz example.com 443
```

---

## 六、总结

`traceroute` 是查看网络路径的常用工具，适合排查访问慢、链路异常、跨网络不可达等问题。建议优先掌握：

```bash
traceroute -n 目标
traceroute -I 目标
traceroute -T -p 端口 目标
```

看 `traceroute` 结果时，不要只盯着单个中间节点。真正有价值的是观察延迟和丢包是否从某一跳开始持续影响后续路径，并结合 `ping`、`curl`、`nc` 一起判断。