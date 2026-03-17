# linux常用命令：rpm命令详解

在 CentOS、Red Hat、Rocky Linux、AlmaLinux 等 RPM 系发行版中，`rpm` 是最基础的软件包管理命令之一。无论是安装离线包、查询已安装软件、查看文件归属，还是卸载指定软件，`rpm` 都非常常用。

---

## 一、`rpm` 是什么？

`rpm` 是 RPM Package Manager 的命令行工具，用于管理 `.rpm` 软件包。

它主要用于：

- 安装 RPM 软件包
- 升级软件包
- 卸载软件包
- 查询软件包信息
- 校验软件包和文件

说明：

- `rpm` 更偏底层包管理
- `yum` / `dnf` 会自动处理依赖，而 `rpm` 本身一般不会自动解决依赖问题

---

## 二、常见使用语法与重点参数

```bash
rpm [选项] [软件包]
```

重点参数说明

- `-i`：安装软件包
- `-U`：升级软件包，若未安装则执行安装
- `-e`：卸载软件包
- `-q`：查询软件包
- `-a`：查询所有已安装软件包
- `-i`（配合 `-q` 使用时）：查看软件包详细信息
- `-l`：查看软件包安装了哪些文件
- `-f`：查询某个文件属于哪个软件包
- `-p`：针对未安装的 `.rpm` 文件查询
- `-v`：显示详细信息
- `-h`：显示进度条

---

## 三、常用命令与使用场景

### 1. 查看系统中所有已安装包

```bash
rpm -qa
```

适合排查某个软件是否已经安装。

### 2. 搜索指定软件包

```bash
rpm -qa | grep nginx
rpm -q nginx
```

一个适合模糊搜索，一个适合精确判断包是否存在。

### 3. 安装或升级 RPM 包

```bash
rpm -ivh nginx.rpm
rpm -Uvh nginx.rpm
```

- `-ivh`：适合首次安装
- `-Uvh`：适合升级，若未安装通常也会执行安装

### 4. 卸载 RPM 包

```bash
rpm -e nginx
```

卸载时一般填写包名，而不是 `.rpm` 文件名。

如需忽略依赖强制卸载，可使用：

```bash
rpm -e --nodeps nginx
```

这个命令风险较高，只适合明确知道依赖关系时使用。

### 5. 查看软件包详细信息

```bash
rpm -qi bash
```

通常可以看到版本、发布时间、安装时间、说明等元数据。

### 6. 查看软件安装了哪些文件

```bash
rpm -ql nginx
```

适合排查配置文件、二进制文件、日志目录安装到了哪里。

### 7. 反查某个文件属于哪个包

```bash
rpm -qf /usr/bin/vim
```

这个场景很常用，尤其适合排查“某个命令来自哪个包”。

### 8. 查看未安装 RPM 文件的信息

```bash
rpm -qpi nginx.rpm
rpm -qpl nginx.rpm
```

适合在离线安装前，先确认包信息和文件列表。

### 9. 校验已安装软件包

```bash
rpm -V nginx
```

可用于检查安装后的文件是否被篡改、缺失或属性发生变化。

---

## 四、常见问题与建议

### 1. 为什么 `rpm -ivh` 安装失败？

最常见原因有：

- 缺少依赖包
- 软件包架构不匹配
- 软件包已安装同名版本
- 文件冲突

可以先看报错信息，再决定是补依赖还是改用：

```bash
rpm -Uvh package.rpm
```

如果依赖较多，更建议使用 `yum localinstall` 或 `dnf install`。

### 2. `-ivh` 和 `-Uvh` 有什么区别？

- `-ivh`：安装，适合首次安装
- `-Uvh`：升级，已安装旧版本时更常用

如果目标包可能已存在，优先考虑 `-Uvh`。

### 3. 为什么卸载时提示依赖冲突？

说明有其他软件依赖当前包。此时不要直接强制卸载，先确认依赖关系，必要时再考虑：

```bash
rpm -e --nodeps 包名
```

这个命令有破坏系统依赖的风险。

### 4. `rpm` 和 `yum` / `dnf` 的区别是什么？

- `rpm`：底层包管理工具，适合本地包安装、查询和精细控制
- `yum` / `dnf`：高级包管理工具，能自动处理仓库和依赖

简单说：

- 查包、查文件归属：`rpm` 很方便
- 装包、升级依赖较复杂的软件：优先 `yum` 或 `dnf`

### 5. 如何查看包是否被修改过？

可以使用：

```bash
rpm -V 包名
```

如果有输出，说明部分文件可能发生了内容、权限、时间戳等变化。

---

## 五、常用命令速查

```bash
rpm -qa
rpm -qa | grep nginx
rpm -q nginx
rpm -qi nginx
rpm -ql nginx
rpm -qf /usr/bin/vim
rpm -ivh package.rpm
rpm -Uvh package.rpm
rpm -e nginx
rpm -qpi package.rpm
rpm -qpl package.rpm
rpm -V nginx
```

---

## 六、总结

`rpm` 是 RPM 系 Linux 发行版中最基础的软件包管理命令。建议至少熟练掌握以下几条：

```bash
rpm -qa
rpm -qa | grep 包名
rpm -ivh package.rpm
rpm -e 包名
rpm -qi 包名
rpm -qf 文件路径
```

掌握这些常用写法后，大多数“查看软件是否安装、离线安装 rpm 包、查询文件归属、卸载软件、核对包信息”的场景都能快速处理。
