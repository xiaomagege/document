# Linux文本处理与筛选命令：sort命令详解

`sort` 是 Linux 中用于排序文本内容的常用命令。它通常按行处理输入，可以对文件、命令输出、日志统计结果进行排序。

在日常运维和数据处理场景中，`sort` 经常和 `awk`、`uniq`、`head`、`tail` 组合，用来做排行榜、去重前排序、按数值排序、按字段排序等操作。

---

## 一、`sort` 是什么？

`sort` 用于对文本行进行排序。

它主要用于：

- 对文件内容按字典序排序
- 按数字大小排序
- 逆序排序
- 按指定字段排序
- 去重排序
- 为 `uniq` 提供相邻重复行
- 生成日志统计排行

需要注意的是，`sort` 默认按整行字典序排序，不是按数字大小排序。如果要按数字排序，需要使用 `-n` 或 `-h`。

---

## 二、常见使用语法与重点参数

```bash
sort [选项] [文件]
命令 | sort [选项]
```

常用参数说明：

- `-n`：按数字大小排序
- `-h`：按人类可读容量排序，例如 `1K`、`20M`、`3G`
- `-r`：逆序排序
- `-u`：排序并去重
- `-k`：按指定字段排序
- `-t`：指定字段分隔符
- `-o 文件`：把结果输出到文件
- `-V`：按版本号排序
- `-M`：按月份名称排序

常见示例：

```bash
sort file.txt
sort -n nums.txt
sort -nr nums.txt
sort -u file.txt
sort -t ':' -k 3 -n /etc/passwd
```

---

## 三、常用命令与使用场景

### 1. 按字典序排序

```bash
sort names.txt
```

默认按整行文本进行字典序排序。

适合排序用户名、域名、普通字符串等文本。

### 2. 按数字排序

```bash
sort -n nums.txt
```

如果不加 `-n`，`10` 可能会排在 `2` 前面，因为默认按字符串比较。

按数字从大到小排序：

```bash
sort -nr nums.txt
```

### 3. 按人类可读容量排序

```bash
du -h --max-depth=1 /var | sort -h
```

`-h` 可以识别 `K`、`M`、`G` 等单位。

查看最大的目录可以用：

```bash
du -h --max-depth=1 /var | sort -hr
```

### 4. 排序并去重

```bash
sort -u file.txt
```

会先排序再去重。

如果只是去掉相邻重复行，也可以使用 `uniq`，但多数情况下需要先 `sort`：

```bash
sort file.txt | uniq
```

### 5. 按指定字段排序

```bash
sort -k 2 file.txt
```

按第 2 个字段排序。

默认字段分隔符是空白字符。

### 6. 指定分隔符后按字段排序

```bash
sort -t ':' -k 3 -n /etc/passwd
```

`-t ':'` 指定冒号为字段分隔符，`-k 3` 表示按第 3 个字段排序，`-n` 表示按数字大小排序。

这个例子会按 UID 排序 `/etc/passwd`。

### 7. 按统计次数排序

统计日志中 IP 访问次数：

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

含义：

- `awk '{print $1}'` 提取 IP
- `sort` 让相同 IP 相邻
- `uniq -c` 统计次数
- `sort -nr` 按次数从大到小排序
- `head` 查看前几名

### 8. 按版本号排序

```bash
sort -V versions.txt
```

适合排序类似 `v1.2.9`、`v1.2.10` 这样的版本字符串。

### 9. 输出排序结果到文件

```bash
sort names.txt -o names.txt
```

`-o` 可以把排序结果写回文件。

不要使用下面这种写法：

```bash
sort names.txt > names.txt
```

因为 shell 可能会先清空输出文件，再执行 `sort`，导致数据丢失。

---

## 四、常见问题与建议

### 1. 为什么 `sort` 排数字结果不对？

因为默认按字符串排序。

数字排序应使用：

```bash
sort -n nums.txt
```

带单位容量排序应使用：

```bash
sort -h sizes.txt
```

### 2. `sort` 和 `uniq` 为什么经常一起用？

`uniq` 只能处理相邻的重复行。

所以统计重复项时通常先排序：

```bash
sort file.txt | uniq -c
```

### 3. 如何按指定列倒序排序？

例如按第 2 列数字倒序：

```bash
sort -k 2 -nr file.txt
```

如果有分隔符，例如冒号：

```bash
sort -t ':' -k 3 -nr file.txt
```

### 4. 大文件排序有什么建议？

`sort` 处理大文件可能消耗较多磁盘临时空间和内存。

可以指定临时目录：

```bash
sort -T /data/tmp big.log
```

也可以通过 `LC_ALL=C` 提升普通字节排序速度：

```bash
LC_ALL=C sort big.log
```

---

## 五、常用命令速查

```bash
sort file.txt
sort -n nums.txt
sort -nr nums.txt
sort -h sizes.txt
sort -hr sizes.txt
sort -u file.txt
sort file.txt | uniq
sort -k 2 file.txt
sort -k 2 -nr file.txt
sort -t ':' -k 3 -n /etc/passwd
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
sort -V versions.txt
sort names.txt -o names.txt
LC_ALL=C sort big.log
sort -T /data/tmp big.log
```

---

## 六、总结

`sort` 是 Linux 文本处理流水线中非常基础的排序命令。建议重点掌握：

```bash
sort file
sort -n file
sort -nr file
sort -u file
sort -k 2 file
```

实际工作中，`sort` 最常见的组合是 `sort | uniq -c | sort -nr`，用于统计重复项排行。只要区分清楚字典序、数字排序和容量排序，就能避免大多数排序结果不符合预期的问题。