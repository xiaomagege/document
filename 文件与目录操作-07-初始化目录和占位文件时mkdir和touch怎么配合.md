# 文件与目录操作07：初始化目录和占位文件时，mkdir 和 touch 怎么配合？

很多部署、排查和脚本初始化动作，最后都离不开两件事：

- 目录先建好
- 文件先准备好

`mkdir` 和 `touch` 单独看都很基础，但它们在一起时，正好对应了“目录结构准备”和“文件标记/时间戳准备”这两类最常见的小动作。

如果只想先记 3 条：

- 建目录优先想到 `mkdir -p`
- 创建占位文件、标记文件、刷新时间戳时，用 `touch`
- 脚本里先考虑“重复执行会不会报错”，再考虑写法漂不漂亮

## 一、什么时候要把 `mkdir` 和 `touch` 放在一起看

最常见的场景有这些：

- 初始化项目目录、日志目录、备份目录
- 提前准备空日志文件或占位文件
- 用标记文件记录某一步已经完成
- 刷新文件时间戳，或让文件继承另一个文件的时间

这两个命令最常见的配合顺序就是：

```bash
mkdir -p /data/app/logs
touch /data/app/logs/app.log
```

## 二、先记住这几条高频用法

```bash
mkdir -p /data/app/{logs,tmp,backup}
mkdir -m 755 scripts
touch app.log
touch /tmp/deploy.done
touch -c exists.txt
touch -d "2026-05-06 10:30:00" app.log
touch -r source.txt target.txt
```

它们分别适合：

- `mkdir -p`：递归建目录，目录已存在也不报错
- `mkdir -m 755`：创建时指定权限
- `touch file`：创建空文件或刷新时间
- `touch /tmp/deploy.done`：创建标记文件
- `touch -c`：只更新时间，不创建新文件
- `touch -d`：手工指定时间
- `touch -r`：复制另一个文件的时间戳

## 三、它们最常见的配合场景

### 1. 初始化目录和日志文件

```bash
mkdir -p /data/app/logs
touch /data/app/logs/app.log
```

适合：

- 部署前先准备目录结构
- 提前让日志路径就位
- 避免程序第一次写日志时因为目录或文件不存在而报错

### 2. 准备标记文件和占位文件

```bash
mkdir -p /tmp/deploy-state
touch /tmp/deploy-state/step1.done
touch logs/.keep
```

这在脚本里很常见，用来表示某一步已经完成，或者保留空目录结构。

### 3. 目录建好后，再处理时间戳

```bash
mkdir -p /data/archive
touch -d "1 day ago" /data/archive/checkpoint.txt
touch -r source.txt /data/archive/checkpoint.txt
```

这适合：

- 模拟时间
- 对齐两个文件的修改时间
- 排查同步、备份、缓存判断问题

## 四、脚本里这组命令最该注意什么

### 1. 先考虑可重复执行

```bash
mkdir -p /data/app/logs
touch /data/app/logs/app.log
```

这组写法的价值就在于，脚本第二次执行通常也不会因为“目录已存在”直接失败。

### 2. 对权限敏感的目录，创建后最好确认一眼

```bash
mkdir -m 755 scripts
ls -ld scripts
```

因为最终权限还可能受 `umask` 影响。

### 3. 路径带变量时记得加引号

```bash
backup_dir="/data/backup/$(date +%F)"
mkdir -p "$backup_dir"
touch "$backup_dir/done.flag"
```

## 五、最常见的误区

### 1. 脚本里还在用普通 `mkdir`

如果脚本要重复执行，`mkdir -p` 往往更稳。

### 2. 以为 `touch` 会清空文件

它不会。默认只会创建文件，或者更新时间戳。

### 3. 不想创建新文件，却忘了 `touch -c`

如果你只是想刷新已有文件时间，用 `-c` 更符合预期。

### 4. 只创建，不确认路径和权限

尤其是变量路径和共享目录场景，建完以后最好确认一眼结果是不是你想要的。

## 六、结论

`mkdir` 负责把目录结构准备好，`touch` 负责把文件占位和时间戳准备好。

部署初始化、脚本标记、日志准备这三类场景里，把 `mkdir -p` 和 `touch` 用稳，往往比记更多参数更有价值。

