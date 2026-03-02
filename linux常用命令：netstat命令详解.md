# linux常用命令：netstat命令详解

在 Linux 运维排障中，`netstat` 是一个经典网络工具。它可以查看端口监听状态、网络连接、路由表和网卡统计信息，常用于排查“端口占用”“服务连不上”“连接异常过多”等问题。

---

## 一、`netstat` 是什么？

`netstat`（network statistics）用于查看系统网络状态，包括：

- 当前网络连接
- 监听端口
- 路由表
- 各协议统计信息
- 网卡收发统计

注意：很多新系统推荐用 `ss` 替代 `netstat`，但 `netstat` 仍非常常见，尤其在老环境和面试场景中。

---

## 二、常见语法

```bash
netstat [选项]
```

常用参数速查：

- `-a`：显示所有连接和监听端口
- `-l`：仅显示监听端口（Listening）
- `-t`：仅显示 TCP 连接
- `-u`：仅显示 UDP 连接
- `-n`：以数字形式显示地址和端口（不做 DNS 解析）
- `-p`：显示 PID/程序名（通常需要 root 权限）
- `-r`：显示路由表
- `-i`：显示网卡接口统计
- `-s`：显示各协议统计信息（TCP/UDP/ICMP 等）
- `-c`：持续输出，每秒刷新一次

常见组合：

- `-an`：查看全部连接（数字显示）
- `-lntp`：查看 TCP 监听端口及对应进程
- `-rn`：查看路由表（数字显示）

---

## 三、最常用命令组合

### 1. 查看所有连接（TCP/UDP）
```bash
netstat -an
```

- `-a`：显示所有连接和监听端口
- `-n`：地址和端口用数字显示（更快、更清晰）

### 2. 查看监听端口
```bash
netstat -lnt
```

- `-l`：仅监听（listen）
- `-t`：TCP
- `-n`：数字格式

### 3. 查看监听端口并显示进程
```bash
sudo netstat -lntp
```

- `-p`：显示 PID/程序名（通常需要 root 权限）

### 4. 查看 UDP 监听
```bash
sudo netstat -lnup
```

### 5. 查看路由表
```bash
netstat -rn
```

- `-r`：路由信息
- `-n`：数字显示（不做 DNS 反查）

---

## 四、输出字段速查（连接类）

常见输出列：

- `Proto`：协议（tcp/udp）
- `Recv-Q`：接收队列未处理字节数
- `Send-Q`：发送队列未确认字节数
- `Local Address`：本地地址:端口
- `Foreign Address`：远端地址:端口
- `State`：连接状态（TCP 才有）
- `PID/Program name`：进程 ID/进程名（带 `-p` 时）

---

## 五、TCP 状态常见值

- `LISTEN`：监听中，等待连接
- `ESTABLISHED`：连接已建立
- `TIME_WAIT`：主动关闭后等待
- `CLOSE_WAIT`：被动关闭等待本地关闭
- `SYN_SENT`：已发起连接，等待响应
- `SYN_RECV`：收到请求，等待确认

排障经验：
- `TIME_WAIT` 大量出现：可能短连接过多
- `CLOSE_WAIT` 堆积：常见于应用未正确关闭 socket
- `SYN_RECV` 异常多：可能连接洪泛或服务处理慢

---

## 六、实战示例

### 1. 查 80 端口是否被占用
```bash
sudo netstat -lntp | grep ':80'
```

### 2. 查某进程相关连接
```bash
sudo netstat -antp | grep nginx
```

### 3. 统计各 TCP 状态数量
```bash
netstat -ant | awk '{print $NF}' | sort | uniq -c | sort -nr
```

### 4. 只看已建立连接
```bash
netstat -ant | grep ESTABLISHED
```

### 5. 排查可疑高频远端 IP
```bash
netstat -ant | awk '/ESTABLISHED/{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr | head
```

---

## 七、`netstat` 与 `ss` 的区别

- `netstat`：经典、通用、很多环境默认有。
- `ss`：更快、信息更丰富，适合高并发场景。

常见替代：
```bash
# netstat -lntp
ss -lntp
```

---

## 八、常见问题与建议

1. 看不到进程名怎么办？  
使用 `sudo` 执行并加 `-p` 参数。

2. 结果里很多域名解析影响查看？  
加 `-n`，直接显示 IP/端口。

3. 端口明明监听了却访问失败？  
继续检查防火墙（`iptables`/`firewalld`）、安全组、服务绑定地址（`0.0.0.0` 或 `127.0.0.1`）。

---

## 九、总结

建议优先掌握这 3 条：

```bash
sudo netstat -lntp
netstat -ant
netstat -rn
```

这三条基本可以覆盖端口监听、连接状态、路由路径三类核心网络排障场景。

