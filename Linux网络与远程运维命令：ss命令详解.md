# Linux网络与远程运维命令：ss命令详解

在 Linux 网络排查中，经常需要查看端口是否监听、连接是否建立、哪个进程占用了端口。过去很多人习惯使用 `netstat`，现在更推荐使用 `ss`。

`ss` 是查看 socket 连接状态的工具，速度快、信息丰富，是排查服务监听、端口占用、TCP 连接状态的常用命令。

---

## 一、`ss` 是什么？

`ss` 是 socket statistics 的缩写，用于显示系统 socket 连接信息。

它主要用于：

- 查看监听端口
- 查看 TCP/UDP 连接
- 查看端口对应的进程
- 排查服务是否启动并监听
- 查看连接状态，例如 `LISTEN`、`ESTAB`、`TIME-WAIT`
- 替代传统 `netstat`

常见排查场景：

- 服务启动了但端口没有监听
- 端口被其他进程占用
- 某个端口连接数异常高
- TCP 连接大量处于 `TIME-WAIT`

---

## 二、常见使用语法与重点参数

```bash
ss [选项] [过滤条件]
```

常用参数说明：

- `-t`：显示 TCP socket
- `-u`：显示 UDP socket
- `-l`：只显示监听状态
- `-n`：不解析服务名和主机名，直接显示数字
- `-p`：显示进程信息
- `-a`：显示所有 socket
- `-r`：解析主机名
- `-s`：显示统计摘要
- `-4`：只显示 IPv4
- `-6`：只显示 IPv6

常见示例：

```bash
ss -lnt
ss -lntp
ss -ant
ss -tunap
ss -s
```

---

## 三、常用命令与使用场景

### 1. 查看 TCP 监听端口

```bash
ss -lnt
```

含义：

- `-l`：监听状态
- `-n`：数字显示端口
- `-t`：TCP

这是查看服务是否监听端口最常用的命令。

### 2. 查看监听端口和进程

```bash
ss -lntp
```

会显示端口对应的进程信息。

例如排查 80 端口被谁占用：

```bash
ss -lntp | grep ':80'
```

如果没有权限看到进程信息，可以使用 sudo：

```bash
sudo ss -lntp
```

### 3. 查看 UDP 监听端口

```bash
ss -lnu
```

适合排查 DNS、NTP、syslog 等 UDP 服务。

如果要显示进程：

```bash
sudo ss -lnup
```

### 4. 查看所有 TCP 连接

```bash
ss -ant
```

会显示所有 TCP 连接，包括监听、已建立、等待关闭等状态。

常见状态包括：

- `LISTEN`：正在监听
- `ESTAB`：连接已建立
- `TIME-WAIT`：等待连接彻底关闭
- `SYN-SENT`：正在发起连接
- `SYN-RECV`：收到连接请求

### 5. 查看已建立连接

```bash
ss -ant state established
```

可以只查看已经建立的 TCP 连接。

查看某个端口的已建立连接：

```bash
ss -ant state established '( sport = :80 or dport = :80 )'
```

### 6. 查看连接统计摘要

```bash
ss -s
```

会显示当前 socket 的总体统计信息。

当连接数异常时，可以先用 `ss -s` 快速观察整体状态。

### 7. 统计某个端口连接数

```bash
ss -ant | grep ':443' | wc -l
```

如果只统计已建立连接：

```bash
ss -ant state established | grep ':443' | wc -l
```

### 8. 按状态统计 TCP 连接

```bash
ss -ant | awk 'NR > 1 {count[$1]++} END {for (s in count) print s, count[s]}'
```

可以快速查看当前 TCP 连接状态分布。

### 9. 只查看 IPv4 或 IPv6

```bash
ss -4 -lnt
ss -6 -lnt
```

有些服务只监听 IPv6 或只监听 IPv4，排查时可以分别确认。

---

## 四、常见问题与建议

### 1. 服务启动了，为什么访问不了？

先看服务是否监听端口：

```bash
ss -lntp | grep ':8080'
```

如果没有监听，需要检查服务启动日志。

如果只监听 `127.0.0.1:8080`，外部机器无法直接访问，需要检查服务绑定地址。

### 2. `0.0.0.0:80` 和 `127.0.0.1:80` 有什么区别？

- `0.0.0.0:80`：监听所有 IPv4 地址
- `127.0.0.1:80`：只监听本机回环地址

服务对外访问异常时，这个差异很关键。

### 3. 为什么看不到进程信息？

普通用户可能无权查看其他用户进程的 socket 信息。

使用：

```bash
sudo ss -lntp
```

### 4. `ss` 和 `netstat` 怎么选？

现在更推荐 `ss`。它通常更快，信息也更完整。

常见替代关系：

```bash
netstat -lntp  ->  ss -lntp
netstat -ant   ->  ss -ant
netstat -s     ->  ss -s
```

---

## 五、常用命令速查

```bash
ss -lnt
sudo ss -lntp
ss -lnu
sudo ss -lnup
ss -ant
ss -ant state established
ss -ant state established '( sport = :80 or dport = :80 )'
ss -s
ss -ant | grep ':443' | wc -l
ss -ant state established | grep ':443' | wc -l
ss -ant | awk 'NR > 1 {count[$1]++} END {for (s in count) print s, count[s]}'
ss -4 -lnt
ss -6 -lnt
```

---

## 六、总结

`ss` 是 Linux 中查看 socket 和端口状态的核心命令。建议优先掌握：

```bash
ss -lnt
sudo ss -lntp
ss -ant
ss -s
```

排查网络服务时，先用 `ss` 确认端口是否监听、监听地址是否正确、端口是否被其他进程占用，再继续检查防火墙、路由、服务日志等问题，会更有条理。