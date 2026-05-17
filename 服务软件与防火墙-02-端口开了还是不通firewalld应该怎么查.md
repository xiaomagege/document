# 服务软件与防火墙02：端口开了还是不通？firewalld 应该怎么查

很多“服务明明启动了，端口也在监听，为什么外部还是访问不到”的问题，最后都会绕到防火墙。

而 `firewalld` 最容易让人混乱的地方，不是命令多，而是 zone、运行时配置、永久配置这几层经常混在一起。结果就是你以为已经放开了，其实只改了一半。

如果只想先记 3 条：

- 先看 `firewall-cmd --state` 和 `--list-all`
- 改正式规则时，优先用 `--permanent` 再 `--reload`
- 端口不通时，不要只看防火墙，还要同时看服务是否监听

## 一、什么情况下先用 `firewalld`

`firewalld` 最适合这些场景：

- 服务端口已经监听，但外部仍然不通
- 想确认某个 zone 到底放了哪些端口和服务
- 想安全地开放某个新端口或服务

最常用的第一组命令通常是：

```bash
firewall-cmd --state
firewall-cmd --get-active-zones
firewall-cmd --list-all
```

这三条基本能先把状态、zone 和当前规则看清。

## 二、先记住这几条命令

```bash
firewall-cmd --state
firewall-cmd --get-active-zones
firewall-cmd --list-all
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```

它们分别适合：

- 看服务是否运行
- 看活动 zone
- 看当前 zone 的详细规则
- 按端口放行
- 按服务名放行
- 让永久规则生效

## 三、端口不通时，推荐先怎么查

### 1. 先确认防火墙是不是在跑

```bash
firewall-cmd --state
```

如果返回 `running`，说明 `firewalld` 正在起作用。

### 2. 再看当前网卡落在哪个 zone

```bash
firewall-cmd --get-active-zones
```

很多误判都发生在这里。你以为自己改的是当前生效的区域，结果规则其实加到了另一个 zone。

### 3. 再看 zone 里到底放了什么

```bash
firewall-cmd --list-all
firewall-cmd --zone=public --list-all
```

这一步重点看：

- 已开放的 services
- 已开放的 ports
- 来源限制是不是卡住了

### 4. 不要忘了同时看服务监听

```bash
ss -lntp | grep 8080
```

如果端口本身都没监听，那继续改防火墙就已经偏题了。

## 四、运行时配置和永久配置怎么分

### 1. 运行时配置

```bash
firewall-cmd --add-port=8080/tcp
```

它会立刻生效，但系统重启后可能丢。

### 2. 永久配置

```bash
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload
```

这是生产环境里更值得优先采用的写法。

如果你只改了 `--permanent` 但没 `--reload`，规则也还不会立刻进到当前运行状态里。

## 五、最常见的几种处理方式

### 1. 放行 HTTP/HTTPS

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
```

### 2. 放行自定义端口

```bash
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload
```

### 3. 关闭不再需要的端口

```bash
firewall-cmd --permanent --remove-port=3306/tcp
firewall-cmd --reload
```

### 4. 明确看某个 zone

```bash
firewall-cmd --zone=public --list-all
```

多网卡、多区域时，这条很有必要。

## 六、最常见的误区

### 1. 只开放了运行时配置

机器一重启，规则就没了。

### 2. 只改了永久配置，但忘了 `reload`

你会以为规则已经加好了，其实当前运行态根本没用上。

### 3. 把所有“不通”都归到防火墙

端口不通还可能是：

- 服务没监听
- 服务只监听在 `127.0.0.1`
- 云安全组没放行
- 上层路由或 ACL 限制

## 七、结论

`firewalld` 排查的关键，不是记住更多参数，而是先把三件事分清：

- 当前防火墙有没有在跑
- 当前生效的是哪个 zone
- 你改的是运行时规则，还是永久规则

把这三层看清，再结合 `ss` 看端口监听，很多“端口开了还是不通”的问题就不会查偏。

