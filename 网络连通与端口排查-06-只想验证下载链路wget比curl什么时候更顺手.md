# 网络连通与端口排查06：只想验证下载链路，wget 比 curl 什么时候更顺手

`wget` 和 `curl` 很容易被放在一起比较，但它们最顺手的场景并不完全一样。

如果你的目标是“把文件稳稳地下回来”，`wget` 往往更直接；如果你的目标是“看接口头、发请求、调 HTTP 细节”，`curl` 往往更灵活。

如果只想先记 3 条：

- 下载文件优先考虑 `wget`
- 断点续传、大文件拉取时，`wget -c` 很实用
- 调接口、看响应头、发 POST 请求时，还是优先 `curl`

## 一、什么情况下先用 `wget`

`wget` 最适合这些场景：

- 下载安装包、备份文件、镜像包
- 验证下载链路本身通不通
- 想断点续传，不想重头再下
- 想后台拉一个大文件

最常见的起手式就是：

```bash
wget URL
wget -c URL
```

## 二、先记住这几条命令

```bash
wget https://example.com/app.tar.gz
wget -O app.tar.gz https://example.com/download
wget -P /tmp https://example.com/app.tar.gz
wget -c https://example.com/big.iso
wget -b https://example.com/big.iso
```

它们分别适合：

- 直接下载
- 指定保存文件名
- 指定保存目录
- 断点续传
- 后台下载

## 三、排查下载链路时，重点看什么

### 1. 地址能不能直接拉下来

```bash
wget https://example.com/app.tar.gz
```

这一步适合回答一个很直接的问题：这个下载地址到底能不能正常拿到文件。

### 2. 大文件中断后能不能续传

```bash
wget -c https://example.com/big.iso
```

如果网络不稳，`-c` 很有价值。它能避免每次中断都从头再来。

### 3. 后台拉取时怎么跟进度

```bash
wget -b https://example.com/big.iso
tail -f wget-log
```

这很适合长时间下载，尤其是你不想一直盯着终端的时候。

## 四、`wget` 和 `curl` 到底怎么分工

### 1. `wget`

更适合：

- 直接把文件下载下来
- 批量拉 URL 列表
- 断点续传
- 后台下载

### 2. `curl`

更适合：

- 查看响应头
- 测接口状态码
- 发 GET/POST/PUT 等请求
- 做探活和自动化检查

如果你只是想确认某个 HTTP 服务有没有回响应、状态码对不对，通常先用 `curl` 更高效。

如果你已经明确要“下载这个文件”，通常 `wget` 更顺手。

## 五、最常见的几种用法

### 1. 下载并改名

```bash
wget -O app.tar.gz https://example.com/download
```

这适合 URL 很丑、文件名不固定，或者你就是想本地统一命名的场景。

### 2. 下载到指定目录

```bash
wget -P /tmp https://example.com/app.tar.gz
```

适合临时目录或脚本固定输出目录。

### 3. 批量下载

```bash
wget -i urls.txt
```

如果你手里是一串下载地址，这个写法就很省事。

### 4. 设置超时和重试

```bash
wget --tries=3 --timeout=10 https://example.com/app.tar.gz
```

这在脚本里很实用，可以避免网络异常时无限卡住。

## 六、最常见的误区

### 1. 用 `wget` 做细致的 HTTP 接口调试

能做，但通常不如 `curl` 顺手。尤其是你要看头、带认证、发 JSON、调 POST 请求时。

### 2. 下载大文件不带 `-c`

网络一断就重头来，尤其浪费时间。

### 3. 只顾下载，不做校验

下载完成后，至少要看文件大小，必要时再校验 hash。否则“下载成功”不等于“文件可用”。

## 七、结论

`wget` 最擅长的是“把文件稳稳地下回来”。

所以文件下载、续传、后台拉取时优先想起它；真正做接口调试和探活时，再把主力交给 `curl`。

