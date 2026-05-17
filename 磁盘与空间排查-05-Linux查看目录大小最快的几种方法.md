# 磁盘与空间排查05：Linux 查看目录大小，最快的几种方法

排查磁盘空间时，真正高频的问题通常不是“怎么查看大小”，而是“怎么最快找到最占空间的那一层”。

这篇文章更适合作为速查：你知道要看大小，但不想在 `du`、`ls`、`df` 之间绕来绕去。

如果只想先记 3 条：

- 看目录总大小：`du -sh /path`
- 看下一层谁最大：`du -h --max-depth=1 /path | sort -hr`
- 看整体分区是否满了：`df -h /path`

## 一、先判断你到底想看什么

不同问题对应不同命令：

| 你想知道什么 | 优先命令 |
| --- | --- |
| 某个目录总共多大 | `du -sh /path` |
| 某个目录下一层谁最大 | `du -h --max-depth=1 /path | sort -hr` |
| 某个文件多大 | `ls -lh file` |
| 当前目录文件按大小排序 | `ls -lhS` |
| 当前路径所在分区还剩多少 | `df -h /path` |
| 查找超过 1G 的文件 | `find /path -type f -size +1G` |

先把问题问清楚，命令就不会选错。

## 二、最快定位大目录

看目录总大小：

```bash
du -sh /var/log
```

看下一层目录大小并排序：

```bash
du -h --max-depth=1 /var | sort -hr
```

这条最适合排查“磁盘满了到底是谁占的”。

比如 `/var` 很大，就继续：

```bash
du -h --max-depth=1 /var/log | sort -hr
```

按层级往下走，通常比一上来全盘扫描更快、更稳。

## 三、最快定位大文件

当前目录按文件大小排序：

```bash
ls -lhS
```

找某个目录下超过 1G 的文件：

```bash
find /data -type f -size +1G
```

同时显示大小并排序：

```bash
find /data -type f -size +1G -exec du -h {} + | sort -hr
```

如果目录特别大，先缩小路径范围，再执行 `find`。

## 四、先看分区还是先看目录

如果你还不知道哪里满了，先看分区：

```bash
df -h
```

如果你已经知道是 `/var`、`/data` 或某个业务目录异常，再看目录：

```bash
du -h --max-depth=1 /data | sort -hr
```

推荐顺序：

1. `df -h` 找到问题挂载点
2. `du -h --max-depth=1 /path | sort -hr` 找一级大目录
3. 继续进入最大目录逐层缩小
4. 必要时用 `find` 找超大文件

## 五、常见误区

### 1. 用 `ls -lh` 看目录大小

`ls -lh` 看到的目录大小不是目录下所有文件的总大小。

看目录总占用应该用：

```bash
du -sh /path
```

### 2. `du -sh *` 看不到隐藏目录

`*` 默认不匹配以 `.` 开头的隐藏目录。

更稳的方式是：

```bash
du -h --max-depth=1 .
```

### 3. 从 `/` 开始全量递归

这可能非常慢，还可能跨到其他挂载点。

排查根分区时更推荐：

```bash
du -xh --max-depth=1 /
```

### 4. 看到大文件就直接删

删除前要确认它是不是：

- 数据库文件
- 正在写入的日志
- 用户上传文件
- 容器运行数据
- 备份或归档文件

## 六、常用命令速查

```bash
df -h
df -h /path
du -sh /path
du -h --max-depth=1 /path | sort -hr
du -xh --max-depth=1 /
ls -lh file.log
ls -lhS
find /data -type f -size +1G
find /data -type f -size +1G -exec du -h {} + | sort -hr
```

## 七、结论

查看目录大小时，别把 `df`、`du`、`ls` 混成一个问题。

`df` 看分区，`du` 看目录，`ls` 看文件。按这个分工走，定位空间问题会快很多。

