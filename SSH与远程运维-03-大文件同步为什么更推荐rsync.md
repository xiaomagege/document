# SSH与远程运维03：大文件同步为什么更推荐 rsync

`scp` 适合临时拷一份，`rsync` 更适合持续同步。

真正让 `rsync` 值得长期掌握的，不是“它能传文件”，而是：

- 它支持增量
- 它适合大目录
- 它能演练
- 它能删除同步
- 它对部署、备份、回传都很好用

如果只想先记 3 条：

- 重复同步目录，优先用 `rsync`
- 涉及 `--delete` 时先 `--dry-run`
- 一定分清源路径末尾 `/` 的区别

## 一、什么时候优先用 `rsync`

这些场景都更适合 `rsync`：

- 大量文件同步
- 重复部署目录
- 备份目录
- 从远程拉回整批日志
- 想保留权限、时间戳、软链接等属性

一句话记：

**目录同步、重复同步、增量同步，就先想 `rsync`。**

## 二、最常用的几条命令

```bash
rsync -av source/ target/
rsync -avz source/ user@example.com:/data/app/
rsync -avzP source/ user@example.com:/data/app/
rsync -avzn --delete source/ user@example.com:/data/app/
rsync -avz -e 'ssh -p 2222' source/ user@example.com:/data/app/
```

它们分别适合：

- `-a`：保留常见属性
- `-v`：看详细输出
- `-z`：传输压缩
- `-P`：带进度，支持中断恢复
- `-n`：演练，不真正执行
- `--delete`：让目标和源保持一致
- `-e`：指定 SSH 端口

## 三、最该先弄明白的一个坑：源路径后面的 `/`

这是 `rsync` 最容易出事故的地方。

```bash
rsync -av source/ target/
```

表示同步 `source` 目录里的内容。

```bash
rsync -av source target/
```

表示同步 `source` 这个目录本身，结果通常会变成：

```text
target/source/
```

这个差异必须非常熟。

## 四、最常见的 4 个场景

### 1. 本地目录同步

```bash
rsync -av source/ target/
```

适合：

- 本机备份
- 构建目录复制
- 目录归档前整理

### 2. 同步到远程机器

```bash
rsync -avz source/ user@example.com:/data/app/
```

适合部署、备份、批量上传。

### 3. 带进度和断点恢复

```bash
rsync -avzP source/ user@example.com:/data/app/
```

大文件或跨网络传输时，这一组通常很实用。

### 4. 删除同步前先演练

```bash
rsync -avzn --delete source/ user@example.com:/data/app/
```

这条命令特别重要。

涉及 `--delete` 时，如果不先演练，很容易把目标端不该删的内容也一起抹掉。

## 五、`rsync` 和 `scp` 怎么选

- 临时传一个文件：`scp`
- 目录部署、备份、重复同步：`rsync`
- 需要增量、排除、删除同步、进度：`rsync`

如果你已经开始在想：

- “能不能只传变化部分”
- “能不能先演练”
- “能不能把目标多余文件也清掉”

那基本就是 `rsync` 的场景了。

## 六、最常见的误区

### 1. 不理解路径末尾 `/`

这是最容易让目标目录结构变样的地方。

### 2. `--delete` 不先演练

更稳的顺序永远是：

```bash
rsync -avzn --delete ...
```

确认输出没问题，再去掉 `-n`。

### 3. 大网络同步不看进度

像 `-P` 这种参数，看起来只是体验问题，实际能明显降低排查传输问题时的焦虑感。

## 七、结论

`rsync` 不是“更复杂的 `scp`”，它是更适合目录同步、增量传输和部署备份的工具。

只要把“路径末尾 `/`”和“`--delete` 先演练”这两件事记牢，它会非常稳。

