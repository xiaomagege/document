# 37｜服务、软件与防火墙｜服务起不来时，systemctl 应该先看什么

服务起不来时，最容易犯的错是直接反复 `restart`。

真正高效的做法通常是先回答几个问题：

- 服务现在到底是 `running`、`inactive` 还是 `failed`
- 最近失败日志里写了什么
- 是配置问题、端口问题、依赖问题，还是 unit 文件问题

如果只想先记 3 条：

- 服务有问题先看 `systemctl status`
- 只会 `restart` 不够，记得配 `journalctl -u`
- 改了 unit 文件后才需要 `daemon-reload`

## 一、服务起不来时，先看哪几层

先不要急着重启，先分层判断：

1. 服务状态是什么
2. 最近日志里报了什么
3. 是服务配置坏了，还是启动环境坏了
4. 是业务配置改了，还是 unit 文件改了

第一步通常就是：

```bash
systemctl status nginx
```

## 二、先记住这几条命令

```bash
systemctl status nginx
systemctl restart nginx
systemctl reload nginx
systemctl --failed
journalctl -u nginx -n 50 --no-pager
systemctl daemon-reload
```

它们分别适合：

- `status`：看当前状态和最近错误
- `restart`：重启服务
- `reload`：重新加载配置
- `--failed`：看当前失败的 unit
- `journalctl -u`：补完整日志
- `daemon-reload`：让 systemd 重新读取 unit 文件

## 三、推荐的排查顺序

### 1. 先看状态

```bash
systemctl status nginx
```

这一步通常已经能回答很多问题：

- 服务有没有启动起来
- 失败发生在什么时候
- 主进程 PID 是多少
- 最近一两条日志在报什么

### 2. 再看完整日志

```bash
journalctl -u nginx -n 50 --no-pager
```

如果 `status` 里的几行不够，日志通常能把问题说得更清楚。

常见方向包括：

- 配置文件语法错
- 端口被占用
- 文件路径不存在
- 权限不对
- 依赖服务没起来

### 3. 再判断该 `reload` 还是 `restart`

- 改的是业务配置，而且服务支持热加载：先 `reload`
- 配置需要完整重启生效：`restart`

别把所有变更都粗暴地归到 `restart` 上。

### 4. 只有改了 unit 文件，才 `daemon-reload`

```bash
systemctl daemon-reload
```

这一步不是每次都要做。

只有当你改的是：

- `/etc/systemd/system/*.service`
- 其他 unit 定义文件

才需要让 systemd 重新读一遍。

## 四、最常见的 4 个问题

### 1. 服务启动失败

先看：

```bash
systemctl status 服务名
journalctl -u 服务名 -n 100 --no-pager
```

### 2. `enable` 了但服务没跑

`enable` 只是开机自启，不会立刻启动。

要立即运行，用：

```bash
systemctl start 服务名
```

或者：

```bash
systemctl enable --now 服务名
```

### 3. 服务显示 `masked`

这说明它被显式屏蔽了。

先解除：

```bash
systemctl unmask 服务名
```

再决定是否 `start` 或 `enable`。

### 4. 改完 unit 文件还是没生效

这通常就是漏了：

```bash
systemctl daemon-reload
```

## 五、`status`、`journalctl`、`--failed` 怎么分工

- `systemctl status`：看单个服务的第一眼状态
- `journalctl -u`：看更完整的日志上下文
- `systemctl --failed`：看系统里哪些 unit 整体失败了

这三条配起来，基本已经覆盖大多数“服务起不来”的第一轮排查。

## 六、最常见的误区

### 1. 反复 `restart`，不看日志

这通常只是把问题重复一遍，不会让问题自己消失。

### 2. 改了普通业务配置却先 `daemon-reload`

这一步只和 unit 文件有关，不是所有配置变更都需要它。

### 3. 混淆 `reload` 和 `restart`

能热加载的服务，`reload` 往往影响更小。

## 七、结论

服务起不来时，先 `status`，再 `journalctl`，最后再决定是改配置、改权限、改 unit，还是重启。

这样比上来就 `restart` 一顿更稳，也更容易把问题一次定位对。
