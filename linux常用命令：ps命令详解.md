# linux常用命令：ps命令详解

在 Linux 日常运维和开发中，`ps` 是排查进程问题最常用的命令之一。无论是查看服务是否启动、定位高 CPU 进程，还是分析进程间关系，`ps` 都非常高效。

---

## 一、`ps` 命令是什么？

`ps`（process status）用于**查看当前系统中的进程快照信息**。  
注意：`ps` 显示的是执行瞬间的数据，不会像 `top` 那样实时刷新。

---

## 二、常见语法

```bash
ps [选项]
```

`ps` 支持三种风格参数（可混用）：

- Unix 风格：`-ef`
- BSD 风格：`aux`
- GNU 长参数：`--sort`

---

## 三、最常用命令组合

### 1. 查看当前终端下的进程
```bash
ps
```

### 2. 查看系统全部进程（标准格式）
```bash
ps -ef
```

![](D:\github\document\cmd\ps-ef.png)

常见字段：

- `UID`：进程所属用户
- `PID`：进程 ID
- `PPID`：父进程 ID
- `C`：CPU 使用相关信息
- `STIME`：启动时间
- `TTY`：终端
- `TIME`：累计 CPU 时间
- `CMD`：启动命令

### 3. 查看全部进程（资源视角）
```bash
ps aux
```

![](D:\github\document\cmd\psaux.png)

常见字段：

- `USER`：所属用户
- `PID`：进程 ID
- `%CPU`：CPU 使用率
- `%MEM`：内存使用率
- `VSZ`：虚拟内存大小（KB）
- `RSS`：物理内存占用（KB）
- `STAT`：进程状态
- `START`：启动时间
- `TIME`：累计 CPU 时间
- `COMMAND`：命令

---

## 四、重点参数说明

- `-e`：显示所有进程（等价 `-A`）
- `-f`：完整格式输出
- `-u <用户>`：查看指定用户进程
- `-p <PID>`：查看指定 PID
- `-o <字段>`：自定义输出列
- `--sort=<字段>`：按字段排序（如 `%cpu`、`-rss`）

---

## 五、实战示例

### 1. 查某个服务是否在运行
```bash
ps -ef | grep nginx
```

更推荐排除 grep 本身：
```bash
ps -ef | grep [n]ginx
```

### 2. 按 CPU 使用率排序（高到低）
```bash
ps aux --sort=-%cpu | head -10
```

### 3. 按内存占用排序
```bash
ps aux --sort=-%mem | head -10
```

### 4. 查看指定 PID 详细信息
```bash
ps -fp 1234
```

### 5. 自定义显示列（常用于脚本）
```bash
ps -eo pid,ppid,user,%cpu,%mem,cmd --sort=-%cpu | head
```

---

---

## 六、进程状态 `STAT` 常见值

- `R`：运行中（Running）
- `S`：可中断睡眠（Sleeping）
- `D`：不可中断睡眠（通常在 IO）
- `T`：已停止/跟踪
- `Z`：僵尸进程（Zombie）
- `I`：空闲内核线程（部分系统）

附加标志（可能组合出现）：
- `<`：高优先级
- `N`：低优先级
- `s`：会话首进程
- `l`：多线程进程
- `+`：前台进程组

---

## 七、`ps` 与 `top` 的区别

- `ps`：一次性快照，适合脚本和精准查询。
- `top`：动态刷新，适合实时监控。

通常排障流程是：先用 `top` 观察，再用 `ps` 精确筛选和定位。

---

## 八、常见问题与建议

1. `ps -ef` 和 `ps aux` 哪个更好？  
两者都常用。`-ef` 偏结构化，`aux` 更直观看资源占用。

2. 为什么看不到某些进程？  
可能权限不足，尝试使用 `sudo ps -ef`。

3. 如何避免误杀进程？  
先用 `ps -fp <PID>` 确认命令和父子关系，再执行 `kill`。

---

## 九、总结

`ps` 是 Linux 进程排查的基础能力。建议至少熟练掌握以下三条：

```bash
ps -ef
ps aux --sort=-%cpu | head
ps -eo pid,ppid,user,%cpu,%mem,cmd --sort=-%mem | head
```

掌握这三种用法，已经能解决大多数进程查看与性能定位问题。

