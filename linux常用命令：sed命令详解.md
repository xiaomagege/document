# linux常用命令：sed命令详解

`sed` 是 Linux 中非常常用的流式文本编辑工具，特别适合对文本进行查找、替换、删除、插入和批量修改。

在日常运维中，修改配置、处理日志、批量替换文本、从命令输出中抽取某些行，都经常会用到 `sed`。它功能强大，但直接修改文件时也需要谨慎，尤其是使用 `-i` 参数时。

---

## 一、`sed` 是什么？

`sed` 是 stream editor 的缩写，意思是流编辑器。

它主要用于：

- 替换文本内容
- 删除指定行
- 打印指定行
- 在指定位置插入或追加内容
- 批量修改配置文件
- 处理命令管道中的文本

`sed` 默认逐行读取输入，对每一行执行指定规则，然后把结果输出到屏幕。默认情况下不会修改原文件。

---

## 二、常见使用语法与重点参数

```bash
sed [选项] '脚本' 文件
命令 | sed [选项] '脚本'
```

常用参数说明：

- `-n`：安静模式，通常配合 `p` 只打印匹配内容
- `-i`：直接修改文件内容
- `-e`：指定多个编辑命令
- `-f 文件`：从脚本文件读取 sed 命令
- `-r` 或 `-E`：使用扩展正则表达式，具体支持和系统版本有关

常见动作：

- `s/旧/新/`：替换每行第一个匹配
- `s/旧/新/g`：替换每行所有匹配
- `p`：打印
- `d`：删除
- `a`：在匹配行后追加
- `i`：在匹配行前插入
- `c`：替换整行

---

## 三、常用命令与使用场景

### 1. 替换每行第一个匹配内容

```bash
sed 's/foo/bar/' file.txt
```

会把每行第一个 `foo` 替换为 `bar`，结果输出到屏幕，不修改原文件。

### 2. 替换每行所有匹配内容

```bash
sed 's/foo/bar/g' file.txt
```

`g` 表示 global，会替换每行中所有 `foo`。

### 3. 直接修改文件

```bash
sed -i 's/foo/bar/g' file.txt
```

`-i` 会直接修改文件内容。

生产环境修改配置文件前建议先备份：

```bash
cp -a app.conf app.conf.bak.$(date +%F)
sed -i 's/old_value/new_value/g' app.conf
```

### 4. 打印指定行

```bash
sed -n '5p' file.txt
```

只打印第 5 行。

打印第 5 到第 10 行：

```bash
sed -n '5,10p' file.txt
```

### 5. 删除指定行

```bash
sed '5d' file.txt
```

删除第 5 行后输出结果，但不会修改原文件。

删除空行：

```bash
sed '/^$/d' file.txt
```

直接删除文件中的空行：

```bash
sed -i '/^$/d' file.txt
```

### 6. 删除匹配关键字的行

```bash
sed '/DEBUG/d' app.log
```

会删除包含 `DEBUG` 的行后输出。

这适合临时过滤日志内容。

### 7. 只打印匹配行

```bash
sed -n '/ERROR/p' app.log
```

只打印包含 `ERROR` 的行。

这个场景也可以使用 `grep ERROR app.log`，但 `sed` 可以继续和范围、替换等动作组合。

### 8. 在匹配行后追加内容

```bash
sed '/server_name/a\    access_log /var/log/nginx/access.log;' nginx.conf
```

`a` 表示 append，会在匹配行后追加新内容。

注意反斜杠和缩进在不同 shell 中可能需要额外转义。

### 9. 在匹配行前插入内容

```bash
sed '/server_name/i\    # nginx server config' nginx.conf
```

`i` 表示 insert，会在匹配行前插入内容。

### 10. 替换整行

```bash
sed '/^port=/c\port=8080' app.conf
```

`c` 表示 change，会把匹配行整体替换为新内容。

### 11. 使用多个规则

```bash
sed -e 's/foo/bar/g' -e '/DEBUG/d' app.log
```

可以同时执行替换和删除。

---

## 四、常见问题与建议

### 1. `sed` 默认会修改文件吗？

不会。默认只把处理结果输出到屏幕。

只有使用 `-i` 时才会直接修改文件：

```bash
sed -i 's/foo/bar/g' file.txt
```

### 2. 使用 `sed -i` 有什么风险？

`sed -i` 会原地修改文件，规则写错可能批量改坏配置或数据。

建议：

- 先不加 `-i` 预览输出
- 修改配置前先备份
- 使用更精确的匹配条件
- 避免对重要文件直接全局替换

### 3. 替换内容中包含 `/` 怎么办？

可以换用其他分隔符：

```bash
sed 's#/data/old#/data/new#g' app.conf
```

`sed` 的替换分隔符不一定必须是 `/`，使用 `#`、`|` 等都可以。

### 4. `sed` 和 `awk` 怎么选？

简单理解：

- 替换、删除、按行编辑：优先考虑 `sed`
- 按列提取、计算、统计：优先考虑 `awk`
- 只按关键字过滤行：优先考虑 `grep`

---

## 五、常用命令速查

```bash
sed 's/foo/bar/' file.txt
sed 's/foo/bar/g' file.txt
sed -i 's/foo/bar/g' file.txt
sed -n '5p' file.txt
sed -n '5,10p' file.txt
sed '5d' file.txt
sed '/^$/d' file.txt
sed -i '/^$/d' file.txt
sed '/DEBUG/d' app.log
sed -n '/ERROR/p' app.log
sed '/server_name/a\    access_log /var/log/nginx/access.log;' nginx.conf
sed '/server_name/i\    # nginx server config' nginx.conf
sed '/^port=/c\port=8080' app.conf
sed -e 's/foo/bar/g' -e '/DEBUG/d' app.log
sed 's#/data/old#/data/new#g' app.conf
```

---

## 六、总结

`sed` 是 Linux 中非常实用的流式文本编辑工具。建议优先掌握：

```bash
sed 's/old/new/g' file
sed -n '1,10p' file
sed '/pattern/d' file
sed -i 's/old/new/g' file
```

日常使用中，`sed` 最常见的风险来自 `-i` 原地修改。线上修改配置前，先备份、再预览、最后执行修改，是更稳妥的习惯。