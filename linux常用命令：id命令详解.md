# linux常用命令：id命令详解

在 Linux 权限排查中，只知道当前用户名往往不够，还需要知道 UID、GID、所属用户组等信息。`id` 就是用来查看用户身份信息的常用命令。

当文件权限看起来没问题，但用户仍然无法访问目录或写入文件时，`id` 可以帮助确认该用户是否属于正确的组。

---

## 一、`id` 是什么？

`id` 用于显示当前用户或指定用户的身份信息。

它主要用于：

- 查看当前用户 UID
- 查看当前用户主组 GID
- 查看用户所属附加组
- 判断用户是否属于某个组
- 排查文件和目录权限问题
- 配合 `chown`、`chgrp`、`chmod` 分析权限

直接执行：

```bash
id
```

会输出类似：

```bash
uid=1000(deploy) gid=1000(deploy) groups=1000(deploy),10(wheel)
```

---

## 二、常见使用语法与重点参数

```bash
id [选项] [用户名]
```

常用参数说明：

- `-u`：只显示 UID
- `-g`：只显示主组 GID
- `-G`：显示所有组 ID
- `-n`：显示名称而不是数字，通常和 `-u`、`-g`、`-G` 配合
- `-r`：显示真实 ID，而不是有效 ID

常见示例：

```bash
id
id deploy
id -u
id -un
id -Gn
```

---

## 三、常用命令与使用场景

### 1. 查看当前用户身份信息

```bash
id
```

会显示 UID、GID 和所属组。

这是排查权限问题时非常常用的命令。

### 2. 查看指定用户信息

```bash
id deploy
```

查看 `deploy` 用户的 UID、GID 和所属组。

如果用户不存在，会提示没有该用户。

### 3. 只查看当前用户 UID

```bash
id -u
```

脚本中常用于判断是否 root。

root 用户 UID 是 0：

```bash
if [ "$(id -u)" -ne 0 ]; then
  echo "please run as root"
  exit 1
fi
```

### 4. 查看当前用户名

```bash
id -un
```

通常和 `whoami` 输出一致。

### 5. 查看当前用户主组

```bash
id -gn
```

显示当前用户的主组名称。

如果想显示主组 ID：

```bash
id -g
```

### 6. 查看当前用户所有组

```bash
id -Gn
```

输出当前用户所属的所有组名称。

这在判断用户是否有共享目录访问权限时很有用。

### 7. 判断用户是否属于某个组

```bash
id -Gn deploy | tr ' ' '\n' | grep '^www$'
```

如果有输出，说明 `deploy` 属于 `www` 组。

### 8. 配合文件权限排查

```bash
id deploy
ls -l app.log
ls -ld /data/app
```

通过用户组信息、文件属主和权限一起判断为什么能访问或不能访问。

---

## 四、常见问题与建议

### 1. UID 和 GID 是什么？

- UID：用户 ID
- GID：用户主组 ID
- groups：用户所属的所有组

Linux 内部主要用数字 ID 做权限判断，用户名和组名只是更易读的显示形式。

### 2. 用户刚加入组，为什么 `id` 看不到？

用户组变更后，当前登录会话可能不会立即生效。

可以重新登录，或重新打开 shell，再查看：

```bash
id -Gn
```

### 3. `id -u` 为什么常用于脚本？

因为它可以稳定判断当前是否 root：

```bash
[ "$(id -u)" -eq 0 ] && echo root
```

root 的 UID 固定为 0。

### 4. 生产环境中如何用 `id` 排查权限？

建议按顺序看：

```bash
whoami
id
ls -l 文件
ls -ld 目录
```

确认当前用户、所属组、文件属主、所属组和权限位，再决定是改用户组、改属主还是改权限。

---

## 五、常用命令速查

```bash
id
id deploy
id -u
id -un
id -g
id -gn
id -G
id -Gn
id -Gn deploy | tr ' ' '\n' | grep '^www$'
whoami
ls -l app.log
ls -ld /data/app
[ "$(id -u)" -eq 0 ] && echo root
```

---

## 六、总结

`id` 是查看用户身份和用户组信息的基础命令。建议优先掌握：

```bash
id
id username
id -u
id -un
id -Gn
```

权限问题不能只看文件权限位，还要看当前用户属于哪些组。`id`、`ls -l`、`chmod`、`chown`、`chgrp` 配合使用，才能完整判断 Linux 权限链路。