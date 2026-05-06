# linux常用命令：du命令详解

当服务器磁盘空间不足时，`df` 可以告诉我们哪个挂载点快满了，但真正要定位“到底是谁占了空间”，通常就要使用 `du`。

`du` 是 Linux 中查看目录和文件磁盘占用的常用命令。它非常适合排查大目录、大文件、日志堆积、缓存膨胀和备份文件过多等问题。

---

## 一、`du` 是什么？

`du` 是 disk usage 的缩写，用于统计文件或目录占用的磁盘空间。

它主要用于：

- 查看某个目录占用了多少磁盘空间
- 找出当前目录下哪些子目录最大
- 排查日志、缓存、备份文件占用过多的问题
- 和 `df` 配合定位磁盘空间不足
- 在清理文件前判断清理收益

需要注意的是：

- `du` 默认会递归统计目录
- `du` 统计的是目录内文件占用空间，不等同于文件系统整体使用量
- 对大目录执行 `du` 可能比较慢，生产环境要注意执行范围

---

## 二、常见使用语法与重点参数

```bash
du [选项] [文件或目录]
```

常用参数说明：

- `-h`：以人类可读方式显示容量，例如 `K`、`M`、`G`
- `-s`：只显示汇总结果
- `-a`：显示文件和目录的占用情况
- `-c`：显示总计
- `-d N`：限制统计目录深度
- `--max-depth=N`：限制统计目录深度，和 `-d` 类似
- `--exclude=模式`：排除指定文件或目录
- `-x`：只统计当前文件系统，不跨挂载点

常见示例：

```bash
du -sh /var/log
du -h --max-depth=1 /data
du -ah /var/log | sort -h
du -sh *
du -xh --max-depth=1 /
```

---

## 三、常用命令与使用场景

### 1. 查看目录总大小

```bash
du -sh /var/log
```

`-s` 表示汇总，`-h` 表示人类可读。

这是查看单个目录总占用最常见的写法，适合快速判断日志目录、数据目录或项目目录是否过大。

### 2. 查看当前目录下每个文件和目录大小

```bash
du -sh *
```

会列出当前目录下每个非隐藏文件和目录的大小。

如果需要包含隐藏目录，可以结合 shell 通配符或直接统计上级目录：

```bash
du -h --max-depth=1 .
```

### 3. 查看一级子目录大小

```bash
du -h --max-depth=1 /data
```

这个命令非常适合定位大目录。

例如 `/data` 空间占用高，可以先看一级目录：

```bash
du -h --max-depth=1 /data | sort -h
```

再进入最大的目录继续排查。

### 4. 按大小排序查看目录

```bash
du -h --max-depth=1 /var | sort -h
```

`sort -h` 可以按人类可读容量排序。

如果想让最大的目录显示在最前面：

```bash
du -h --max-depth=1 /var | sort -hr
```

### 5. 查找大文件和大目录

```bash
du -ah /var/log | sort -hr | head -20
```

`-a` 会同时显示文件和目录，适合快速找出最大的日志文件或压缩包。

对非常大的目录执行该命令可能较慢，建议先用 `--max-depth=1` 缩小范围。

### 6. 只统计当前文件系统

```bash
du -xh --max-depth=1 /
```

`-x` 表示不跨越文件系统边界。

这在根目录 `/` 下排查空间时很有用，可以避免把 `/data`、`/mnt` 等单独挂载的磁盘也统计进去。

### 7. 排除指定目录

```bash
du -h --max-depth=1 /data --exclude='cache'
```

可以排除名称匹配的目录或文件。

例如统计项目目录时排除依赖目录：

```bash
du -h --max-depth=1 . --exclude='node_modules'
```

### 8. 查看多个目录总大小

```bash
du -sh /var/log /data /backup
```

适合同时比较多个关键目录的空间占用。

如果想显示总计：

```bash
du -sch /var/log /data /backup
```

### 9. 结合 `find` 查找大文件

```bash
find /var/log -type f -size +100M -exec du -h {} +
```

可以找出 `/var/log` 下大于 100M 的文件，并显示大小。

如果想按大小排序：

```bash
find /var/log -type f -size +100M -exec du -h {} + | sort -hr
```

---

## 四、常见问题与建议

### 1. `du` 执行很慢怎么办？

`du` 会遍历目录树，大目录、大量小文件或网络挂载都会导致执行较慢。

建议：

- 先用 `df -h` 确认问题分区
- 再用 `du -h --max-depth=1` 从大范围逐层缩小
- 避免一开始就对 `/` 执行深度递归
- 对根目录排查时加 `-x`，避免跨文件系统

### 2. `du` 和 `df` 结果不一致怎么办？

常见原因包括：

- 文件被删除但进程仍然占用
- 目录下有挂载点
- 权限不足导致部分目录统计失败
- 文件系统预留空间或元数据占用

可以使用：

```bash
lsof | grep deleted
```

排查已删除但仍被进程打开的文件。

### 3. 为什么 `du -sh *` 看不到隐藏目录？

因为 `*` 默认不匹配以 `.` 开头的隐藏文件或目录。

可以用：

```bash
du -h --max-depth=1 .
```

这样会统计当前目录本身下一层内容，包含隐藏目录。

### 4. 生产环境清理前应该注意什么？

建议先看再删：

```bash
du -h --max-depth=1 /var | sort -hr
```

确认目录来源后再处理。不要仅凭文件大就直接删除，尤其是数据库文件、业务上传文件、正在写入的日志和容器运行目录。

---

## 五、常用命令速查

```bash
du -sh /var/log
du -sh *
du -h --max-depth=1 .
du -h --max-depth=1 /data
du -h --max-depth=1 /data | sort -h
du -h --max-depth=1 /var | sort -hr
du -ah /var/log | sort -hr | head -20
du -xh --max-depth=1 /
du -h --max-depth=1 . --exclude='node_modules'
du -sch /var/log /data /backup
find /var/log -type f -size +100M -exec du -h {} + | sort -hr
```

---

## 六、总结

`du` 是定位目录和文件空间占用的核心命令。建议重点掌握：

```bash
du -sh /path
du -h --max-depth=1 /path
du -ah /path | sort -hr | head
```

排查磁盘空间问题时，可以先用 `df -h` 判断哪个挂载点空间不足，再用 `du` 从对应目录逐层定位大目录或大文件。这样比盲目清理文件更准确，也更安全。