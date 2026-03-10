# linux常用命令：firewalld命令详解

在 Linux 运维中，防火墙配置是服务器安全和服务可访问性的基础能力。`firewalld` 是很多主流发行版中常见的动态防火墙管理工具，适合用来开放端口、允许服务访问、限制来源以及管理不同网络区域策略。

---

## 一、`firewalld` 是什么？

`firewalld` 是 Linux 下常见的防火墙管理服务，底层通常基于 `nftables` 或 `iptables`，通过更易用的方式对网络访问策略进行统一管理。

它的核心特点包括：

- 支持动态更新规则，不必每次都重启防火墙
- 支持按区域（zone）管理不同网络环境
- 支持按服务名、端口、协议进行放行
- 支持运行时配置与永久配置分离

日常管理时，通常配合 `firewall-cmd` 命令使用。

---

## 二、语法与参数

```bash
firewall-cmd [选项]
```

`firewall-cmd` 的日常使用，核心就是围绕“查看状态、查看规则、开放端口、开放服务、永久生效、重载配置”这几类操作展开。

常见参数和用法如下：

- `firewall-cmd --state`：查看防火墙是否运行
- `firewall-cmd --get-active-zones`：查看当前活动区域
- `firewall-cmd --list-all`：查看当前区域详细规则
- `firewall-cmd --list-ports`：查看当前区域已开放端口
- `firewall-cmd --get-default-zone`：查看默认 zone
- `firewall-cmd --set-default-zone=public`：设置默认 zone
- `firewall-cmd --add-port=80/tcp`：临时开放 80 端口
- `firewall-cmd --remove-port=80/tcp`：关闭指定端口
- `firewall-cmd --add-service=http`：按服务名放行
- `firewall-cmd --remove-service=http`：取消放行服务
- `firewall-cmd --add-source=192.168.1.0/24`：允许指定来源
- `firewall-cmd --permanent --add-service=http`：永久放行 HTTP 服务
- `firewall-cmd --reload`：重载规则使永久配置生效

示例：

```bash
firewall-cmd --state
firewall-cmd --list-all
firewall-cmd --list-ports
firewall-cmd --permanent --add-port=3306/tcp
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
```

---

## 三、最常用命令组合

### 1. 查看防火墙状态

```bash
firewall-cmd --state
```

如果返回 `running`，说明 `firewalld` 服务正在运行。

### 2. 查看当前活动区域

```bash
firewall-cmd --get-active-zones
```

可以看到当前网卡接口绑定到了哪个 zone，比如 `public`、`internal` 等。

### 3. 查看当前区域规则

```bash
firewall-cmd --list-all
```

这条命令常用于快速确认已经放行了哪些服务、端口和来源。

### 4. 查看当前已开放端口

```bash
firewall-cmd --list-ports
```

如果只想快速看当前 zone 已经放行了哪些端口，这条命令比 `--list-all` 更直接。

### 5. 开放指定端口

```bash
firewall-cmd --add-port=8080/tcp
```

这类操作默认只对当前运行时配置生效，系统重启后可能失效。

### 6. 永久开放端口并重载

```bash
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload
```

这是生产环境中更常见的写法。

---

## 四、实战示例

### 1. 开放 Web 服务端口

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
```

相比直接开放端口，按服务名放行更直观，也更方便维护。

### 2. 放行应用自定义端口

```bash
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload
```

适用于 Java、Go、Node.js 等应用监听的自定义端口。

### 3. 删除不再使用的端口

```bash
firewall-cmd --permanent --remove-port=3306/tcp
firewall-cmd --reload
```

服务下线或迁移后，建议及时关闭不再需要的端口。

### 4. 只允许指定来源访问

```bash
firewall-cmd --permanent --add-source=192.168.1.0/24
firewall-cmd --reload
```

适合内网管理、数据库白名单等场景。

### 5. 查看某个 zone 的规则

```bash
firewall-cmd --zone=public --list-all
```

当机器存在多个 zone 时，明确指定 zone 更稳妥。

---

## 五、运行时配置与永久配置的区别

`firewalld` 有两套配置：

- 运行时配置：立即生效，但重启后可能丢失
- 永久配置：写入配置文件，重启后仍然保留

常见原则：

- 临时测试时，可直接执行 `--add-port`
- 正式上线时，建议加 `--permanent`
- 修改永久配置后，记得执行 `firewall-cmd --reload`

例如：

```bash
firewall-cmd --add-port=9090/tcp
firewall-cmd --permanent --add-port=9090/tcp
firewall-cmd --reload
```

---

## 六、`firewalld` 与 `iptables` 的区别

- `firewalld`：更偏向高层管理，支持 zone 和动态更新
- `iptables`：更底层、更细粒度，但配置和维护复杂度更高

在现代 Linux 发行版中，日常服务器运维通常优先使用 `firewalld`；如果需要非常复杂的规则控制，才会进一步接触底层规则工具。

---

## 七、常见问题与建议

1. 为什么明明开放了端口，外部还是访问不到？
除了防火墙规则，还要检查服务是否真的监听该端口、监听地址是否正确，以及云平台安全组是否已放行。

2. 为什么加了规则，重启后失效？
大概率是只修改了运行时配置，没有加 `--permanent`。

3. 开放端口和开放服务有什么区别？
开放服务更语义化，例如 `http`、`https`、`ssh`；开放端口更直接，适合自定义应用。

4. 修改完永久配置后为什么没生效？
因为还没有执行 `firewall-cmd --reload`。

5. 生产环境怎么更稳妥？
先 `--list-all` 看清当前规则，再逐步添加或删除，避免误封 SSH 等关键访问通道。

---

## 八、总结

`firewalld` 是 Linux 服务器防火墙管理的常用工具。建议优先熟练掌握以下几条：

```bash
firewall-cmd --state
firewall-cmd --get-active-zones
firewall-cmd --list-all
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```

掌握这些基础命令后，大多数“查看防火墙状态、开放端口、放行服务、持久化规则”的场景都能顺利处理。
