# SSH与远程运维02：Linux 远程拷贝文件，scp 最常用的几种写法

`scp` 的价值非常朴素：临时传个文件，直接、顺手、够快。

它不是最强的同步工具，但在“现在就要把这个文件拷过去”这种场景里，几乎总能派上用场。

如果只想先记 3 条：

- 上传文件：`scp file user@host:/path/`
- 下载文件：`scp user@host:/path/file ./`
- 指定端口时注意 `scp` 用的是大写 `-P`

## 一、什么时候优先用 `scp`

这些场景最适合 `scp`：

- 上传安装包、脚本、配置文件
- 下载一份日志或导出文件
- 临时复制一个小目录
- 远程机器已经能 SSH，但你不想搭更复杂的同步流程

一句话记：

**少量、临时、直接传，就先想 `scp`。**

## 二、最常用的几条命令

```bash
scp app.tar.gz user@example.com:/tmp/
scp user@example.com:/var/log/app.log ./
scp -r ./dist user@example.com:/data/app/
scp -P 2222 app.tar.gz user@example.com:/tmp/
scp -i ~/.ssh/id_rsa app.tar.gz user@example.com:/tmp/
```

它们分别适合：

- 上传文件
- 下载文件
- 递归复制目录
- 指定 SSH 端口
- 指定私钥

## 三、最常见的 4 个场景

### 1. 上传文件到远程机器

```bash
scp app.tar.gz user@example.com:/tmp/
```

这是最常见的 `scp` 用法。

### 2. 从远程机器拉日志回来

```bash
scp user@example.com:/var/log/app.log ./
```

适合：

- 下载一份错误日志
- 拉回导出结果
- 临时取证

### 3. 复制目录

```bash
scp -r ./dist user@example.com:/data/app/
```

如果忘了 `-r`，目录复制通常会失败。

### 4. 非默认端口或指定私钥

```bash
scp -P 2222 -i ~/.ssh/id_rsa app.tar.gz user@example.com:/tmp/
```

这里最容易踩的点是：

- `ssh` 端口参数是小写 `-p`
- `scp` 端口参数是大写 `-P`

## 四、`scp` 和 `rsync` 怎么选

你可以直接这样记：

- 临时传一两个文件：`scp`
- 大量文件、重复同步、增量同步：`rsync`

`scp` 更像“快递一趟”，`rsync` 更像“持续同步”。

## 五, 最常见的误区

### 1. 目录复制忘了加 `-r`

这几乎是最常见的小坑。

### 2. 把 `-P` 写成小写

在 `scp` 里，小写 `-p` 不是端口，而是保留时间和权限。

### 3. 大量文件还坚持用 `scp`

`scp` 没问题，但在大量小文件、重复同步、断点续传场景里，`rsync` 通常更舒服。

## 六、结论

`scp` 很适合临时、安全、直接地传文件。

当你已经能 SSH 上去，只是需要顺手传个文件或拉份日志回来时，它通常就是最快的办法。

