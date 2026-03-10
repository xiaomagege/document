# Linux 常用命令整理

以下内容根据图片识别整理为 Markdown 文档，个别字符因图片清晰度和反光影响，可能存在轻微误差。

## 1. 进程监控

```bash
(1) top -c
# 按照 CPU 使用率排序：Shift + P
# 按照内存使用率排序：Shift + M

(2) ps 命令
# 按照第四列（内存使用情况）中数字逆序进行排列和输出
ps aux | head -1; ps aux | sort -rnk 4 | head -5
# ps aux | head -1 显示列头
# sort 命令：-r 反转，-n 数字值，-k 关键字

(3) top -p pid
# 实时监控进程
```

## 2. 某个进程的线程（进程）监控

```bash
top -Hp pid
```

## 3. 打印进程的堆栈信息

```bash
printf '%x\n' pid   # 获得十六进制进程号
jstack pid | grep pid -A 50
# 打印进程 pid 下，子进程（线程）pid（十六进制）的堆栈信息 50 行
```

## 4. 防火墙

### 4.1 firewalld

```bash
# 查看防火墙开放端口
firewall-cmd --zone=public --list-ports

# 开放防火墙端口
firewall-cmd --zone=public --add-port=8809/tcp --permanent
firewall-cmd --reload
```

### 4.2 iptables

```bash
# 查看防火墙策略
iptables -nL

# 添加防火墙策略
iptables -A INPUT -p tcp --dport 8809 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 8809 -j ACCEPT
```

## 5. 根据端口号查看进程

```bash
netstat -lnp | grep 5601
```

## 6. 后台启动进程

```bash
nohup ./kibana --allow-root &
```

## 7. 查看 CPU 核数

```bash
cat /proc/cpuinfo | grep "processor" | wc -l
```

## 8. 查看进程的目录

```bash
ps -Af | grep str      # 查看进程的 pid
ls -l /proc/pid        # 根据 pid 查找进程目录
```

说明：

`/proc` 文件系统是一个伪文件系统，只存在于内存中，而不占用外存空间。它以文件系统的方式为访问系统内核数据的操作提供接口。

## 9. 查看未占用端口进程

```bash
netstat -tunlp | grep 9500
```

## 10. 查看某个进程的端口

```bash
netstat -nap | grep pid
```

## 11. 格式化查看当前时间

```bash
date +%Y-%m-%d' '%H:%M:%S
```

## 12. 查看磁盘空间

```bash
df -h
```

## 13. 当前文件夹大小

```bash
du -sh
```

## 14. 查看 CPU 核数

```bash
cat /proc/cpuinfo | grep "cpu cores" | uniq
```

## 15. 查看 CPU 逻辑核数

```bash
cat /proc/cpuinfo | grep "processor" | wc -l
```

## 16. 查看 CPU 个数

```bash
cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l
```

## 17. rsync 传输文件

```bash
rsync -avz /var/lib/jenkins/workspace/ibase-admin/ibase-admin/target/ibase-admin/ \
root@172.168.3.213:/root/ibase_admin/BUILD/opt/zf_bas
```

## 18. 查看操作系统发行版本

```bash
cat /etc/os-release
nkvers
```

## 19. 查看文件夹大小

```bash
du -ah --max-depth=1 /tmp | sort -hr | head -n 20
```

## 20. 查看分区及挂载情况

```bash
lsblk -f
```

## 21. 查看 CPU 情况

```bash
lscpu
```

示例输出：

```text
Architecture:           x86_64
CPU op-mode(s):         32-bit, 64-bit
Byte Order:             Little Endian
CPU(s):                 40          # 逻辑核数
On-line CPU(s) list:    0-39
Thread(s) per core:     2           # 每核线程数
Core(s) per socket:     10          # 每个 CPU 核数
Socket(s):              2           # CPU 个数
NUMA node(s):           2
Vendor ID:              GenuineIntel
CPU family:             6
Model:                  79
Model name:             Intel(R) Xeon(R) CPU E5-2630 v4 @ 2.20GHz
Stepping:               1
```

## 22. rpm 相关命令

```bash
rpm -qa              # 查看已安装的所有 rpm 包
rpm -qa | grep xxx   # 搜索
rpm -e xxx           # 卸载指定 rpm 包
rpm -ivh xxx.rpm     # 安装 rpm 包
```

## 23. vim 相关命令

```bash
Shift + G    # 跳到文件末尾
?string      # 从下往上查找字符串
:nob         # 取消注释
```

## 24. MobaXterm 快捷键

```text
复制快捷键：Ctrl + Ins
粘贴快捷键：Shift + Ins
```

## 25. systemctl

```bash
# 列出所有服务
systemctl list-units --type=service

# 列出所有已加载的 units
systemctl
systemctl | grep 名称
```

## 26. 查看系统文件句柄数

> 图片中仅能清晰识别标题，具体命令不够清楚。

## 27. 根据打开文件句柄数按降序排列，第二列为进程

```bash
lsof | awk '{print $2}' | sort | uniq -c | sort -nr | more
```

## 28. sha256 计算

```bash
sha256sum a > a.sha256sum
```

## 29. IO 情况监控

```bash
iostat -x -k 2 3
# -k 显示 KB/s
# -m 显示 MB/s

iotop -oPa
```

## 30. 部署

```bash
nohup java -jar bas-app-0.0.1-SNAPSHOT.jar > bas.log 2>&1 &
nohup java -jar ibase-update-helper-1.7.0.jar > updateZoneCode.log 2>&1 &
```

## 31. 查看磁盘的读写速度

```bash
iostat -d -m 1 10
# -d 磁盘的使用情况
# -m 以 M 为单位显示
# 每 1s 输出一次
```

## 32. 磁盘分区情况

```bash
fdisk -l
```

查看挂载情况：

```bash
lsblk
```

是否是固态：

```bash
cat /sys/block/sdb/queue/rotational
```

## 33. 查看硬盘型号、厂商

```bash
lshw -class disk
```

## 34. 文件远程同步

```bash
rsync -av root@10.11.203.22:/opt/zfnwq/base/nginx/html/event ./
```

## 35. 遍历查找文件文本内容

```bash
grep -r Exception ./
```

## 36. SUP 允许同时登录

```bash
sed -i 's@ multiAccountLogin:.*@ multiAccountLogin: true@g' /opt/njzf/sup-service/config/bootstrap.yml
systemctl restart sup-service
```

## 37. 查询/删除 sequence

```sql
select * from user_sequences;

drop sequence BAS_ALARM_META_ID_SEQ;
drop sequence BAS_ALARM_TAG_MAPPING_ID_SEQ;
drop sequence BAS_INCIDENT_META_ID_SEQ;
drop sequence BAS_INCIDENT_TAG_MAPPING_ID_SEQ;
drop sequence BAS_PAGE_FILTER_CONFIG_ID_SEQ;
drop sequence BAS_PROPERTY_META_ID_SEQ;
drop sequence BAS_REF_CONSTANT_DATA_ID_SEQ;
drop sequence BAS_BASELINE_VALUE_CORRECT_ID_SEQ;
drop sequence BAS_INDEX_CLASSIFY_ID_SEQ;
drop sequence BAS_INCIDENT_VERIFY_FIELD_ID_SEQ;
```