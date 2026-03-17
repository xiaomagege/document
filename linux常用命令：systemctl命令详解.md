# linux常用命令：systemctl命令详解

在使用 `systemd` 作为初始化系统的 Linux 发行版中，`systemctl` 是最核心的服务管理命令之一。无论是启动服务、查看状态、设置开机自启，还是排查服务启动失败问题，`systemctl` 都是高频使用工具。

---

## 一、`systemctl` 是什么？

`systemctl` 是 `systemd` 的管理命令，用于控制系统中的各种 unit。

最常见的 unit 类型包括：

- `service`：服务
- `socket`：套接字
- `target`：运行目标，类似传统运行级别
- `mount`：挂载点
- `timer`：定时任务

在日常运维中，最常打交道的是 `service`，例如 `nginx.service`、`docker.service`、`sshd.service`。

---

## 二、常见使用语法

```bash
systemctl [命令] [服务名]
```

常见示例：

- `systemctl status nginx`：查看 `nginx` 服务状态
- `systemctl start nginx`：启动 `nginx`
- `systemctl stop nginx`：停止 `nginx`
- `systemctl restart nginx`：重启 `nginx`
- `systemctl enable nginx`：设置开机自启

说明：

- 服务名通常可写成 `nginx` 或 `nginx.service`
- 部分操作需要 `root` 权限，常配合 `sudo` 使用

---

## 三、最常用命令组合

### 1. 查看服务状态

```bash
systemctl status nginx
```

这是最常用的一条命令，可以快速看到：

- 服务是否正在运行
- 最近启动日志
- 主进程 PID
- 启动失败原因

### 2. 启动服务

```bash
systemctl start nginx
```

适用于服务当前未启动的场景。

### 3. 停止服务

```bash
systemctl stop nginx
```

适用于需要临时停服务、发布变更或排障时使用。

### 4. 重启服务

```bash
systemctl restart nginx
```

修改配置后经常需要重启服务使配置生效。

### 5. 重新加载配置

```bash
systemctl reload nginx
```

如果服务支持热加载，优先使用 `reload`，影响通常比 `restart` 更小。

### 6. 设置开机自启

```bash
systemctl enable nginx
```

表示系统启动时自动拉起该服务。

### 7. 取消开机自启

```bash
systemctl disable nginx
```

适合不希望服务随系统自动启动的场景。

---

## 四、重点命令说明

- `status`：查看服务当前状态和最近日志
- `start`：启动服务
- `stop`：停止服务
- `restart`：重启服务
- `reload`：重新加载配置
- `enable`：设置开机自启
- `disable`：取消开机自启
- `is-active`：检查服务是否正在运行
- `is-enabled`：检查服务是否已设置开机自启
- `daemon-reload`：重新加载 unit 文件
- `list-units --type=service`：查看当前已加载的服务
- `list-unit-files --type=service`：查看系统中所有服务及启用状态

示例：

```bash
systemctl is-active nginx
systemctl is-enabled nginx
systemctl list-units --type=service
systemctl list-unit-files --type=service
systemctl daemon-reload
```

---

## 五、常见实战示例

### 1. 查看服务是否启动成功

```bash
systemctl status docker
```

如果看到 `active (running)`，通常表示服务已正常运行。

### 2. 批量排查启动失败服务

```bash
systemctl --failed
```

适合在系统异常、重启后或部署完成后快速检查失败的 unit。

### 3. 修改 unit 文件后重新加载

```bash
systemctl daemon-reload
systemctl restart myapp
```

如果修改了 `/etc/systemd/system/` 下的服务文件，需要先执行 `daemon-reload`，否则 `systemd` 不会感知到变更。

### 4. 判断服务是否开机自启

```bash
systemctl is-enabled sshd
```

常见返回值：

- `enabled`：已启用开机自启
- `disabled`：未启用
- `static`：不能直接 enable，通常由其他 unit 依赖启动

### 5. 配合日志排查服务失败原因

```bash
systemctl status nginx
journalctl -u nginx -n 50 --no-pager
```

仅看 `status` 不够时，可以结合 `journalctl` 查看更完整的服务日志。

---

## 六、常见输出状态说明

使用 `systemctl status` 时，经常会看到以下状态：

- `active (running)`：服务正在运行
- `inactive`：服务未运行
- `failed`：服务启动失败或运行异常退出
- `activating`：服务正在启动中
- `deactivating`：服务正在停止中

对于开机自启状态，常见值包括：

- `enabled`：已启用
- `disabled`：未启用
- `static`：静态 unit，不能直接设置为自启
- `masked`：被屏蔽，无法启动

---

## 七、常见问题与建议

### 1. 为什么服务启动失败？

先执行：

```bash
systemctl status 服务名
journalctl -u 服务名 -n 100 --no-pager
```

重点看以下几类问题：

- 配置文件语法错误
- 端口被占用
- 依赖服务未启动
- 文件权限或路径错误
- 可执行文件不存在

### 2. `enable` 之后为什么服务没有立刻启动？

`enable` 只是设置开机自启，不会立即启动服务。要立刻运行，还需要执行：

```bash
systemctl start 服务名
```

或者直接：

```bash
systemctl enable --now 服务名
```

### 3. `reload` 和 `restart` 有什么区别？

- `reload`：重新加载配置，尽量不中断服务
- `restart`：停止后再启动，通常影响更大

如果服务支持热加载，优先考虑 `reload`。

### 4. 什么情况下要执行 `daemon-reload`？

只有在修改了 unit 文件后，才需要执行：

```bash
systemctl daemon-reload
```

如果只是改了业务配置文件，一般不需要执行这一步。

### 5. 为什么服务显示 `masked`？

说明该服务被显式屏蔽了，不能直接启动。可先取消屏蔽：

```bash
systemctl unmask 服务名
```

然后再根据需要执行 `start` 或 `enable`。

---

## 八、常用命令速查

```bash
systemctl status nginx
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx
systemctl enable nginx
systemctl disable nginx
systemctl enable --now nginx
systemctl is-active nginx
systemctl --failed
journalctl -u nginx -n 50 --no-pager
```

---

## 九、总结

`systemctl` 是 Linux 服务管理最核心的命令之一。建议至少熟练掌握以下操作：

```bash
systemctl status 服务名
systemctl start 服务名
systemctl restart 服务名
systemctl enable 服务名
systemctl --failed
journalctl -u 服务名 -n 50 --no-pager
```

掌握这些常用写法后，大多数“服务启动失败、服务未运行、开机自启配置、发布后重启、日志排障”的场景都能快速处理。
