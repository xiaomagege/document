# 10｜磁盘与空间排查｜磁盘挂载为什么不生效？mount 排查时看什么

挂载问题经常表现得很“玄”：目录存在，但看不到数据；`df` 里没有新盘；重启后挂载消失；执行 `mount -a` 又报错。

`mount` 要解决的核心问题其实很简单：

**把哪个文件系统，挂到哪个目录上。**

如果只想先记 3 条：

- 挂载前先用 `lsblk -f` 确认设备、文件系统和 UUID
- 挂载点目录原有内容不会删除，只是会被新挂载覆盖显示
- 改 `/etc/fstab` 后先执行 `mount -a` 验证，不要直接重启

## 一、什么时候会用到 `mount`

这些场景都离不开 `mount`：

- 新数据盘挂到 `/data`
- 临时挂载 ISO 镜像
- 挂载 NFS 等网络文件系统
- 排查目录为什么看不到原来的文件
- 临时把文件系统改成只读或读写
- 验证 `/etc/fstab` 自动挂载配置

先看当前挂载状态：

```bash
df -Th
lsblk -f
mount | grep /data
```

这三条能把“空间、设备、挂载关系”放到一起看。

## 二、最常用的挂载命令

```bash
mkdir -p /data
mount /dev/sdb1 /data
mount -t xfs /dev/sdb1 /data
mount -o ro /dev/sdb1 /mnt
mount -o remount,rw /data
mount UUID=xxxx-xxxx /data
mount -a
umount /data
```

它们分别适合：

- `mount /dev/sdb1 /data`：临时挂载分区
- `mount -t xfs ...`：明确指定文件系统类型
- `mount -o ro ...`：只读挂载
- `mount -o remount,rw ...`：重新挂载为读写
- `mount UUID=...`：按 UUID 挂载
- `mount -a`：验证 `/etc/fstab`
- `umount`：卸载挂载点

## 三、挂载不生效时怎么排查

### 1. 先确认设备是否存在

```bash
lsblk -f
```

如果看不到目标设备，问题不在 `mount`，而在磁盘识别、云平台挂载或设备扫描。

### 2. 再确认是否有文件系统

```bash
blkid
lsblk -f
```

如果 `FSTYPE` 为空，说明分区可能还没格式化。没有文件系统的分区不能直接当普通目录挂载。

### 3. 确认挂载点是否存在

```bash
mkdir -p /data
mount /dev/sdb1 /data
```

如果报 `mount point does not exist`，就是挂载点目录不存在。

### 4. 确认是不是已经挂载到别处

```bash
findmnt /data
mount | grep /data
df -Th /data
```

有时候不是挂不上，而是已经挂到了别的位置，或者挂载点被重复配置。

### 5. 看错误类型

常见错误方向：

- `unknown filesystem type`：文件系统类型不识别或没有格式化
- `mount point does not exist`：挂载点目录不存在
- `already mounted`：已经挂载
- `permission denied`：权限、选项或网络文件系统限制
- `device is busy`：卸载时仍有进程占用

## 四、挂载后原目录文件“不见了”怎么办

挂载会覆盖显示挂载点目录。

比如 `/data` 原来有文件，执行：

```bash
mount /dev/sdb1 /data
```

之后看到的是 `/dev/sdb1` 里的内容。原来 `/data` 目录里的文件没有被删，只是被挂载遮住了。

卸载后会重新看到：

```bash
umount /data
```

这也是为什么生产环境挂载前要确认挂载点目录是否为空或是否已有业务文件。

## 五、`/etc/fstab` 怎么稳一点

自动挂载建议用 UUID：

```bash
blkid
lsblk -f
```

修改前先备份：

```bash
cp -a /etc/fstab /etc/fstab.bak.$(date +%F)
```

修改后先验证：

```bash
mount -a
```

只有 `mount -a` 没报错，再考虑重启验证。否则配置错误可能导致启动阶段挂载失败。

## 六、卸载失败时看什么

卸载命令是：

```bash
umount /data
```

如果提示设备忙，先看占用进程：

```bash
lsof +f -- /data
fuser -vm /data
```

不要为了卸载直接强行处理，先确认是否有数据库、日志写入、备份任务或 shell 当前目录还在挂载点里面。

## 七、结论

`mount` 排查的关键不是背参数，而是把链路看完整：设备是否存在、分区是否有文件系统、挂载点是否存在、是否已经挂载、`fstab` 是否正确。

按 `lsblk -f -> blkid -> mount -> df -Th -> mount -a` 这个顺序看，绝大多数挂载问题都能很快收敛。
