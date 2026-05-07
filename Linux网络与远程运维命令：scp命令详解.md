# Linux网络与远程运维命令：scp命令详解

`scp` 是 Linux 中基于 SSH 的文件复制命令，可以在本机和远程服务器之间安全传输文件或目录。

它语法简单，适合临时复制文件、上传安装包、下载日志、在两台服务器之间传输少量文件。对于大量文件或增量同步，通常更推荐 `rsync`。

---

## 一、`scp` 是什么？

`scp` 是 secure copy 的缩写，用于通过 SSH 加密通道复制文件。

它主要用于：

- 从本机上传文件到远程服务器
- 从远程服务器下载文件到本机
- 复制目录
- 指定 SSH 端口传输文件
- 指定私钥传输文件
- 在两台远程服务器之间复制文件

基本格式：

```bash
scp 源路径 目标路径
```

远程路径格式：

```bash
user@host:/path/to/file
```

---

## 二、常见使用语法与重点参数

```bash
scp [选项] 源 目标
```

常用参数说明：

- `-r`：递归复制目录
- `-P 端口`：指定 SSH 端口，注意是大写 `P`
- `-i 私钥文件`：指定私钥
- `-p`：保留文件修改时间和权限
- `-C`：启用压缩
- `-v`：显示详细过程
- `-l 限速`：限制传输带宽，单位通常是 Kbit/s

常见示例：

```bash
scp app.tar.gz user@example.com:/tmp/
scp user@example.com:/var/log/app.log ./
scp -r ./dist user@example.com:/data/app/
scp -P 2222 app.tar.gz user@example.com:/tmp/
```

---

## 三、常用命令与使用场景

### 1. 上传文件到远程服务器

```bash
scp app.tar.gz user@example.com:/tmp/
```

把本地 `app.tar.gz` 上传到远程服务器 `/tmp/` 目录。

### 2. 从远程服务器下载文件

```bash
scp user@example.com:/var/log/app.log ./
```

把远程服务器上的日志文件下载到当前目录。

### 3. 上传并改名

```bash
scp app.tar.gz user@example.com:/tmp/app-20260506.tar.gz
```

目标路径写具体文件名时，可以在上传时改名。

### 4. 复制目录

```bash
scp -r ./dist user@example.com:/data/app/
```

`-r` 表示递归复制目录。

如果目录较大、文件很多，建议考虑 `rsync`。

### 5. 指定 SSH 端口

```bash
scp -P 2222 app.tar.gz user@example.com:/tmp/
```

注意 `scp` 指定端口是大写 `-P`，而 `ssh` 是小写 `-p`。

这是很多人容易混淆的地方。

### 6. 指定私钥

```bash
scp -i ~/.ssh/id_rsa app.tar.gz user@example.com:/tmp/
```

适合使用密钥认证的服务器。

### 7. 保留时间和权限

```bash
scp -p app.conf user@example.com:/tmp/
```

`-p` 会尽量保留修改时间、访问时间和权限。

### 8. 启用压缩

```bash
scp -C large.log user@example.com:/tmp/
```

传输文本文件或日志时可能有帮助。

对于已经压缩过的文件，例如 `.zip`、`.gz`，效果通常不明显。

### 9. 远程服务器之间复制文件

```bash
scp user1@host1:/tmp/a.log user2@host2:/tmp/
```

这类操作依赖 SSH 配置和网络连通性。生产环境中如果复杂，通常建议先下载到本机或使用 `rsync`/对象存储等方式。

---

## 四、常见问题与建议

### 1. `scp -P` 和 `ssh -p` 为什么大小写不同？

`scp` 指定端口使用大写：

```bash
scp -P 2222 file user@host:/tmp/
```

`ssh` 指定端口使用小写：

```bash
ssh -p 2222 user@host
```

要特别注意这个差异。

### 2. 传输目录为什么失败？

复制目录需要加 `-r`：

```bash
scp -r dir user@host:/tmp/
```

### 3. 文件很多时为什么不推荐 `scp`？

`scp` 适合简单复制，但对大量小文件、断点续传、增量同步支持不如 `rsync`。

大量文件同步建议使用：

```bash
rsync -avz source/ user@host:/path/
```

### 4. 生产环境使用 `scp` 有什么建议？

建议：

- 上传前确认目标路径
- 覆盖重要文件前先备份
- 目录传输使用 `-r`，大量文件用 `rsync`
- 指定端口时注意大写 `-P`
- 使用密钥时保护好私钥权限

---

## 五、常用命令速查

```bash
scp app.tar.gz user@example.com:/tmp/
scp user@example.com:/var/log/app.log ./
scp app.tar.gz user@example.com:/tmp/app-20260506.tar.gz
scp -r ./dist user@example.com:/data/app/
scp -P 2222 app.tar.gz user@example.com:/tmp/
scp -i ~/.ssh/id_rsa app.tar.gz user@example.com:/tmp/
scp -p app.conf user@example.com:/tmp/
scp -C large.log user@example.com:/tmp/
scp user1@host1:/tmp/a.log user2@host2:/tmp/
rsync -avz source/ user@host:/path/
```

---

## 六、总结

`scp` 是基于 SSH 的安全文件复制命令，适合临时上传、下载少量文件或目录。建议优先掌握：

```bash
scp file user@host:/path/
scp user@host:/path/file ./
scp -r dir user@host:/path/
scp -P port file user@host:/path/
scp -i key file user@host:/path/
```

简单文件传输用 `scp` 很方便；如果是大量文件、增量同步、需要保留更多属性或支持断点能力，优先考虑 `rsync`。