# SSH与远程运维01：SSH 登录失败时，第一步应该查什么？

SSH 问题最容易让人焦躁，因为它往往卡在最入口的一层：机器在那儿，你却进不去。

这类问题如果一上来就乱试密码、乱换密钥，通常只会更乱。更稳的做法，是先把它拆成几层：

- 网络通不通
- 端口开没开
- 用户和密钥对不对
- 服务端到底拒绝了什么

如果只想先记 3 条：

- 登录失败先确认用户、主机、端口是不是对的
- 连不上时直接加 `-v`，别盲猜
- 免密、多环境连接时，尽早用 `~/.ssh/config`

## 一、SSH 登录失败时，先分成哪几类

常见报错其实大致就落在这几类：

- `Connection timed out`：更像网络或安全组问题
- `Connection refused`：更像端口没监听或服务没起
- `Permission denied`：更像用户、密码、密钥或授权问题
- 卡在握手过程：更像算法、证书、配置兼容性问题

所以第一步不是“继续试”，而是先分层判断。

## 二、先记住这几条命令

```bash
ssh user@host
ssh -p 2222 user@host
ssh -i ~/.ssh/id_rsa user@host
ssh -v user@host
nc -vz host 22
```

它们分别适合：

- `ssh user@host`：最基本登录
- `-p`：指定端口
- `-i`：指定私钥
- `-v`：看详细连接过程
- `nc -vz`：先测 22 端口是不是可达

## 三、推荐的排查顺序

### 1. 先确认主机和端口能不能到

```bash
nc -vz host 22
```

如果端口都不通，问题通常还没到 SSH 认证层。

这时优先查：

- 安全组
- 防火墙
- SSH 服务是否监听
- IP 或端口是不是写错了

### 2. 再用 `ssh -v` 看握手过程

```bash
ssh -v user@host
```

这一步最有价值，因为它会告诉你：

- 连到了哪台主机
- 用了哪个私钥
- 试了哪些认证方式
- 失败是发生在网络、握手还是认证阶段

如果一层不够，再用：

```bash
ssh -vvv user@host
```

### 3. 再判断是用户问题还是密钥问题

常见问题包括：

- 用户名不对
- 私钥文件不对
- 公钥没进远端 `authorized_keys`
- 私钥权限太宽
- 服务端禁止密码登录或 root 登录

私钥权限至少要注意：

```bash
chmod 600 ~/.ssh/id_rsa
```

### 4. 多环境连接时别一直手打参数

`~/.ssh/config` 很值得早点用起来。

例如：

```text
Host prod
    HostName example.com
    User deploy
    Port 2222
    IdentityFile ~/.ssh/id_rsa
```

之后就能直接：

```bash
ssh prod
```

## 四、最常见的 4 个问题

### 1. `Connection refused`

更像：

- SSH 服务没启动
- 端口不是 22
- 服务端没监听这个端口

服务端可以看：

```bash
ss -lntp | grep ':22'
systemctl status sshd
```

### 2. `Connection timed out`

更像：

- 网络不通
- 安全组阻断
- 防火墙拦截
- 中间链路问题

### 3. `Permission denied`

更像：

- 用户名错了
- 密钥不对
- 认证方式不匹配
- 服务端授权没配好

### 4. 只能本机 SSH，外部进不来

这通常要看：

- 服务监听地址
- 端口是否对外开放
- 安全组来源 IP 限制

## 五、除了登录，SSH 还常用在哪

### 1. 执行远程命令

```bash
ssh user@host 'hostname && uptime'
```

### 2. 本地端口转发

```bash
ssh -L 8080:127.0.0.1:80 user@host
```

### 3. 动态代理

```bash
ssh -D 1080 -N user@host
```

这些都很常用，但前提都是：基本连接要先打通。

## 六、结论

SSH 登录失败时，先别乱试，把问题分成“端口可达、服务监听、用户认证、密钥配置”四层去看。

先 `nc`，再 `ssh -v`，这条顺序通常比反复硬连更有效。

