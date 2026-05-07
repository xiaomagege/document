# Linux磁盘与空间排查命令：mount命令详解

在 Linux 中，磁盘、分区、镜像文件、网络文件系统等资源通常需要挂载到某个目录后才能访问。`mount` 就是用来把文件系统挂载到目录上的核心命令。

无论是新增数据盘、挂载 NFS、临时挂载 ISO 镜像，还是排查目录为什么看不到原来的内容，`mount` 都是必须掌握的命令。

---

## 一、`mount` 是什么？

`mount` 用于把一个文件系统挂载到 Linux 目录树中的某个挂载点。

它主要用于：

- 挂载磁盘分区
- 挂载新数据盘
- 查看当前系统挂载情况
- 挂载 NFS 等网络文件系统
- 挂载 ISO 镜像或 loop 设备
- 临时调整只读、读写等挂载选项

Linux 中所有文件都在同一棵目录树下。一个分区如果没有挂载到目录，就不能通过普通路径访问。

例如把 `/dev/sdb1` 挂载到 `/data` 后，访问 `/data` 实际就是访问这个分区里的内容。

---

## 二、常见使用语法与重点参数

```bash
mount [选项] 设备 挂载点
mount [选项]
```

常用参数说明：

- `-t 类型`：指定文件系统类型，例如 `xfs`、`ext4`、`nfs`
- `-o 选项`：指定挂载选项，例如 `ro`、`rw`、`loop`
- `-a`：挂载 `/etc/fstab` 中配置的所有文件系统
- `-r`：以只读方式挂载
- `-w`：以读写方式挂载
- `--bind`：把一个目录挂载到另一个目录
- `-L 标签`：按文件系统标签挂载
- `-U UUID`：按 UUID 挂载

常见示例：

```bash
mount
mount /dev/sdb1 /data
mount -t xfs /dev/sdb1 /data
mount -o ro /dev/sdb1 /mnt
mount -a
```

---

## 三、常用命令与使用场景

### 1. 查看当前挂载情况

```bash
mount
```

不带参数执行会显示当前系统所有挂载信息。

输出较多时，可以结合过滤：

```bash
mount | grep /data
```

日常查看磁盘挂载点更常用：

```bash
df -Th
lsblk -f
```

### 2. 挂载磁盘分区

```bash
mount /dev/sdb1 /data
```

表示把 `/dev/sdb1` 挂载到 `/data`。

挂载前需要确保挂载点存在：

```bash
mkdir -p /data
mount /dev/sdb1 /data
```

### 3. 指定文件系统类型挂载

```bash
mount -t xfs /dev/sdb1 /data
```

大多数情况下系统可以自动识别文件系统类型，但明确指定类型可以让操作更清晰。

常见类型包括：

- `xfs`
- `ext4`
- `nfs`
- `iso9660`

### 4. 只读挂载

```bash
mount -o ro /dev/sdb1 /mnt
```

只读挂载适合数据恢复、故障排查或不希望误写入的场景。

也可以写成：

```bash
mount -r /dev/sdb1 /mnt
```

### 5. 重新挂载为读写或只读

```bash
mount -o remount,rw /data
```

把已挂载的 `/data` 重新挂载为读写。

重新挂载为只读：

```bash
mount -o remount,ro /data
```

这类操作通常用于维护、修复或临时调整挂载参数。

### 6. 按 UUID 挂载

```bash
mount UUID=xxxx-xxxx /data
```

也可以使用：

```bash
mount -U xxxx-xxxx /data
```

UUID 可以通过以下命令查看：

```bash
blkid
lsblk -f
```

在 `/etc/fstab` 中推荐使用 UUID，避免设备名变化导致挂载到错误磁盘。

### 7. 挂载 `/etc/fstab` 中的文件系统

```bash
mount -a
```

会尝试挂载 `/etc/fstab` 中配置但尚未挂载的文件系统。

修改 `/etc/fstab` 后，建议先执行：

```bash
mount -a
```

验证配置是否正确，再重启服务器。

### 8. 挂载 ISO 镜像

```bash
mount -o loop system.iso /mnt
```

`loop` 选项可以把镜像文件当作块设备挂载。

挂载后可以进入 `/mnt` 查看镜像内容。

### 9. bind 挂载目录

```bash
mount --bind /data/app /mnt/app
```

把一个已有目录挂载到另一个目录位置。

这种方式常用于容器、chroot、临时目录映射等场景。

### 10. 卸载挂载点

虽然卸载使用的是 `umount`，但它和 `mount` 经常配合使用：

```bash
umount /data
```

如果提示设备忙，可以查看占用进程：

```bash
lsof +f -- /data
```

或：

```bash
fuser -vm /data
```

---

## 四、常见问题与建议

### 1. 挂载后目录原来的文件不见了？

挂载会把新文件系统覆盖显示在挂载点目录上。

例如 `/data` 原来有文件，执行：

```bash
mount /dev/sdb1 /data
```

之后看到的是 `/dev/sdb1` 中的内容，原来 `/data` 目录里的文件并没有被删除，只是被挂载覆盖暂时看不到。

卸载后会重新显示：

```bash
umount /data
```

### 2. `mount: unknown filesystem type` 怎么办？

可能原因：

- 文件系统类型写错
- 分区没有格式化
- 系统缺少对应文件系统支持
- 设备不是有效文件系统

可以先检查：

```bash
lsblk -f
blkid
```

### 3. `mount: mount point does not exist` 怎么办？

说明挂载目录不存在，需要先创建：

```bash
mkdir -p /data
mount /dev/sdb1 /data
```

### 4. `mount -a` 有什么风险？

`mount -a` 会按 `/etc/fstab` 尝试挂载文件系统。如果配置错误，可能导致挂载失败；如果写入了错误设备，也可能挂载到非预期位置。

修改 `/etc/fstab` 前建议先备份：

```bash
cp -a /etc/fstab /etc/fstab.bak.$(date +%F)
```

修改后先用 `mount -a` 验证，不要直接重启。

### 5. 生产环境挂载数据盘的建议流程是什么？

建议按顺序确认：

```bash
lsblk -f
blkid
mkdir -p /data
mount /dev/sdb1 /data
df -Th
```

确认无误后再配置 `/etc/fstab` 实现开机自动挂载。

---

## 五、常用命令速查

```bash
mount
mount | grep /data
mkdir -p /data
mount /dev/sdb1 /data
mount -t xfs /dev/sdb1 /data
mount -o ro /dev/sdb1 /mnt
mount -o remount,rw /data
mount -o remount,ro /data
blkid
lsblk -f
mount UUID=xxxx-xxxx /data
mount -U xxxx-xxxx /data
mount -a
mount -o loop system.iso /mnt
mount --bind /data/app /mnt/app
umount /data
lsof +f -- /data
fuser -vm /data
```

---

## 六、总结

`mount` 是 Linux 文件系统挂载的核心命令。它解决的是“把哪个设备或文件系统挂到哪个目录上”的问题。

日常使用建议重点掌握：

```bash
mount /dev/sdb1 /data
mount -t xfs /dev/sdb1 /data
mount -o remount,rw /data
mount -a
```

生产环境中，挂载前要先用 `lsblk -f`、`blkid`、`df -Th` 确认设备、文件系统和挂载点。涉及 `/etc/fstab` 时先备份、再验证，避免因为自动挂载配置错误影响系统启动。