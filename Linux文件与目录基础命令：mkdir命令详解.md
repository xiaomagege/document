# Linux文件与目录基础命令：mkdir命令详解

在 Linux 日常使用、项目部署和服务器运维中，`mkdir` 是最常用的目录创建命令。无论是创建项目目录、日志目录、备份目录，还是在脚本中初始化目录结构，`mkdir` 都非常高频。

`mkdir` 看起来只是“创建目录”，但实际工作中经常会遇到多级目录不存在、目录已经存在、权限不足、目录权限不符合预期等问题。掌握几个常用参数，可以让目录创建更稳定、更适合脚本化操作。

---

## 一、`mkdir` 是什么？

`mkdir` 是 make directory 的缩写，用于创建目录。

它主要用于：

- 创建单个目录
- 一次创建多个目录
- 递归创建多级目录
- 创建目录时指定权限
- 在脚本中初始化目录结构

说明：

- 如果目标目录已经存在，普通 `mkdir` 会报错
- 如果父目录不存在，普通 `mkdir` 也会报错
- 使用 `-p` 可以递归创建多级目录，并且目录已存在时不报错

---

## 二、常见使用语法与重点参数

```bash
mkdir [选项] 目录
```

重点参数说明：

- `-p`：递归创建多级目录，目录已存在时不报错
- `-m`：创建目录时指定权限
- `-v`：显示创建过程

常见示例：

```bash
mkdir logs
mkdir -p /data/app/logs
mkdir -m 755 scripts
mkdir -pv /data/app/{logs,tmp,backup}
```

日常使用中最常用的是 `mkdir` 和 `mkdir -p`。

---

## 三、常用命令与使用场景

### 1. 创建单个目录

```bash
mkdir logs
```

会在当前目录下创建一个名为 `logs` 的目录。

创建后可以查看：

```bash
ls -ld logs
```

### 2. 一次创建多个目录

```bash
mkdir logs tmp backup
```

会在当前目录下同时创建三个目录：

```bash
logs
tmp
backup
```

适合初始化简单目录结构。

### 3. 创建指定路径下的目录

```bash
mkdir /data/app/logs
```

如果 `/data/app` 已经存在，就会创建 `/data/app/logs`。

如果父目录 `/data/app` 不存在，则会报错：

```bash
No such file or directory
```

这种场景更适合使用 `-p`。

### 4. 递归创建多级目录

```bash
mkdir -p /data/app/logs
```

如果 `/data`、`/data/app`、`/data/app/logs` 中某些目录不存在，`mkdir -p` 会自动按层级创建。

这是部署脚本和初始化脚本中最常用的写法。

### 5. 目录已存在时不报错

```bash
mkdir -p /data/app/logs
```

如果目录已经存在，命令不会报错。

这对脚本很重要，因为脚本重复执行时，不会因为目录已经存在而中断。

### 6. 创建目录时显示过程

```bash
mkdir -pv /data/app/logs
```

示例输出：

```bash
mkdir: created directory '/data'
mkdir: created directory '/data/app'
mkdir: created directory '/data/app/logs'
```

适合排查脚本创建目录时到底创建了哪些路径。

### 7. 创建目录时指定权限

```bash
mkdir -m 755 scripts
```

表示创建 `scripts` 目录，并指定权限为 `755`。

查看权限：

```bash
ls -ld scripts
```

说明：

- `755`：属主可读写执行，其他用户可读和进入
- `700`：只有属主可读写和进入
- `775`：属主和用户组可读写进入，其他用户可读和进入

实际权限还可能受到 `umask` 影响，必要时创建后用 `chmod` 再调整。

### 8. 使用花括号创建多个子目录

```bash
mkdir -p /data/app/{logs,tmp,backup}
```

会创建：

```bash
/data/app/logs
/data/app/tmp
/data/app/backup
```

适合快速创建一组同级目录。

### 9. 创建日期目录

```bash
mkdir -p /data/backup/$(date +%F)
```

示例目录：

```bash
/data/backup/2026-05-06
```

适合日志备份、数据库备份、文件归档等场景。

### 10. 脚本中创建目录前先确认路径

```bash
backup_dir=/data/backup/$(date +%F)
mkdir -p "$backup_dir"
```

变量加双引号是一个好习惯，可以避免路径中包含空格或特殊字符时出错。

### 11. 创建目录后立即进入

```bash
mkdir -p /data/app/logs
cd /data/app/logs
pwd
```

这种组合适合初始化目录并继续在目录中操作。

也可以写成：

```bash
mkdir -p /data/app/logs && cd /data/app/logs
```

`&&` 表示前一个命令成功后才执行后一个命令。

### 12. 创建目录失败时排查原因

```bash
mkdir /root/test
```

如果普通用户执行，可能提示：

```bash
Permission denied
```

常见原因包括：

- 父目录不存在
- 当前用户没有写权限
- 路径写错
- 目标文件名已经被普通文件占用

可以检查：

```bash
pwd
ls -ld /root
ls -ld /data/app
```

---

## 四、常见问题与建议

### 1. `mkdir` 和 `mkdir -p` 有什么区别？

普通 `mkdir` 只创建最后一级目录，要求父目录必须存在，且目标目录不能已经存在。

`mkdir -p` 会递归创建多级目录，并且目录已存在时不报错。

脚本中通常优先使用：

```bash
mkdir -p /path/to/dir
```

### 2. 为什么创建目录时提示 File exists？

说明目标目录已经存在。

例如：

```bash
mkdir logs
```

如果 `logs` 已经存在，就会报错。可以改用：

```bash
mkdir -p logs
```

### 3. 为什么创建目录时提示 No such file or directory？

通常是父目录不存在。

例如：

```bash
mkdir /data/app/logs
```

如果 `/data/app` 不存在，就会失败。使用：

```bash
mkdir -p /data/app/logs
```

可以递归创建。

### 4. 为什么创建目录时提示 Permission denied？

说明当前用户对父目录没有写权限。

查看父目录权限：

```bash
ls -ld /data
```

如果没有权限，需要切换到有权限的目录，或由管理员调整权限。

### 5. `mkdir -m` 设置的权限为什么和预期不一致？

可能受 `umask` 影响。

可以查看当前 `umask`：

```bash
umask
```

如果需要严格控制权限，可以创建后再执行：

```bash
chmod 755 scripts
```

### 6. 生产环境中使用 `mkdir` 有什么建议？

建议：

- 脚本中优先使用 `mkdir -p`
- 变量路径加双引号
- 创建关键目录后用 `ls -ld` 检查权限
- 不要在不确认路径的情况下创建到错误位置

例如：

```bash
app_dir=/data/app
mkdir -p "$app_dir/logs" "$app_dir/tmp"
ls -ld "$app_dir" "$app_dir/logs" "$app_dir/tmp"
```

---

## 五、常用命令速查

```bash
mkdir logs
mkdir logs tmp backup
mkdir /data/app/logs
mkdir -p /data/app/logs
mkdir -pv /data/app/logs
mkdir -m 755 scripts
mkdir -p /data/app/{logs,tmp,backup}
mkdir -p /data/backup/$(date +%F)
mkdir -p /data/app/logs && cd /data/app/logs
```

---

## 六、总结

`mkdir` 是 Linux 下最基础的目录创建命令。建议优先掌握以下几条：

```bash
mkdir dir
mkdir -p /path/to/dir
mkdir -pv /path/to/dir
mkdir -m 755 dir
mkdir -p /data/app/{logs,tmp,backup}
```

实际工作中，最常用也最稳妥的是 `mkdir -p`。它能递归创建多级目录，并且目录已存在时不会报错，非常适合脚本和重复执行的初始化任务。遇到创建失败时，优先检查父目录是否存在、当前用户是否有写权限，以及路径是否写对。
