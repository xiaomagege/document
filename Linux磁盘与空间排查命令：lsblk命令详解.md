# Linux磁盘与空间排查命令：lsblk命令详解

在 Linux 中排查磁盘、分区、挂载点和数据盘识别问题时，`lsblk` 是一个非常高频的命令。它可以用树形结构展示块设备关系，让我们快速看清楚磁盘、分区、LVM、挂载目录之间的对应关系。

相比直接查看 `/dev` 下的设备文件，`lsblk` 的输出更直观，适合在磁盘扩容、挂载数据盘、排查云服务器磁盘、确认系统盘和数据盘时使用。

---

## 一、`lsblk` 是什么？

`lsblk` 是 list block devices 的缩写，用于列出系统中的块设备信息。

它主要用于：

- 查看服务器识别到了哪些磁盘
- 查看磁盘和分区的层级关系
- 查看分区是否已经挂载
- 查看挂载点对应的设备
- 查看文件系统类型和 UUID
- 辅助磁盘分区、格式化、挂载和扩容操作

常见块设备包括：

- 物理磁盘，例如 `/dev/sda`
- 云盘或虚拟磁盘，例如 `/dev/vda`、`/dev/xvda`
- NVMe 磁盘，例如 `/dev/nvme0n1`
- 分区，例如 `/dev/sda1`
- LVM 逻辑卷
- loop 设备

---

## 二、常见使用语法与重点参数

```bash
lsblk [选项] [设备]
```

常用参数说明：

- `-f`：显示文件系统类型、LABEL、UUID、挂载点
- `-a`：显示所有块设备，包括空设备
- `-d`：只显示磁盘本身，不显示分区
- `-o 字段`：自定义输出字段
- `-p`：显示完整设备路径
- `-r`：以原始格式输出，适合脚本处理
- `-J`：以 JSON 格式输出
- `-b`：以字节为单位显示大小
- `-m`：显示设备权限、属主和属组

常见示例：

```bash
lsblk
lsblk -f
lsblk -d
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT
lsblk -p
```

---

## 三、常用命令与使用场景

### 1. 查看所有块设备

```bash
lsblk
```

默认会以树形结构显示磁盘、分区和挂载点关系。

常见输出字段包括：

- `NAME`：设备名
- `MAJ:MIN`：主设备号和次设备号
- `RM`：是否可移除设备
- `SIZE`：设备大小
- `RO`：是否只读
- `TYPE`：设备类型
- `MOUNTPOINTS`：挂载点

### 2. 查看文件系统和 UUID

```bash
lsblk -f
```

会显示文件系统类型、标签、UUID 和挂载点。

在配置 `/etc/fstab` 自动挂载时，通常需要 UUID，可以通过这个命令查看。

### 3. 只查看磁盘，不显示分区

```bash
lsblk -d
```

适合快速查看当前机器有几块磁盘以及每块磁盘大小。

如果只关心磁盘设备，可以配合字段筛选：

```bash
lsblk -d -o NAME,SIZE,MODEL,TYPE
```

### 4. 自定义输出字段

```bash
lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINTS
```

这样可以只显示排查时真正关心的信息。

常用字段包括：

- `NAME`
- `SIZE`
- `TYPE`
- `FSTYPE`
- `UUID`
- `MOUNTPOINTS`
- `MODEL`
- `SERIAL`

### 5. 显示完整设备路径

```bash
lsblk -p
```

默认输出可能是 `sda`、`sda1`，加 `-p` 后会显示 `/dev/sda`、`/dev/sda1`。

在写脚本或执行格式化、挂载命令前，使用完整路径更清晰。

### 6. 查看某个设备

```bash
lsblk /dev/sda
```

只显示指定设备及其分区关系。

对于 NVMe 设备，可以这样查看：

```bash
lsblk /dev/nvme0n1
```

### 7. 以 JSON 格式输出

```bash
lsblk -J
```

适合自动化脚本或配置管理工具读取。

如果只输出指定字段：

```bash
lsblk -J -o NAME,SIZE,FSTYPE,MOUNTPOINTS
```

### 8. 查看设备权限

```bash
lsblk -m
```

会显示设备文件的权限、属主和属组。

当普通用户无法访问某个块设备时，可以用它辅助排查权限问题。

### 9. 判断新磁盘是否被系统识别

插入新盘或云平台挂载数据盘后，可以执行：

```bash
lsblk
```

如果看到一个没有分区、没有挂载点的新设备，例如 `/dev/vdb`，说明系统已经识别到磁盘，但还需要后续分区、格式化和挂载。

---

## 四、常见问题与建议

### 1. `lsblk` 看到磁盘了，为什么不能直接使用？

新磁盘通常还需要完成几个步骤：

```bash
分区 -> 格式化 -> 创建挂载目录 -> 挂载 -> 配置开机自动挂载
```

`lsblk` 只能说明系统识别到了设备，不代表文件系统已经可用。

### 2. `lsblk` 看不到新磁盘怎么办？

可以检查：

- 云平台或虚拟化平台是否已经挂载磁盘
- 系统是否需要重新扫描磁盘总线
- 设备名称是否不是预期的 `/dev/sdb`，而是 `/dev/vdb` 或 `/dev/nvme...`
- `dmesg` 是否有磁盘识别信息

常用辅助命令：

```bash
dmesg | tail
fdisk -l
```

### 3. `MOUNTPOINTS` 为空是什么意思？

说明该设备或分区当前没有挂载到目录。

如果是新数据盘，需要先确认是否已经分区和格式化，再执行挂载。

### 4. `lsblk -f` 看不到 UUID 怎么办？

可能原因：

- 设备没有文件系统
- 分区还没有格式化
- 读取权限不足

可以用下面命令进一步确认：

```bash
blkid
```

### 5. 生产环境操作磁盘前有什么建议？

执行 `fdisk`、`mkfs`、`mount` 等操作前，建议先保存当前设备信息：

```bash
lsblk -f
blkid
df -Th
```

尤其是格式化命令风险很高，必须确认目标设备不是系统盘或已有数据盘。

---

## 五、常用命令速查

```bash
lsblk
lsblk -f
lsblk -d
lsblk -d -o NAME,SIZE,MODEL,TYPE
lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINTS
lsblk -p
lsblk /dev/sda
lsblk /dev/nvme0n1
lsblk -J
lsblk -J -o NAME,SIZE,FSTYPE,MOUNTPOINTS
lsblk -m
```

---

## 六、总结

`lsblk` 是查看 Linux 块设备结构的核心命令。它最适合回答这几个问题：

- 机器上有哪些磁盘？
- 每块磁盘有哪些分区？
- 分区是否已经格式化？
- 分区挂载到了哪里？
- UUID 是什么？

日常使用中建议优先掌握：

```bash
lsblk
lsblk -f
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINTS
```

在磁盘挂载、扩容和故障排查前，先用 `lsblk` 看清设备关系，可以明显降低误操作风险。