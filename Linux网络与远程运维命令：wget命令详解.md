# Linux网络与远程运维命令：wget命令详解

`wget` 是 Linux 中常用的命令行下载工具，适合从 HTTP、HTTPS、FTP 地址下载文件。它用法简单，支持断点续传、后台下载、递归下载等功能。

如果主要任务是“下载文件”，`wget` 通常比 `curl` 更直接；如果主要任务是“调试接口”，`curl` 往往更灵活。

---

## 一、`wget` 是什么？

`wget` 用于从网络下载文件。

它主要用于：

- 下载单个文件
- 指定保存文件名
- 断点续传
- 后台下载
- 限速下载
- 批量下载 URL 列表
- 简单递归下载网页资源

常见场景包括：

- 下载安装包
- 下载备份文件
- 下载脚本或配置
- 从内网文件服务器拉取文件

---

## 二、常见使用语法与重点参数

```bash
wget [选项] URL
```

常用参数说明：

- `-O 文件`：保存为指定文件名
- `-P 目录`：保存到指定目录
- `-c`：断点续传
- `-b`：后台下载
- `-q`：安静模式
- `--limit-rate=速度`：限制下载速度
- `--tries=N`：设置重试次数
- `--timeout=秒`：设置超时时间
- `-i 文件`：从文件读取 URL 列表批量下载
- `--no-check-certificate`：不校验证书

常见示例：

```bash
wget https://example.com/app.tar.gz
wget -O app.tar.gz https://example.com/download
wget -P /tmp https://example.com/app.tar.gz
wget -c https://example.com/big.iso
```

---

## 三、常用命令与使用场景

### 1. 下载文件

```bash
wget https://example.com/app.tar.gz
```

默认会把文件保存到当前目录，并使用 URL 中的文件名。

### 2. 指定保存文件名

```bash
wget -O app.tar.gz https://example.com/download
```

当 URL 中没有合适文件名，或者下载地址是动态接口时，可以用 `-O` 指定保存名称。

### 3. 保存到指定目录

```bash
wget -P /tmp https://example.com/app.tar.gz
```

会把文件下载到 `/tmp` 目录下。

### 4. 断点续传

```bash
wget -c https://example.com/big.iso
```

如果之前下载中断，`-c` 可以尝试从中断位置继续下载。

适合大文件、弱网络环境。

### 5. 后台下载

```bash
wget -b https://example.com/big.iso
```

后台下载时，日志默认写入 `wget-log`。

查看日志：

```bash
tail -f wget-log
```

### 6. 限制下载速度

```bash
wget --limit-rate=1m https://example.com/big.iso
```

避免下载占满带宽。

速度单位可以使用 `k`、`m` 等。

### 7. 设置重试次数和超时

```bash
wget --tries=3 --timeout=10 https://example.com/app.tar.gz
```

网络不稳定时，可以通过重试和超时控制下载行为。

### 8. 批量下载 URL 列表

```bash
wget -i urls.txt
```

`urls.txt` 中每行一个 URL。

适合批量下载多个文件。

### 9. 安静模式下载

```bash
wget -q https://example.com/app.tar.gz
```

不输出下载过程，适合脚本中使用。

如果需要安静模式并指定文件名：

```bash
wget -q -O app.tar.gz https://example.com/download
```

### 10. 忽略证书校验

```bash
wget --no-check-certificate https://self-signed.example.com/file.tar.gz
```

只建议用于测试环境或临时排查。

生产环境应修复证书配置，而不是长期忽略校验。

---

## 四、常见问题与建议

### 1. `wget -O` 和 `wget -P` 有什么区别？

- `-O` 指定保存后的文件名
- `-P` 指定保存目录

例如：

```bash
wget -O /tmp/app.tar.gz https://example.com/download
wget -P /tmp https://example.com/app.tar.gz
```

第一条明确保存为 `/tmp/app.tar.gz`，第二条保存到 `/tmp` 目录并使用远程文件名。

### 2. 下载中断后怎么继续？

使用：

```bash
wget -c URL
```

前提是服务端支持断点续传。

### 3. 后台下载在哪里看进度？

后台下载默认日志文件是 `wget-log`：

```bash
tail -f wget-log
```

### 4. `wget` 和 `curl` 怎么选？

简单理解：

- 下载文件：优先 `wget`
- 调试接口、看响应头、发 POST 请求：优先 `curl`
- 脚本探活：通常 `curl` 更方便
- 批量下载 URL 列表：`wget -i` 很方便

### 5. 生产环境使用 `wget` 有什么建议？

建议：

- 下载大文件时使用 `-c` 支持续传
- 下载后校验文件大小或哈希
- 脚本中设置超时和重试次数
- 不要长期使用 `--no-check-certificate`
- 不要直接执行从网络下载的脚本，先查看内容

---

## 五、常用命令速查

```bash
wget https://example.com/app.tar.gz
wget -O app.tar.gz https://example.com/download
wget -O /tmp/app.tar.gz https://example.com/download
wget -P /tmp https://example.com/app.tar.gz
wget -c https://example.com/big.iso
wget -b https://example.com/big.iso
tail -f wget-log
wget --limit-rate=1m https://example.com/big.iso
wget --tries=3 --timeout=10 https://example.com/app.tar.gz
wget -i urls.txt
wget -q https://example.com/app.tar.gz
wget -q -O app.tar.gz https://example.com/download
wget --no-check-certificate https://self-signed.example.com/file.tar.gz
```

---

## 六、总结

`wget` 是 Linux 中非常实用的文件下载工具。建议优先掌握：

```bash
wget URL
wget -O file URL
wget -P dir URL
wget -c URL
wget -i urls.txt
```

如果目标是下载文件，`wget` 简单直接；如果目标是调试接口、请求头、状态码和请求耗时，`curl` 更适合。下载生产文件时，记得校验文件完整性，避免下载中断或内容异常影响后续部署。