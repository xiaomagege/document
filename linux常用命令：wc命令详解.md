# linux常用命令：wc命令详解

`wc` 是 Linux 中用于统计文本行数、单词数、字节数和字符数的常用命令。它看起来简单，但在脚本判断、日志统计、数据核对和命令结果计数中非常实用。

例如统计日志行数、统计某个命令输出了多少条记录、判断文件是否为空，都会经常用到 `wc`。

---

## 一、`wc` 是什么？

`wc` 是 word count 的缩写，用于统计文本内容的数量信息。

它主要用于：

- 统计文件行数
- 统计单词数
- 统计字节数
- 统计字符数
- 统计命令输出条数
- 配合管道做结果计数

默认执行 `wc file.txt` 会同时输出行数、单词数、字节数和文件名。

---

## 二、常见使用语法与重点参数

```bash
wc [选项] [文件]
命令 | wc [选项]
```

常用参数说明：

- `-l`：统计行数
- `-w`：统计单词数
- `-c`：统计字节数
- `-m`：统计字符数
- `-L`：显示最长行长度

常见示例：

```bash
wc file.txt
wc -l file.txt
wc -w file.txt
wc -c file.txt
cat file.txt | wc -l
```

---

## 三、常用命令与使用场景

### 1. 查看文件统计信息

```bash
wc file.txt
```

默认输出通常包含：

```bash
行数 单词数 字节数 文件名
```

适合快速了解文本文件规模。

### 2. 统计文件行数

```bash
wc -l access.log
```

这是 `wc` 最常用的写法之一。

它可以统计日志、清单、CSV、结果文件有多少行。

### 3. 统计命令输出行数

```bash
ps -ef | wc -l
```

可以统计当前进程输出行数。

统计包含关键字的日志行数：

```bash
grep 'ERROR' app.log | wc -l
```

### 4. 统计单词数

```bash
wc -w README.md
```

`-w` 按单词统计，适合英文文本或按空白分隔的内容。

对中文文章来说，`wc -w` 的结果通常不适合作为词数统计依据。

### 5. 统计字节数

```bash
wc -c app.log
```

`-c` 统计字节数，和文件大小相关。

如果文件包含中文等多字节字符，字节数和字符数可能不同。

### 6. 统计字符数

```bash
wc -m file.txt
```

`-m` 统计字符数。

对于包含中文的文本，`wc -m` 通常比 `wc -c` 更符合“字符数量”的理解。

### 7. 查看最长行长度

```bash
wc -L access.log
```

可以查看文件中最长一行的长度。

排查日志格式异常、某些行特别长、导入文件字段过长时有帮助。

### 8. 统计目录下文件数量

```bash
find /var/log -type f | wc -l
```

`wc` 经常和 `find` 配合，用于统计文件数量。

统计当前目录下普通文件数量：

```bash
find . -maxdepth 1 -type f | wc -l
```

### 9. 统计去重后的数量

```bash
awk '{print $1}' access.log | sort -u | wc -l
```

可以统计访问日志中不同 IP 的数量。

这是日志分析中非常常见的组合。

---

## 四、常见问题与建议

### 1. `wc -l` 统计的是行数还是换行符数量？

`wc -l` 统计的是换行符数量。

如果文件最后一行没有换行符，某些情况下结果可能和肉眼看到的“行数”不完全一致。

### 2. 为什么 `wc -c` 和 `wc -m` 不一样？

`wc -c` 统计字节数，`wc -m` 统计字符数。

在 UTF-8 文本中，一个中文字符通常占多个字节，所以两者可能不同。

### 3. 管道中 `wc -l` 的结果为什么比预期多 1？

有些命令输出包含表头。

例如：

```bash
ps -ef | wc -l
```

会把表头也算进去。如果只统计实际进程，需要根据场景过滤表头。

### 4. 脚本中如何只拿数字？

管道方式通常只输出数字：

```bash
grep 'ERROR' app.log | wc -l
```

如果直接统计文件：

```bash
wc -l app.log
```

会带文件名。可以用：

```bash
wc -l < app.log
```

只输出行数。

---

## 五、常用命令速查

```bash
wc file.txt
wc -l file.txt
wc -w README.md
wc -c app.log
wc -m file.txt
wc -L access.log
cat file.txt | wc -l
wc -l < app.log
grep 'ERROR' app.log | wc -l
find /var/log -type f | wc -l
find . -maxdepth 1 -type f | wc -l
awk '{print $1}' access.log | sort -u | wc -l
```

---

## 六、总结

`wc` 是 Linux 中最常用的统计类命令之一。建议优先掌握：

```bash
wc -l file
wc -c file
wc -m file
命令 | wc -l
```

它最大的价值在于和管道组合使用。无论是统计日志匹配行数、文件数量、去重后的结果数量，还是在脚本中做条件判断，`wc` 都是非常简单可靠的工具。