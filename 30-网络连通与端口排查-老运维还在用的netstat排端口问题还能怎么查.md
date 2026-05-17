# 30｜网络连通与端口排查｜老运维还在用的 netstat，排端口问题还能怎么查

`netstat` 现在确实不算新工具了，但它还远没有退出一线。

很多老环境里它依然随手可用，而且在“先看监听端口、看连接状态、看路由”这几件事上，它依然能很快给你第一眼判断。真正要注意的不是会不会用，而是知道它和 `ss` 的边界。

如果只想先记 3 条：

- 老环境排端口时，`netstat -lntp` 依然很好用
- 看连接状态时，`netstat -ant` 很直观
- 新环境优先 `ss`，但看到 `netstat` 别慌

## 一、什么情况下先用 `netstat`

`netstat` 最适合这些场景：

- 老机器上先看端口有没有监听
- 先看连接状态堆在哪
- 先看本机路由表

最常见的起手式就是：

```bash
netstat -lntp
netstat -ant
netstat -rn
```

## 二、先记住这几条命令

```bash
sudo netstat -lntp
netstat -ant
netstat -ant | grep ESTABLISHED
netstat -rn
netstat -ant | awk '{print $NF}' | sort | uniq -c | sort -nr
```

它们分别适合：

- 看监听端口和进程
- 看所有 TCP 连接
- 只看已建立连接
- 看路由
- 粗看 TCP 状态分布

## 三、排端口问题时，它最适合帮你看什么

### 1. 端口有没有监听

```bash
sudo netstat -lntp | grep ':80'
```

这适合快速回答：服务到底有没有把端口真正拉起来。

### 2. 连接状态堆在哪

```bash
netstat -ant
```

这一步很适合观察：

- `LISTEN`
- `ESTABLISHED`
- `TIME_WAIT`
- `CLOSE_WAIT`

如果某个状态异常多，通常已经是线索了。

### 3. 路由是不是走歪了

```bash
netstat -rn
```

虽然现在很多人更习惯 `ip route`，但 `netstat -rn` 在老环境里依然够直接。

## 四、它和 `ss` 怎么分工

### 1. `netstat`

优点是经典、常见、很多老系统里现成可用。

### 2. `ss`

通常更快、信息更丰富，也更适合新系统。

所以最稳的理解是：

- 老环境先用 `netstat` 没问题
- 新环境优先 `ss`

## 五、最常见的误区

### 1. 只看监听，不看状态

端口在监听不代表连接层一定没问题，状态分布同样重要。

### 2. 看到 `TIME_WAIT` 多就立刻下结论

它可能只是短连接很多，不一定就是故障本身。

### 3. 把 `netstat` 当唯一答案

真正排障时，通常还要继续配：

```bash
ss -lntp
nc -vz host 端口
curl -v URL
```

## 六、结论

`netstat` 现在更像一把“老但顺手”的工具。

排端口和连接状态时，它依然能给你很快的第一眼判断；只是到了新系统和高并发场景，主力已经更多交给 `ss` 了。
