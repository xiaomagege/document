# Linux网络与远程运维命令：rsync命令详解

`rsync` 是 Linux 中非常强大的文件同步工具。它既可以本地同步目录，也可以通过 SSH 在服务器之间同步文件，并且支持增量传输、删除同步、排除规则、限速等功能。

相比 `scp`，`rsync` 更适合大量文件、重复同步、部署目录、备份任务和跨服务器增量传输。

---

## 一、`rsync` 是什么？

`rsync` 用于高效同步文件和目录。

它主要用于：

- 本地目录同步
- 本机和远程服务器之间同步
- 增量传输文件
- 备份目录
- 部署静态文件或应用目录
- 排除不需要同步的文件
- 删除目标端多余文件以保持一致

`rsync` 的核心优势是增量同步。重复执行时，只传输发生变化的内容，效率比全量复制更高。

---

## 二、常见使用语法与重点参数

```bash
rsync [选项] 源 目标
```

常用参数说明：

- `-a`：归档模式，保留权限、时间、符号链接等属性
- `-v`：显示详细输出
- `-z`：传输时压缩
- `-h`：人类可读输出
- `-P`：显示进度并支持断点续传，等价于 `--partial --progress`
- `--delete`：删除目标端源中不存在的文件
- `--exclude=模式`：排除指定文件或目录
- `-n` / `--dry-run`：演练，不真正执行
- `-e`：指定远程 shell，例如 SSH 端口

常见示例：

```bash
rsync -av source/ target/
rsync -avz source/ user@example.com:/data/app/
rsync -avz --delete source/ user@example.com:/data/app/
rsync -avzn --delete source/ user@example.com:/data/app/
```

---

## 三、常用命令与使用场景

### 1. 本地同步目录

```bash
rsync -av source/ target/
```

把 `source/` 目录里的内容同步到 `target/`。

注意源路径末尾的 `/` 很重要：

- `source/`：同步目录里面的内容
- `source`：同步目录本身

### 2. 同步目录本身

```bash
rsync -av source target/
```

结果通常是 `target/source/...`。

这和 `rsync -av source/ target/` 不一样。

### 3. 同步到远程服务器

```bash
rsync -avz source/ user@example.com:/data/app/
```

通过 SSH 把本地目录同步到远程服务器。

`-z` 表示传输压缩，适合网络传输。

### 4. 从远程服务器同步到本地

```bash
rsync -avz user@example.com:/var/log/app/ ./app-log/
```

适合下载日志、备份远程目录。

### 5. 显示进度和支持断点

```bash
rsync -avzP source/ user@example.com:/data/app/
```

`-P` 会显示传输进度，并保留部分传输文件，便于断点续传。

### 6. 删除目标端多余文件

```bash
rsync -avz --delete source/ user@example.com:/data/app/
```

会让目标目录和源目录保持一致。

风险很高：目标端存在但源端不存在的文件会被删除。

执行前建议先演练：

```bash
rsync -avzn --delete source/ user@example.com:/data/app/
```

### 7. 排除指定文件或目录

```bash
rsync -avz --exclude='*.log' --exclude='cache/' source/ user@example.com:/data/app/
```

可以排除日志、缓存、临时文件等不需要同步的内容。

### 8. 指定 SSH 端口

```bash
rsync -avz -e 'ssh -p 2222' source/ user@example.com:/data/app/
```

如果 SSH 不是默认 22 端口，需要通过 `-e` 指定。

### 9. 只演练不执行

```bash
rsync -avzn source/ user@example.com:/data/app/
```

`-n` 表示 dry-run，只显示将要同步的内容，不真正修改目标端。

涉及 `--delete` 时，强烈建议先 dry-run。

### 10. 限制带宽

```bash
rsync -avz --bwlimit=1024 source/ user@example.com:/data/app/
```

限制传输速度，单位通常是 KB/s。

适合生产环境中避免同步任务占满带宽。

---

## 四、常见问题与建议

### 1. 源路径后面的 `/` 有什么区别？

这是 `rsync` 最容易踩坑的地方。

```bash
rsync -av source/ target/
```

同步 `source` 目录里的内容到 `target/`。

```bash
rsync -av source target/
```

同步 `source` 目录本身到 `target/`，结果是 `target/source/`。

### 2. `--delete` 有什么风险？

`--delete` 会删除目标端多余文件。

如果源路径写错、目标路径写错，可能造成目标目录数据被大量删除。

执行前先用：

```bash
rsync -avzn --delete source/ target/
```

确认输出后再真正执行。

### 3. `rsync` 和 `scp` 怎么选？

简单理解：

- 临时复制少量文件：`scp`
- 大量文件、重复同步、部署目录：`rsync`
- 需要增量、排除、删除同步、进度：`rsync`

### 4. 生产环境使用 `rsync` 有什么建议？

建议：

- 涉及删除时先 `--dry-run`
- 明确源路径是否带 `/`
- 对关键目录先备份
- 使用 `--exclude` 排除缓存、日志、临时文件
- 大文件或跨网络同步使用 `-P`
- 必要时用 `--bwlimit` 限速

---

## 五、常用命令速查

```bash
rsync -av source/ target/
rsync -av source target/
rsync -avz source/ user@example.com:/data/app/
rsync -avz user@example.com:/var/log/app/ ./app-log/
rsync -avzP source/ user@example.com:/data/app/
rsync -avz --delete source/ user@example.com:/data/app/
rsync -avzn --delete source/ user@example.com:/data/app/
rsync -avz --exclude='*.log' --exclude='cache/' source/ user@example.com:/data/app/
rsync -avz -e 'ssh -p 2222' source/ user@example.com:/data/app/
rsync -avzn source/ user@example.com:/data/app/
rsync -avz --bwlimit=1024 source/ user@example.com:/data/app/
```

---

## 六、总结

`rsync` 是 Linux 中非常适合备份、部署和增量同步的工具。建议优先掌握：

```bash
rsync -av source/ target/
rsync -avz source/ user@host:/path/
rsync -avzP source/ user@host:/path/
rsync -avzn --delete source/ user@host:/path/
```

使用 `rsync` 时，最重要的是理解源路径末尾 `/` 的含义，以及谨慎使用 `--delete`。只要先 dry-run、确认路径，再执行真实同步，就能显著降低误操作风险。