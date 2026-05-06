# linux常用命令：sudo命令详解

`sudo` 是 Linux 中用于以其他用户身份执行命令的工具，最常见的是让普通用户以 root 权限执行管理命令。

相比直接切换到 root，`sudo` 更适合生产环境中的权限控制和审计。它可以按用户、用户组、命令范围进行授权，减少长期使用 root 的风险。

---

## 一、`sudo` 是什么？

`sudo` 是 superuser do 的常见理解，用于以授权身份执行命令。

它主要用于：

- 普通用户执行需要 root 权限的命令
- 以指定用户身份执行命令
- 控制哪些用户可以执行哪些命令
- 记录提权操作日志
- 避免共享 root 密码
- 配合自动化部署执行受控管理操作

常见用法：

```bash
sudo command
```

例如：

```bash
sudo systemctl restart nginx
```

---

## 二、常见使用语法与重点参数

```bash
sudo [选项] 命令
```

常用参数说明：

- `-u 用户`：以指定用户身份执行命令
- `-i`：打开目标用户的登录 shell，常用于 root shell
- `-s`：使用当前 shell 或指定 shell
- `-l`：列出当前用户可执行的 sudo 命令
- `-k`：清除当前 sudo 时间戳，下次需要重新认证
- `-v`：刷新 sudo 认证时间
- `-E`：保留环境变量，需要谨慎使用

常见示例：

```bash
sudo systemctl status nginx
sudo -u deploy whoami
sudo -i
sudo -l
sudo -k
```

---

## 三、常用命令与使用场景

### 1. 以 root 权限执行命令

```bash
sudo systemctl restart nginx
```

普通用户如果有 sudo 权限，就可以执行需要管理员权限的命令。

### 2. 以指定用户执行命令

```bash
sudo -u deploy whoami
```

会以 `deploy` 用户身份执行 `whoami`。

常用于验证服务用户权限：

```bash
sudo -u appuser touch /var/log/myapp/test.log
```

### 3. 进入 root 登录 shell

```bash
sudo -i
```

进入 root 的登录式 shell。

执行高风险操作前建议确认：

```bash
whoami
pwd
```

### 4. 查看当前用户 sudo 权限

```bash
sudo -l
```

会列出当前用户被允许执行的命令。

排查“为什么没有 sudo 权限”时很有用。

### 5. 清除 sudo 缓存认证

```bash
sudo -k
```

下次执行 sudo 时会重新要求认证。

### 6. 刷新 sudo 认证时间

```bash
sudo -v
```

可以更新 sudo 认证时间戳，常用于需要连续执行多个 sudo 命令的脚本前置检查。

### 7. 编辑 sudoers 文件

```bash
sudo visudo
```

不要直接用普通编辑器改 `/etc/sudoers`，因为语法错误可能导致 sudo 不可用。

`visudo` 会在保存前做语法检查。

### 8. sudoers 常见授权示例

允许 `deploy` 重启 nginx：

```bash
deploy ALL=(root) /bin/systemctl restart nginx
```

允许 `wheel` 组使用 sudo：

```bash
%wheel ALL=(ALL) ALL
```

具体路径要用系统中的真实命令路径，可以用：

```bash
which systemctl
```

---

## 四、常见问题与建议

### 1. `sudo: command not found` 怎么办？

可能是系统没有安装 sudo，或当前 PATH 找不到。

在最小化系统或容器中比较常见。

需要使用 root 安装或改用系统已有的权限方式。

### 2. 用户不在 sudoers 中怎么办？

报错可能类似：

```bash
user is not in the sudoers file
```

需要由管理员把用户加入 sudo 授权，例如加入 `wheel` 或 `sudo` 组，具体取决于发行版。

### 3. `sudo` 和 `su` 怎么选？

简单理解：

- 临时执行一条管理员命令：用 `sudo`
- 长时间切换到另一个用户环境：用 `su - user` 或 `sudo -iu user`
- 生产环境授权和审计：优先 `sudo`

### 4. 为什么不建议所有人都能 `sudo ALL`？

`sudo ALL` 权限很大，接近完整 root 权限。

生产环境应尽量按命令授权，或者通过用户组控制，并保留审计记录。

### 5. 生产环境使用 `sudo` 有什么建议？

建议：

- 使用 `visudo` 修改 sudoers
- 按最小权限授权
- 不共享 root 密码
- 对高风险命令限制范围
- 定期审查 sudo 权限
- 脚本中避免滥用 `sudo -E` 保留环境变量

---

## 五、常用命令速查

```bash
sudo command
sudo systemctl restart nginx
sudo systemctl status nginx
sudo -u deploy whoami
sudo -u appuser touch /var/log/myapp/test.log
sudo -i
whoami
pwd
sudo -l
sudo -k
sudo -v
sudo visudo
which systemctl
sudo -iu deploy
```

---

## 六、总结

`sudo` 是 Linux 中最常用的受控提权工具。建议优先掌握：

```bash
sudo command
sudo -u user command
sudo -i
sudo -l
sudo visudo
```

生产环境中，`sudo` 的价值不只是提权，更是控制和审计。合理的 sudo 授权应该遵循最小权限原则，让用户能完成必要操作，但不获得不必要的 root 能力。