# 22｜文本处理与批量处理｜日志去重统计时，uniq 应该怎么配合 sort

`uniq` 最容易让人误会的地方，是很多人以为它会“全局去重”。

其实它只处理相邻重复行，所以它在日志和文本统计里真正好用的时候，往往都不是单独上，而是和 `sort` 一起配合。

如果只想先记 3 条：

- `uniq` 只处理相邻重复行
- 想全局去重，通常先 `sort`
- 想做次数统计，最常见的是 `sort | uniq -c | sort -nr`

## 一、什么情况下先用 `uniq`

`uniq` 最适合这些场景：

- 想去掉重复行
- 想统计每个值出现了多少次
- 想看哪些值重复最多

最常见的起手式通常是：

```bash
sort file.txt | uniq
sort file.txt | uniq -c
```

## 二、先记住这几条命令

```bash
sort file.txt | uniq
sort file.txt | uniq -c
sort file.txt | uniq -d
sort file.txt | uniq -u
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

它们分别适合：

- 排序后去重
- 排序后统计次数
- 只看重复项
- 只看只出现一次的项
- 做访问排行

## 三、为什么它几乎总和 `sort` 搭配

如果文件内容是：

```text
a
a
b
a
```

直接：

```bash
uniq file.txt
```

结果还会保留最后那个 `a`，因为它和前面的 `a` 并不相邻。

所以你真正想全局去重时，通常要先：

```bash
sort file.txt | uniq
```

## 四、日志统计里最常见的几种用法

### 1. 看唯一列表

```bash
sort file.txt | uniq
```

### 2. 看每个值出现多少次

```bash
sort file.txt | uniq -c
```

### 3. 看谁出现得最多

```bash
sort file.txt | uniq -c | sort -nr
```

这在统计访问 IP、状态码、接口路径时都非常常见。

### 4. 只看重复项

```bash
sort file.txt | uniq -d
```

适合快速找“重复值有哪些”。

## 五、什么时候直接用 `sort -u` 就够了

如果你只是想要一个去重后的唯一列表，不关心次数，也不关心重复项本身，那么：

```bash
sort -u file.txt
```

通常就够了。

但只要你想：

- 看次数
- 只看重复
- 只看唯一

就更适合用 `uniq -c`、`uniq -d`、`uniq -u` 这套。

## 六、最常见的误区

### 1. 不排序就直接 `uniq`

这通常只能去掉相邻重复，不是很多人脑子里想要的“全局去重”。

### 2. 统计次数后不再排序

```bash
sort file.txt | uniq -c
```

能看到次数，但如果你真正想看排行，通常还要再接一段：

```bash
sort file.txt | uniq -c | sort -nr
```

### 3. 把 `uniq` 当成所有去重场景的唯一解

如果只是简单唯一列表，`sort -u` 可能更直接。

## 七、结论

`uniq` 最值钱的地方，不是“去重”这两个字本身，而是它和 `sort` 组合后能快速做统计和排行。

记住“只处理相邻重复行”这条，基本就抓住了它的核心。
