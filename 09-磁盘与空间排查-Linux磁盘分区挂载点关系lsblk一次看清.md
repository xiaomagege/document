# 09｜磁盘与空间排查｜Linux 磁盘、分区、挂载点关系，lsblk 一次看清

磁盘排查最怕的一件事，是还没看清设备关系就开始分区、格式化或挂载。

`lsblk` 的作用就是先把“磁盘、分区、文件系统、挂载点”这条链路摆出来，让你知道现在系统到底识别到了什么。

如果只想先记 3 条：

- 新盘、扩容、挂载前先看 `lsblk -f`
- `MOUNTPOINTS` 为空，只说明没挂载，不代表磁盘不可用
- 执行 `fdisk`、`mkfs`、`mount` 前，先确认目标设备不是系统盘和已有数据盘

## 一、什么时候先看 `lsblk`

这些场景都应该先看 `lsblk`：

- 云服务器新挂了一块数据盘
- 扩容后想确认系统是否识别
- 不确定 `/data` 对应哪个分区
- 准备分区、格式化、挂载
- 配置 `/etc/fstab` 前需要确认 UUID
- 怀疑挂载点和设备关系不一致

先执行：

```bash
lsblk -f
```

这条命令通常比单独 `lsblk` 更适合排查，因为它会同时显示文件系统类型、UUID 和挂载点。

## 二、输出里重点看什么

常见字段里，排查时最值得看的是：

- `NAME`：设备名
- `SIZE`：设备大小
- `TYPE`：设备类型
- `FSTYPE`：文件系统类型
- `UUID`：文件系统 UUID
- `MOUNTPOINTS`：挂载点

可以用自定义字段让输出更聚焦：

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,UUID,MOUNTPOINTS
```

如果你要执行后续操作，建议显示完整路径：

```bash
lsblk -p -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

## 三、几种常见读法

### 1. 看到磁盘，但没有分区

比如只看到 `/dev/vdb`，没有 `/dev/vdb1`。

这通常说明系统识别到了新磁盘，但还没有分区。

下一步可能是：

```bash
fdisk /dev/vdb
```

但动手前一定先确认这不是系统盘，也不是已有数据盘。

### 2. 看到分区，但没有 `FSTYPE`

这通常说明分区存在，但还没有格式化成文件系统。

下一步可能是：

```bash
mkfs.xfs /dev/vdb1
```

格式化是高风险动作，只能对确认无数据的新分区执行。

### 3. 看到 `FSTYPE`，但 `MOUNTPOINTS` 为空

这说明分区有文件系统，但当前没有挂载到目录。

下一步通常是：

```bash
mkdir -p /data
mount /dev/vdb1 /data
```

### 4. 看到挂载点，但空间还是不对

这时要配合：

```bash
df -Th
mount | grep /data
```

确认当前路径到底挂到了哪个设备上。

## 四、常用命令

```bash
lsblk
lsblk -f
lsblk -d
lsblk -p
lsblk -o NAME,SIZE,TYPE,FSTYPE,UUID,MOUNTPOINTS
lsblk -d -o NAME,SIZE,MODEL,TYPE
lsblk /dev/vdb
lsblk -J -o NAME,SIZE,FSTYPE,MOUNTPOINTS
```

它们分别适合：

- `lsblk`：看设备树
- `lsblk -f`：看文件系统、UUID、挂载点
- `lsblk -d`：只看磁盘本身
- `lsblk -p`：显示完整设备路径
- `lsblk -o ...`：只看关心字段
- `lsblk -J`：给脚本或自动化工具读取

## 五、和 `df`、`fdisk`、`mount` 怎么配合

你可以这样分工：

- `lsblk`：看设备结构
- `df`：看文件系统空间
- `fdisk`：改分区表
- `mount`：把文件系统挂到目录

新增数据盘时，常见顺序是：

```bash
lsblk -f
fdisk -l
fdisk /dev/vdb
lsblk -f
mkfs.xfs /dev/vdb1
mkdir -p /data
mount /dev/vdb1 /data
df -Th
```

## 六、最常见的误区

### 1. 看到磁盘就直接格式化

`lsblk` 看到磁盘，只说明设备存在，不代表它是空盘。

格式化前必须确认：

- 设备名
- 大小
- 是否已有分区
- 是否已有文件系统
- 是否已经挂载

### 2. 把整盘设备和分区设备混淆

`/dev/vdb` 是整块磁盘，`/dev/vdb1` 是分区。

通常：

- 分区操作针对整盘设备
- 格式化和挂载通常针对分区设备

### 3. 只看设备名，不看 UUID

云服务器或虚拟化环境里，设备名可能变化。配置 `/etc/fstab` 时，更推荐使用 UUID。

## 七、结论

`lsblk` 是磁盘操作前的第一张地图。

在分区、格式化、挂载之前，先用 `lsblk -f` 看清设备、分区、文件系统和挂载点关系，可以显著降低误操作风险。
