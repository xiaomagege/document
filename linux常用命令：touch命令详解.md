# linux常用命令：touch命令详解

在 Linux 日常使用、脚本编写和运维排查中，`touch` 是一个非常常用的文件命令。它最常见的用法是创建空文件，但它真正的作用不止于此，还可以更新文件的访问时间和修改时间。

很多时候，我们会用 `touch` 创建占位文件、初始化日志文件、触发某些依赖时间戳的任务，或者在排查文件同步、备份、构建问题时调整文件时间。理解 `touch` 的常见用法，能让文件处理更灵活。

---

## 一、`touch` 是什么？

`touch` 用于创建空文件，或更新已有文件的时间戳。

它主要用于：

- 创建一个空文件
- 一次创建多个空文件
- 更新文件的访问时间和修改时间
- 指定文件时间戳
- 在脚本中创建标记文件或占位文件

说明：

- 如果文件不存在，`touch` 默认会创建文件
- 如果文件已经存在，`touch` 默认会更新时间戳，不会清空文件内容
- 如果不想创建不存在的文件，可以使用 `-c`

---

## 二、常见使用语法与重点参数

```bash
touch [选项] 文件
```

重点参数说明：

- `-c`：文件不存在时不创建
- `-a`：只更新访问时间 atime
- `-m`：只更新修改时间 mtime
- `-t`：指定时间戳，格式通常为 `[[CC]YY]MMDDhhmm[.ss]`
- `-d`：使用可读性更好的时间字符串指定时间
- `-r`：参考另一个文件的时间戳

常见示例：

```bash
touch app.log
touch file1 file2 file3
touch -c exists.txt
touch -d "2026-05-06 10:00:00" app.log
touch -r source.txt target.txt
```

---

## 三、常用命令与使用场景

### 1. 创建一个空文件

```bash
touch app.log
```

如果 `app.log` 不存在，会创建一个空文件。

查看文件：

```bash
ls -l app.log
```

### 2. 一次创建多个空文件

```bash
touch file1.txt file2.txt file3.txt
```

会同时创建多个空文件。

适合快速初始化测试文件或占位文件。

### 3. 创建隐藏文件

```bash
touch .env
```

以 `.` 开头的文件是隐藏文件。

查看隐藏文件：

```bash
ls -la
```

这种写法常用于创建 `.env`、`.gitignore`、`.keep` 等文件。

### 4. 更新已有文件时间戳

```bash
touch app.log
```

如果 `app.log` 已经存在，文件内容不会变化，但访问时间和修改时间会更新为当前时间。

查看时间：

```bash
ls -l app.log
stat app.log
```

### 5. 只更新访问时间

```bash
touch -a app.log
```

只更新访问时间 atime，不主动更新修改时间 mtime。

这个场景相对少见，但在测试文件访问时间相关逻辑时会用到。

### 6. 只更新修改时间

```bash
touch -m app.log
```

只更新修改时间 mtime。

构建系统、备份系统、同步工具经常会关注 mtime，因此这个参数在排查时间戳相关问题时有用。

### 7. 文件不存在时不创建

```bash
touch -c app.log
```

如果 `app.log` 存在，就更新时间戳；如果不存在，则什么也不做。

适合只想更新已有文件时间，不希望误创建新文件的场景。

### 8. 指定文件时间戳

```bash
touch -t 202605061030 app.log
```

表示把 `app.log` 的时间设置为：

```text
2026-05-06 10:30
```

`-t` 的格式一般是：

```text
[[CC]YY]MMDDhhmm[.ss]
```

例如：

```bash
touch -t 202605061030.45 app.log
```

表示设置到 2026-05-06 10:30:45。

### 9. 使用可读时间字符串设置时间

```bash
touch -d "2026-05-06 10:30:00" app.log
```

`-d` 比 `-t` 更直观，可读性更好。

也可以使用相对时间：

```bash
touch -d "1 day ago" app.log
```

### 10. 参考另一个文件的时间戳

```bash
touch -r source.txt target.txt
```

表示把 `target.txt` 的时间戳设置为和 `source.txt` 一样。

适合文件同步、备份恢复、构建缓存排查等场景。

### 11. 创建目录保留文件

```bash
touch logs/.keep
```

有些空目录不会被版本管理工具提交，可以在目录里创建一个 `.keep` 文件用于保留目录结构。

例如：

```bash
mkdir -p logs
touch logs/.keep
```

### 12. 脚本中创建标记文件

```bash
touch /tmp/deploy.done
```

脚本执行完成后创建一个标记文件，后续流程可以通过它判断某个步骤是否完成。

检查标记文件：

```bash
[ -f /tmp/deploy.done ] && echo "done"
```

---

## 四、常见问题与建议

### 1. `touch` 会清空文件内容吗？

不会。

如果文件已经存在，`touch file` 只更新时间戳，不会修改文件内容。

如果你想清空文件，通常使用：

```bash
> app.log
```

或者：

```bash
truncate -s 0 app.log
```

不要把 `touch` 理解成清空文件的命令。

### 2. 为什么 `touch` 文件时提示 Permission denied？

常见原因：

- 当前目录没有写权限
- 目标文件存在但当前用户没有修改权限
- 父目录权限不允许创建文件

可以检查：

```bash
pwd
ls -ld .
ls -l app.log
whoami
```

### 3. `touch` 创建文件后为什么大小是 0？

因为 `touch` 创建的是空文件，默认不会写入任何内容。

查看大小：

```bash
ls -lh app.log
```

如果需要写入内容，可以使用：

```bash
echo "hello" > app.log
```

### 4. `touch -c` 适合什么场景？

适合只想更新已有文件时间，但不希望文件不存在时被创建的场景。

例如：

```bash
touch -c /data/app/app.log
```

如果路径写错，`-c` 不会生成一个意外的新文件。

### 5. 为什么修改时间会影响构建或同步？

很多工具会根据文件修改时间判断是否需要重新处理文件，例如：

- 构建工具
- 备份工具
- 文件同步工具
- 定时清理脚本

因此，`touch` 可以用于测试或触发这类基于时间戳的流程。

### 6. 生产环境中使用 `touch` 有什么建议？

建议：

- 创建文件前先确认当前目录 `pwd`
- 脚本中使用绝对路径
- 不要误以为 `touch` 会清空文件
- 更新时间戳前确认是否会影响同步、备份或构建流程

---

## 五、常用命令速查

```bash
touch app.log
touch file1.txt file2.txt file3.txt
touch .env
touch -a app.log
touch -m app.log
touch -c app.log
touch -t 202605061030 app.log
touch -d "2026-05-06 10:30:00" app.log
touch -d "1 day ago" app.log
touch -r source.txt target.txt
touch logs/.keep
touch /tmp/deploy.done
```

---

## 六、总结

`touch` 是 Linux 下常用的文件创建和时间戳更新命令。建议优先掌握以下几条：

```bash
touch file.txt
touch file1 file2 file3
touch -c file.txt
touch -d "2026-05-06 10:30:00" file.txt
touch -r source.txt target.txt
```

实际工作中，`touch` 最常见的用途是创建空文件和更新文件时间。它不会清空已有文件内容，这是和重定向、`truncate` 等命令最大的区别。遇到时间戳相关的问题时，可以配合 `ls -l`、`stat` 一起排查。
