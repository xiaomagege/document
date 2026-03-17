# linux常用命令：iptables命令详解

在 Linux 运维中，`iptables` 是最经典的防火墙管理工具之一。虽然很多新系统默认更常使用 `firewalld` 或 `nftables`，但在大量生产环境、老系统和排障场景中，`iptables` 依然非常常见。

---

## 一、`iptables` 是什么？

`iptables` 是 Linux 下用于配置内核网络包过滤规则的工具，常用于实现防火墙、端口控制、来源限制和网络转发策略。

它的核心能力包括：

- 控制哪些流量可以进入服务器
- 控制哪些流量可以离开服务器
- 按来源 IP、端口、协议进行过滤
- 配合 NAT 做端口转发和地址转换

因此，`iptables` 是 Linux 网络安全和访问控制的重要基础工具。

---

## 二、常见使用语法

```bash
iptables [选项] 链 规则
```

常见示例：

- `iptables -L -n`：查看当前规则
- `iptables -L -n --line-numbers`：按编号查看规则
- `iptables -A INPUT -p tcp --dport 22 -j ACCEPT`：允许 TCP 22 端口访问
- `iptables -A INPUT -s 192.168.1.10 -j ACCEPT`：允许指定来源 IP
- `iptables -D INPUT 1`：删除 INPUT 链第 1 条规则

---

## 三、最常用命令组合

### 1. 查看当前防火墙规则

```bash
iptables -L -n
```

其中：

- `-L`：列出规则
- `-n`：以数字形式显示 IP 和端口，避免反向解析更快

### 2. 带编号查看规则

```bash
iptables -L -n --line-numbers
```

删除规则时非常常用，因为可以直接按编号删除。

### 3. 开放指定端口

```bash
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

表示在 `INPUT` 链追加一条规则，允许外部访问本机 80 端口。

### 4. 放行已建立连接

```bash
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

这是很多服务器规则中的基础项，用来允许已经建立或相关联的返回流量。

### 5. 拒绝指定端口访问

```bash
iptables -A INPUT -p tcp --dport 3306 -j DROP
```

适合阻止外部直接访问不应暴露的端口。

---

## 四、核心概念：表与链

理解 `iptables`，最关键的是先理解“表”和“链”。

常见表：

- `filter`：默认表，用于包过滤
- `nat`：用于地址转换
- `mangle`：用于修改数据包

常见链：

- `INPUT`：进入本机的数据包
- `OUTPUT`：本机发出的数据包
- `FORWARD`：经过本机转发的数据包

日常服务器防火墙配置中，最常操作的通常是 `filter` 表里的 `INPUT` 链。

---

## 五、重点参数说明

- `-L`：列出规则
- `-A`：追加规则
- `-I`：插入规则
- `-D`：删除规则
- `-F`：清空规则
- `-P`：设置默认策略
- `-p`：指定协议，如 `tcp`、`udp`
- `-s`：指定源地址
- `--dport`：指定目标端口
- `-j`：指定动作，如 `ACCEPT`、`DROP`、`REJECT`
- `-t`：指定表，如 `nat`
- `-n`：数字方式显示
- `--line-numbers`：显示规则编号

示例：

```bash
iptables -L -n --line-numbers
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -I INPUT 1 -p tcp --dport 22 -j ACCEPT
iptables -D INPUT 3
iptables -P INPUT DROP
```

---

## 六、实战示例

### 1. 允许 SSH 登录

```bash
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

这是远程管理服务器时最关键的一条规则之一，实际操作时要优先确保 SSH 不会被误封。

### 2. 允许 Web 服务访问

```bash
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

适用于开放 HTTP 和 HTTPS 服务。

### 3. 只允许指定 IP 访问数据库

```bash
iptables -A INPUT -p tcp -s 192.168.1.100 --dport 3306 -j ACCEPT
iptables -A INPUT -p tcp --dport 3306 -j DROP
```

先允许指定来源，再拒绝其他来源访问数据库端口。

### 4. 按编号删除规则

```bash
iptables -L -n --line-numbers
iptables -D INPUT 2
```

先看编号，再删除指定规则，这是最稳妥的常见做法。

### 5. 查看 NAT 规则

```bash
iptables -t nat -L -n
```

当机器做了端口映射、地址转换或网关转发时，这条命令很有用。

---

## 七、查看开放端口相关规则

很多人会问：`iptables` 怎么查看已经开放了哪些端口？

常见方法有两种：

### 1. 查看所有 INPUT 链规则

```bash
iptables -L INPUT -n --line-numbers
```

可以直接看到哪些端口对应了 `ACCEPT`、`DROP` 或 `REJECT` 规则。

### 2. 只筛选端口相关规则

```bash
iptables -L INPUT -n | grep dpt:
```

这样可以更快聚焦到包含目标端口的规则。

如果想进一步看某个具体端口，比如 80 端口，可用：

```bash
iptables -L INPUT -n | grep 'dpt:80'
```

---

## 八、默认策略与规则顺序

`iptables` 的规则是按顺序匹配的，匹配到第一条符合条件的规则后就会执行对应动作。

这意味着两点非常重要：

- 规则顺序会直接影响最终结果
- 默认策略决定未匹配流量的处理方式

例如：

```bash
iptables -P INPUT DROP
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

表示默认拒绝所有进入流量，只放行 SSH 和 HTTP。

如果顺序写错，可能会导致本来应该放行的流量先被拦截。

---

## 九、保存规则与开机生效

`iptables` 命令直接添加的规则，很多系统在重启后不会自动保留，因此通常还需要保存规则。

不同发行版做法略有差异，常见方式包括：

- CentOS 6/部分旧系统：`service iptables save`
- 较新系统：使用 `iptables-save` 和 `iptables-restore`
- Debian/Ubuntu：常借助 `iptables-persistent`

常见示例：

```bash
iptables-save
iptables-save > /etc/sysconfig/iptables
```

具体保存位置和服务管理方式需要根据发行版确认。

---

## 十、`iptables` 与 `firewalld` 的区别

- `iptables`：更底层，规则控制更直接
- `firewalld`：更高层，支持 zone 和动态管理

简单理解：

- 想直接精确控制规则细节：更常接触 `iptables`
- 想更方便地管理服务、端口和区域：更常使用 `firewalld`

很多系统里，`firewalld` 底层仍可能调用 `iptables` 或相关内核能力。

---

## 十一、常见问题与建议

1. 为什么加了规则还是访问不了？
除了 `iptables` 规则，还要检查服务是否监听端口、是否绑定正确地址，以及云安全组或外层网络策略是否放行。

2. 为什么删规则前建议先看编号？
因为手工重输整条规则容易出错，而按编号删除更直观、更稳妥。

3. 为什么要特别小心 SSH 规则？
因为如果误删或误拦截 22 端口，可能会直接导致远程连接中断。

4. `DROP` 和 `REJECT` 有什么区别？
`DROP` 是直接丢弃，客户端通常表现为超时；`REJECT` 会明确返回拒绝信息。

5. 生产环境怎么操作更安全？
先 `iptables -L -n --line-numbers` 看清现状，再逐条添加或调整规则，避免一次性大改。

---

## 十二、总结

`iptables` 是 Linux 服务器网络访问控制的经典工具。建议优先熟练掌握以下几条：

```bash
iptables -L -n
iptables -L -n --line-numbers
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -D INPUT 1
iptables -t nat -L -n
```

掌握这些基础操作后，大多数“查看规则、开放端口、限制来源、删除规则、查看 NAT”的场景都能完成。
