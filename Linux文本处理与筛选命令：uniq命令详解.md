# Linux文本处理与筛选命令：uniq命令详解

`uniq` 是 Linux 中用于处理重复行的常用命令。它可以删除相邻重复行，也可以统计重复次数，常和 `sort` 一起用于日志统计、列表去重和数据清洗。

需要注意的是，`uniq` 只能识别相邻的重复行。如果重复内容分散在文件不同位置，通常需要先 `sort` 再 `uniq`。

---

## 一、`uniq` 是什么？

`uniq` 用于报告或过滤文本中的重复行。

它主要用于：

- 删除相邻重复行
- 统计重复行出现次数
- 只显示重复行
- 只显示不重复行
- 配合 `sort` 做去重统计
- 统计日志中的 IP、状态码、接口访问次数

`uniq` 的关键点是“相邻重复”。这也是它和很多人直觉不一样的地方。

---

## 二、常见使用语法与重点参数

```bash
uniq [选项] [输入文件 [输出文件]]
命令 | uniq [选项]
```

常用参数说明：

- `-c`：统计每行重复次数
- `-d`：只显示重复出现的行
- `-u`：只显示只出现一次的行
- `-i`：忽略大小写
- `-f N`：比较时跳过前 N 个字段
- `-s N`：比较时跳过前 N 个字符
- `-w N`：只比较前 N 个字符

常见示例：

```bash
uniq file.txt
sort file.txt | uniq
sort file.txt | uniq -c
sort file.txt | uniq -d
sort file.txt | uniq -u
```

---

## 三、常用命令与使用场景

### 1. 删除相邻重复行

```bash
uniq file.txt
```

如果文件内容是：

```bash
a
a
b
a
```

输出会是：

```bash
a
b
a
```

最后一行 `a` 不会被去掉，因为它和前面的 `a` 不相邻。

### 2. 先排序再去重

```bash
sort file.txt | uniq
```

这是最常见的去重写法。

如果只是想得到唯一列表，也可以用：

```bash
sort -u file.txt
```

### 3. 统计重复次数

```bash
sort file.txt | uniq -c
```

会在每行前显示出现次数。

如果想按次数从高到低排序：

```bash
sort file.txt | uniq -c | sort -nr
```

### 4. 统计访问 IP 次数

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

这是日志分析中非常常见的命令组合。

它可以快速找出访问次数最多的 IP。

### 5. 只显示重复行

```bash
sort file.txt | uniq -d
```

只输出出现过多次的行，但每个重复值只显示一次。

如果需要看到重复次数，可以使用：

```bash
sort file.txt | uniq -c | sort -nr
```

### 6. 只显示唯一行

```bash
sort file.txt | uniq -u
```

只显示在排序后只出现一次的行。

适合找出没有重复的记录。

### 7. 忽略大小写去重

```bash
sort -f file.txt | uniq -i
```

`-i` 忽略大小写。

为了让大小写不同的内容相邻，排序时也建议加 `-f`。

### 8. 跳过前几个字段比较

```bash
uniq -f 1 file.txt
```

比较时跳过第 1 个字段。

例如某些日志第一列是时间，但后面内容才是要比较的主体，就可以使用 `-f`。

### 9. 只比较前几个字符

```bash
uniq -w 8 file.txt
```

只比较每行前 8 个字符。

适合按固定前缀去重，例如日期、编号前缀等。

---

## 四、常见问题与建议

### 1. 为什么 `uniq` 没有去掉所有重复行？

因为 `uniq` 只处理相邻重复行。

如果要全局去重，先排序：

```bash
sort file.txt | uniq
```

或直接：

```bash
sort -u file.txt
```

### 2. `uniq -c` 前面的数字是什么意思？

表示该行连续重复出现的次数。

通常和 `sort` 配合后，这个数字就可以理解为该值在全局出现的次数：

```bash
sort file.txt | uniq -c
```

### 3. `uniq` 和 `sort -u` 怎么选？

简单去重可以使用：

```bash
sort -u file.txt
```

如果要统计次数、只看重复项或只看唯一项，使用：

```bash
sort file.txt | uniq -c
sort file.txt | uniq -d
sort file.txt | uniq -u
```

### 4. 生产环境中怎么用更稳妥？

建议：

- 统计前先确认提取的字段是否正确
- `uniq` 前通常先 `sort`
- 大文件排序时注意临时空间
- 日志分析时先缩小时间范围，避免全量扫描过慢

---

## 五、常用命令速查

```bash
uniq file.txt
sort file.txt | uniq
sort -u file.txt
sort file.txt | uniq -c
sort file.txt | uniq -c | sort -nr
sort file.txt | uniq -d
sort file.txt | uniq -u
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
sort -f file.txt | uniq -i
uniq -f 1 file.txt
uniq -w 8 file.txt
```

---

## 六、总结

`uniq` 是处理重复行的基础命令。它最重要的特点是只处理相邻重复行，所以实际使用中经常写成：

```bash
sort file.txt | uniq
sort file.txt | uniq -c | sort -nr
```

掌握 `uniq -c`、`uniq -d`、`uniq -u` 这几种用法，就能完成大部分去重和重复项统计任务。