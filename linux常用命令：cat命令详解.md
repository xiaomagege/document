# linux常用命令：cat命令详解

在 Linux 日常使用和运维中，`cat` 是最基础的文件查看命令之一。无论是快速查看文本内容、拼接多个文件、输出配置文件，还是配合重定向创建文件，`cat` 都非常常用。

---

## 一、`cat` 是什么？

`cat` 是 concatenate 的缩写，用于查看、拼接并输出文件内容。

它主要用于：

- 快速查看文本文件内容
- 将多个文件内容连续输出
- 配合重定向写入文件
- 查看带行号的文本内容

说明：

- `cat` 适合查看内容较短的文件
- 大文件或需要翻页查看时，更适合用 `less` 或 `more`

---

## 二、常见使用语法与重点参数

```bash
cat [选项] [文件]
```

重点参数说明

- `-n`：显示行号
- `-b`：只对非空行显示行号
- `-s`：压缩连续空行为一行
- `-A`：显示所有不可见字符
- `-E`：在每行结尾显示 `$`
- `-T`：将 Tab 显示为 `^I`

---

## 三、常用命令与使用场景

### 1. 查看文件内容

```bash
cat /etc/passwd
```

这是最基础的用法，适合快速输出整个文件内容。

### 2. 同时查看多个文件

```bash
cat file1.txt file2.txt
```

会按顺序连续输出多个文件的内容。

### 3. 显示行号

```bash
cat -n app.log
```

适合排查配置文件、日志或脚本时快速定位内容所在行。

### 4. 只给非空行编号

```bash
cat -b file.txt
```

适合文本里空行较多，但你只关心有效内容编号的场景。

### 5. 压缩连续空行

```bash
cat -s file.txt
```

当文件中存在大量连续空行时，这样输出会更紧凑。

### 6. 查看不可见字符

```bash
cat -A file.txt
```

适合排查换行符、Tab、隐藏字符等格式问题。

### 7. 拼接文件生成新文件

```bash
cat part1.txt part2.txt > all.txt
```

适合把多个文本文件合并成一个文件。

### 8. 使用重定向创建文件

```bash
cat > hello.txt
```

输入内容后按 `Ctrl+D` 结束，可直接创建一个新文件。

### 9. 追加内容到文件末尾

```bash
cat >> hello.txt
```

适合手工往文件末尾追加少量内容。

---

## 四、常见问题与建议

### 1. `cat` 适合查看大文件吗？

不适合。`cat` 会直接一次性输出全部内容，大文件容易刷屏。查看大文件更适合使用：

```bash
less file.log
```

或：

```bash
tail -n 100 file.log
```

### 2. `cat -n` 和 `nl` 有什么区别？

两者都能显示行号，但 `cat -n` 更直接，适合快速查看；`nl` 在格式控制上更灵活。

### 3. 为什么排查文件格式问题时常用 `cat -A`？

因为它可以把不可见字符显示出来，便于发现多余空格、Tab、Windows 换行符等问题。

### 4. 为什么脚本里不建议滥用 `cat`？

有些场景可以直接由命令本身读取文件，例如 `grep pattern file.txt`，不必先写成 `cat file.txt | grep pattern`。后者通常多了一层无必要的管道。

### 5. 实际工作里最常用的是哪几个组合？

- `cat file`
- `cat -n file`
- `cat -A file`
- `cat file1 file2 > newfile`

---

## 五、常用命令速查

```bash
cat file.txt
cat file1.txt file2.txt
cat -n file.txt
cat -b file.txt
cat -s file.txt
cat -A file.txt
cat part1.txt part2.txt > all.txt
cat > newfile.txt
cat >> newfile.txt
```

---

## 六、总结

`cat` 是 Linux 下最基础的文件查看命令之一。建议优先熟练掌握以下几条：

```bash
cat file.txt
cat -n file.txt
cat -A file.txt
cat file1.txt file2.txt > all.txt
cat > newfile.txt
```

掌握这些用法后，大多数“快速查看文本、查看行号、排查隐藏字符、拼接文件、临时写入文件”的场景都能快速处理。
