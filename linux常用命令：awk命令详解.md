# linux常用命令：awk命令详解

在 Linux 文本处理工具中，`awk` 是非常强大的一个。它特别适合处理“按行、按列”组织的数据，例如日志、配置、命令输出、CSV 类文本和统计结果。

如果说 `grep` 更擅长按关键字筛选行，`sed` 更擅长按规则替换文本，那么 `awk` 更擅长按字段提取、计算、格式化和汇总。

---

## 一、`awk` 是什么？

`awk` 是一种文本处理工具，也可以看作一门小型文本处理语言。

它主要用于：

- 按列提取文本
- 根据条件过滤行
- 对字段进行计算
- 格式化输出
- 统计日志或命令结果
- 处理结构化文本数据

`awk` 默认按行读取输入，每一行再按空白字符切分成字段。

常见内置变量：

- `$0`：整行内容
- `$1`：第 1 个字段
- `$2`：第 2 个字段
- `NF`：当前行字段数量
- `NR`：当前处理的行号
- `FS`：输入字段分隔符
- `OFS`：输出字段分隔符

---

## 二、常见使用语法与重点参数

```bash
awk '条件 {动作}' 文件
命令 | awk '条件 {动作}'
```

常用参数说明：

- `-F 分隔符`：指定输入字段分隔符
- `-v 变量=值`：传入外部变量
- `-f 脚本文件`：从脚本文件读取 awk 程序

常见示例：

```bash
awk '{print $1}' access.log
awk -F ':' '{print $1}' /etc/passwd
awk '$5 > 80 {print $0}' disk.txt
awk '{sum += $1} END {print sum}' nums.txt
```

---

## 三、常用命令与使用场景

### 1. 打印整行内容

```bash
awk '{print $0}' file.txt
```

`$0` 表示当前整行。

这个写法和 `cat file.txt` 类似，但通常用于和条件、字段处理组合。

### 2. 打印指定字段

```bash
awk '{print $1, $2}' access.log
```

默认按空白字符分隔字段，输出第 1 列和第 2 列。

例如查看 `ls -l` 输出中的权限和文件名：

```bash
ls -l | awk '{print $1, $9}'
```

### 3. 指定分隔符

```bash
awk -F ':' '{print $1, $7}' /etc/passwd
```

`/etc/passwd` 使用冒号分隔字段，这里会打印用户名和登录 shell。

处理 CSV 类文本时可以使用：

```bash
awk -F ',' '{print $1, $3}' data.csv
```

### 4. 按条件过滤行

```bash
awk '$3 > 100 {print $0}' data.txt
```

表示只输出第 3 个字段大于 100 的行。

也可以判断字符串：

```bash
awk '$1 == "ERROR" {print $0}' app.log
```

### 5. 打印行号

```bash
awk '{print NR, $0}' file.txt
```

`NR` 表示当前处理的行号。

如果只想跳过第一行表头：

```bash
awk 'NR > 1 {print $0}' data.txt
```

### 6. 打印最后一个字段

```bash
awk '{print $NF}' file.txt
```

`NF` 表示当前行字段数量，所以 `$NF` 表示最后一个字段。

这个写法在处理不固定列数的日志时很常用。

### 7. 统计字段总和

```bash
awk '{sum += $1} END {print sum}' nums.txt
```

`END` 块会在所有行处理完成后执行。

统计命令输出中的容量或数量时经常用这种写法。

### 8. 统计平均值

```bash
awk '{sum += $1} END {print sum / NR}' nums.txt
```

如果文件可能为空，建议增加判断：

```bash
awk '{sum += $1} END {if (NR > 0) print sum / NR}' nums.txt
```

### 9. 按关键字统计次数

```bash
awk '{count[$1]++} END {for (k in count) print k, count[k]}' access.log
```

这里使用了数组 `count`，可以统计第 1 列每个值出现的次数。

常见场景是统计访问 IP：

```bash
awk '{count[$1]++} END {for (ip in count) print ip, count[ip]}' access.log
```

### 10. 格式化输出

```bash
awk '{printf "%-20s %s\n", $1, $2}' file.txt
```

`printf` 可以控制输出宽度和格式，适合生成更整齐的结果。

---

## 四、常见问题与建议

### 1. `awk` 默认按什么分隔字段？

默认按连续空白字符分隔，包括空格和 Tab。

如果文本使用冒号、逗号或其他字符分隔，需要用 `-F` 指定：

```bash
awk -F ':' '{print $1}' /etc/passwd
```

### 2. `$0`、`$1`、`NF`、`NR` 分别是什么意思？

常用理解如下：

- `$0`：整行
- `$1`：第 1 个字段
- `$2`：第 2 个字段
- `NF`：当前行字段数
- `$NF`：最后一个字段
- `NR`：当前行号

这些变量是掌握 `awk` 的基础。

### 3. 如何在 shell 变量和 `awk` 之间传值？

推荐使用 `-v`：

```bash
limit=80
awk -v n="$limit" '$1 > n {print $0}' data.txt
```

不要随意拼接 shell 字符串，避免引号和特殊字符导致脚本行为异常。

### 4. 生产环境中使用 `awk` 有什么建议？

建议：

- 先用小样本验证表达式
- 明确输入分隔符
- 对表头行使用 `NR > 1` 跳过
- 统计前确认字段位置是否稳定
- 写复杂逻辑时考虑使用 `awk -f script.awk`

---

## 五、常用命令速查

```bash
awk '{print $0}' file.txt
awk '{print $1, $2}' file.txt
awk -F ':' '{print $1, $7}' /etc/passwd
awk -F ',' '{print $1, $3}' data.csv
awk '$3 > 100 {print $0}' data.txt
awk '$1 == "ERROR" {print $0}' app.log
awk '{print NR, $0}' file.txt
awk 'NR > 1 {print $0}' data.txt
awk '{print $NF}' file.txt
awk '{sum += $1} END {print sum}' nums.txt
awk '{sum += $1} END {if (NR > 0) print sum / NR}' nums.txt
awk '{count[$1]++} END {for (k in count) print k, count[k]}' access.log
```

---

## 六、总结

`awk` 是 Linux 下非常实用的字段处理和文本统计工具。建议优先掌握这几类用法：

```bash
awk '{print $1}' file
awk -F ':' '{print $1}' file
awk '$3 > 100 {print $0}' file
awk '{sum += $1} END {print sum}' file
```

日常运维中，`awk` 经常和 `grep`、`sort`、`uniq`、`head` 等命令组合使用。掌握字段提取、条件过滤和汇总统计，就能处理大部分日志分析和命令输出整理场景。