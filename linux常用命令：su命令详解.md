# linux常用命令：su命令详解

`su` 是 Linux 中用于切换用户身份的命令。它可以从当前用户切换到 root，也可以切换到其他普通用户。

在服务器运维中，`su` 常用于临时切换用户排查权限问题、验证服务运行用户环境、进入 root shell 等场景。使用时要理解普通切换和登录式切换的区别。

---

## 一、`su` 是什么？

`su` 常被理解为 substitute user 或 switch user，用于切换当前 shell 的用户身份。

它主要用于：

- 切换到 root 用户
- 切换到指定普通用户
- 使用目标用户环境执行命令
- 排查不同用户下的权限和环境变量问题
- 验证服务用户能否访问文件或目录

常见写法：

```bash
su -
su - deploy
```

其中 `-` 表示使用登录式 shell，会加载目标用户的登录环境。

---

## 二、常见使用语法与重点参数

```bash
su [选项] [用户名]
```

常用参数说明：

- `-` 或 `-l`：登录式切换，加载目标用户环境
- `-c 命令`：以目标用户执行一条命令
- `-s shell`：指定 shell
- `-m` / `-p`：保留当前环境变量，具体支持依系统而定

常见示例：

```bash
su -
su - root
su - deploy
su - deploy -c 'whoami && pwd'
```

---

## 三、常用命令与使用场景

### 1. 切换到 root

```bash
su -
```

会提示输入 root 密码。

成功后进入 root 的登录式 shell。

### 2. 切换到指定用户

```bash
su - deploy
```

切换到 `deploy` 用户，并加载该用户的登录环境。

切换后可以确认身份：

```bash
whoami
id
```

### 3. 不带 `-` 切换用户

```bash
su deploy
```

这种方式会切换用户，但不完整加载目标用户登录环境。

很多时候更推荐：

```bash
su - deploy
```

因为它更接近用户正常登录后的环境。

### 4. 以指定用户执行命令

```bash
su - deploy -c 'whoami && pwd'
```

执行完成后返回当前用户。

适合在脚本中临时以某个用户运行一条命令。

### 5. 验证服务用户权限

```bash
su - appuser -c 'touch /var/log/myapp/test.log'
```

可以测试 `appuser` 是否对日志目录有写权限。

### 6. 指定 shell

```bash
su -s /bin/bash nginx
```

某些服务用户默认 shell 可能是 `/sbin/nologin`，临时排查时可以指定 shell。

是否允许这样使用取决于系统策略和权限配置。

---

## 四、常见问题与建议

### 1. `su` 和 `su -` 有什么区别？

- `su user`：切换用户，但保留较多当前环境
- `su - user`：登录式切换，加载目标用户环境

排查用户环境、执行部署、模拟用户登录时，通常推荐：

```bash
su - user
```

### 2. `su` 和 `sudo` 有什么区别？

- `su`：切换到另一个用户，通常需要目标用户密码
- `sudo`：以授权身份执行命令，通常需要当前用户密码和 sudo 权限

生产环境中更常用 `sudo` 做可审计、可控制的提权。

### 3. 为什么普通用户不能 `su` 到 root？

可能原因：

- root 密码不可用或未设置
- PAM 策略限制
- 用户不在允许 `su` 的组中
- 系统禁用了 root 密码登录

这类限制在生产环境中很常见。

### 4. 使用 `su` 有什么安全建议？

建议：

- 尽量使用 `su - user` 获得明确环境
- 不要共享 root 密码
- 生产环境优先使用 `sudo` 做细粒度授权
- 切换用户后先 `whoami` 和 `pwd` 确认状态
- 执行高风险命令前确认当前身份

---

## 五、常用命令速查

```bash
su -
su - root
su deploy
su - deploy
whoami
id
su - deploy -c 'whoami && pwd'
su - appuser -c 'touch /var/log/myapp/test.log'
su -s /bin/bash nginx
sudo -u deploy command
```

---

## 六、总结

`su` 是 Linux 中切换用户身份的命令。建议优先掌握：

```bash
su -
su - user
su - user -c 'command'
```

日常使用时，重点记住 `su user` 和 `su - user` 的环境差异。涉及生产权限管理时，`su` 更适合临时切换和排查，长期授权和审计通常更适合使用 `sudo`。