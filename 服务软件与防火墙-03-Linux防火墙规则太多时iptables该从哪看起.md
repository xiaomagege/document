# 服务软件与防火墙03：Linux 防火墙规则太多时，iptables 该从哪看起

`iptables` 最让人头大的地方，通常不是“不会写规则”，而是线上一堆旧规则摆在那里，你根本不知道该先看哪条。

这也是为什么排查 `iptables` 时，起手不该是急着加规则，而是先把默认策略、链、顺序和目标端口相关规则看清。否则越改越乱是常态。

如果只想先记 3 条：

- 排查时先 `iptables -L -n --line-numbers`
- 先看 `INPUT` 链，再看目标端口对应规则
- 别在没看清规则前随手 `-F` 或大面积改默认策略

## 一、什么情况下先用 `iptables`

`iptables` 最适合这些场景：

- 老系统或老环境里仍然主要靠 `iptables` 管规则
- 端口明明监听了，但访问总是被拦
- 你怀疑不是服务问题，而是包过滤顺序有问题

最常用的第一条命令通常就是：

```bash
iptables -L -n --line-numbers
```

它能先把最关键的三件事摆出来：

- 规则顺序
- 动作是 `ACCEPT`、`DROP` 还是 `REJECT`
- 哪条规则最值得先看

## 二、先记住这几条命令

```bash
iptables -L -n --line-numbers
iptables -L INPUT -n --line-numbers
iptables -L INPUT -n | grep dpt:
iptables -t nat -L -n
iptables -D INPUT 3
```

它们分别适合：

- 看整体规则和编号
- 聚焦 `INPUT` 链
- 快速筛目标端口相关规则
- 看 NAT 规则
- 按编号删除单条规则

## 三、排查时先看哪几层

### 1. 先看默认策略

如果默认策略就是 `DROP`，那没有被显式放行的流量本来就会被拦。

```bash
iptables -L -n --line-numbers
```

先看 `Chain INPUT (policy ...)` 这一行很重要。

### 2. 再看 `INPUT` 链顺序

`iptables` 是按顺序匹配的，第一条命中的规则就会决定命运。

这意味着：

- 后面的放行规则，可能永远轮不到
- 前面一条宽泛的 `DROP`，就足以把后面都盖掉

### 3. 再看目标端口规则

```bash
iptables -L INPUT -n | grep 'dpt:80'
iptables -L INPUT -n | grep 'dpt:443'
```

如果你在查具体端口，直接筛端口相关规则会快很多。

### 4. 如果怀疑做了转发，再看 NAT

```bash
iptables -t nat -L -n
```

端口映射、转发、SNAT/DNAT 相关问题，不看 NAT 表就容易漏。

## 四、最常见的几种处理方式

### 1. 先按编号定位，再删除单条

```bash
iptables -L INPUT -n --line-numbers
iptables -D INPUT 3
```

这比凭感觉重写整段规则稳得多。

### 2. 放行 SSH 和 Web 端口

```bash
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

但真正在线上操作时，顺序依然要结合当前规则环境看。

### 3. 只允许指定来源访问数据库

```bash
iptables -A INPUT -p tcp -s 192.168.1.100 --dport 3306 -j ACCEPT
iptables -A INPUT -p tcp --dport 3306 -j DROP
```

这个例子本身就说明了顺序的重要性。

## 五、`iptables` 最容易让人踩坑的地方

### 1. 规则顺序

同样的几条规则，顺序不同，结果可能完全不同。

### 2. 默认策略

你没显式允许的流量，最终能不能过，很多时候就取决于默认策略是 `ACCEPT` 还是 `DROP`。

### 3. 一着急就清空规则

像下面这类动作：

```bash
iptables -F
iptables -P INPUT DROP
```

在线上风险都很大，尤其是远程操作时很容易把自己锁在门外。

## 六、推荐的联动排查顺序

如果你怀疑问题在防火墙，不要只盯 `iptables` 本身。更稳的顺序通常是：

```bash
ss -lntp | grep 端口
nc -vz host 端口
iptables -L -n --line-numbers
iptables -t nat -L -n
```

先确认端口本身有监听，再确认流量是不是被规则拦了。

## 七、结论

`iptables` 排查最关键的不是会不会写规则，而是会不会先看清现状。

先看默认策略，再看 `INPUT` 链顺序，再看目标端口相关规则。把这条顺序守住，面对一大堆旧规则时就没那么容易慌。

