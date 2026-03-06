> 本文整理了我们关于 Spark SQL 中 **group by** 与 **join** 的性能讨论：它们在执行计划（physical plan）中的典型表现形式、`HashAggregate / ObjectHashAggregate / SortAggregate` 的区别，以及在不同数据形态下谁更可能成为瓶颈。

---

## 1. 先给结论：group by 与 join 谁更贵？

经验上常见的排序是：

- **大表 Shuffle Join**（例如 `SortMergeJoin`）通常最贵  
- **普通 group by（HashAggregate）**通常次之  
- **broadcast join（BroadcastHashJoin）**往往很便宜  
- **但**：一旦 group by 变成 **高基数 / 严重倾斜 / ObjectHashAggregate / spill 很多**，它可能比 join 更贵

一句话总结：

> **join 通常更容易因为 shuffle 与结果膨胀变贵；group by 通常因为高基数、倾斜、spill、对象聚合而变贵。**

---

## 2. group by 在执行计划里长什么样？

你观察到的典型结构是：

- `ObjectHashAggregate`
- `Exchange hashpartitioning(...)`
- `ObjectHashAggregate`

这其实是 Spark 聚合的常规“两阶段聚合 + shuffle”结构（无论是 HashAggregate 还是 SortAggregate，本质逻辑都类似）：

### 2.1 第一个 Aggregate：Partial / 局部聚合（map-side）
- 在每个输入分区内部先做一轮聚合
- 目标是：**尽量减少后续 shuffle 的数据量**

如果同一个分区内 key 重复很多，这一步很赚；如果 key 基数高、几乎不重复，这一步收益很小。

### 2.2 Exchange hashpartitioning：Shuffle 重分区
- 按 group key 做 hash 分区
- 保证 **相同 key 的数据进入同一个 reducer 分区**

这一步往往是 group by 成本的大头：网络传输 + shuffle 写盘/读盘 + 排队。

### 2.3 第二个 Aggregate：Final / 最终聚合（reduce-side）
- reducer 端合并 partial 结果
- 如果分组数巨大或存在倾斜，仍可能出现 **内存压力、spill、GC 抖动**

---

## 3. join 在执行计划里长什么样？

join 的成本差异极大，关键取决于 **join 类型**：

### 3.1 Broadcast join：`BroadcastHashJoin`
- 小表被广播到每个 executor
- 大表通常不需要按 join key shuffle（或很少）
- **通常非常快**，经常比一次大 shuffle 的 group by 还轻

### 3.2 Shuffle join：`SortMergeJoin` / `ShuffledHashJoin`
- 两边都需要按 join key `Exchange hashpartitioning`
- 再做 merge 或 hash 连接
- **网络与磁盘压力大**，大表对大表时通常是全链路最贵算子之一

---

## 4. SortAggregate、HashAggregate、ObjectHashAggregate 的区别

它们都是聚合算子，区别在于 **聚合策略** 与 **聚合状态的存储方式**。

### 4.1 HashAggregate（通常最快）
- 使用高效的 Tungsten/UnsafeRow 结构维护 `key -> aggBuffer` 的 hash map
- 通常代码生成友好（WholeStageCodegen）
- **优点**：一般情况下最快  
- **缺点**：高基数或倾斜时 hash map 变大 → spill / OOM 风险上升

### 4.2 ObjectHashAggregate（通常更重）
- 仍是 hash 聚合，但聚合状态使用 **Java 对象**
- 对象开销 + GC 压力大 → 更容易 spill，通常更慢

常见触发因素（很常见，但不止这些）：
- `collect_list / collect_set`
- 聚合状态涉及复杂类型（array/map/struct）
- 自定义 UDAF
- 复杂表达式导致无法走更“原生”的 HashAggregate

### 4.3 SortAggregate（更稳但可能更慢）
- 基于排序：先按 key 排序，再扫描聚合
- **优点**：内存压力小，更稳（高基数时更不容易 OOM）  
- **缺点**：排序 CPU/IO 成本高，大数据时也会 spill

---

## 5. 先 group by 再 join vs 先 join 再 group：谁更贵？

在 **语义等价** 的前提下，通常：

- **先 group by 再 join 更省**
  - 聚合先“降维”（减少行数/数据量）
  - 后续 join 参与 shuffle/比较的数据更少

- **先 join 再 group 往往更贵**
  - join 可能放大行数（1:N 或 M:N）
  - 再 group 等于在更大的数据上 shuffle + 聚合

但以下场景可能必须或更适合“先 join 再 group”：
- 分组维度来自 join 后的列（不 join 拿不到 group key）
- join 是 broadcast join（成本很低）
- join 近似 1:1，输出不膨胀
- 需要先 join 做过滤/补齐，才能保证聚合口径正确（尤其 outer join）

> 注意：很多时候两种写法 **结果并不等价**，尤其涉及 outer join、过滤条件位置、distinct、以及聚合列来自哪一侧表。

---

## 6. 不同情况下：HashAggregate / ObjectHashAggregate / Join 谁更可能更贵？

下面是实战中最常用的判断框架。

### 6.1 Join 更可能更贵的情况
- 大表对大表 **shuffle join**（`SortMergeJoin`）  
- join 输出膨胀明显（1:N、M:N）
- join key 倾斜导致少数分区巨大、单 task 超慢
- join 前缺乏有效过滤/裁剪列，shuffle 体积大

### 6.2 Group by 更可能更贵的情况
- **高基数 group by**：不同 key 数接近行数
  - partial aggregate 几乎帮不上忙
  - shuffle 数据接近全量
  - 聚合状态巨大 → spill
- 严重倾斜：少数 key 极热导致 reducer 分区超大
- 使用复杂聚合触发 **ObjectHashAggregate**
  - 对象多 + GC 高 + spill 多

### 6.3 HashAggregate vs SortAggregate：哪个更贵？
- 内存足且基数适中：**HashAggregate 通常更快**
- 高基数/容易 spill：**SortAggregate 更稳**，整体可能更快（避免 hash map 爆炸带来的 spill/GC）

---

## 7. 为什么会看到 `ObjectHashAggregate -> Exchange -> ObjectHashAggregate`？

这通常意味着：
1. 你正在走“两阶段聚合”：partial + final  
2. 需要按 key shuffle（`Exchange hashpartitioning`）  
3. 聚合函数或数据类型使 Spark 选择了 **ObjectHashAggregate**

因此当你发现 group by 比 join 更贵，常见根因之一就是：
- **ObjectHashAggregate + 高基数/倾斜 → 大量 spill + GC → 性能崩**

---

## 8. 用 Spark UI 快速定位瓶颈（推荐看这三类指标）

1) **Shuffle Read/Write**
- `Exchange` 对应的 shuffle 读写量越大，网络+磁盘越重  
- shuffle join 通常两边都 shuffle；group by 通常是一边 shuffle

2) **Spill（Memory/Disk Spill）**
- spill 多：聚合/排序状态顶不住（高基数、对象聚合、分区过大）

3) **Task 时间分布**
- 极少数 task 特别慢：高概率是 **数据倾斜**（group 或 join 都可能）

---

## 9. 总结

- **默认**：大表 shuffle join 通常更贵  
- **例外**：高基数/倾斜/复杂聚合触发 ObjectHashAggregate 时，group by 可能更贵  
- **broadcast join** 常常很便宜，甚至比一次大 shuffle 的 group by 更轻  
- “先 group 再 join”通常更省，但必须满足 **语义允许且聚合真能降维**

---

## 附：术语速查

- **高基数 group by**：group key 的不同取值数非常多，接近输入行数  
- **倾斜（skew）**：少数 key 占据大量数据，导致少数 reducer 分区/任务异常慢  
- **spill**：内存不够，将中间数据写到磁盘（显著拖慢）  
- **broadcast join**：小表广播到各 executor，本地 join，通常很快  
- **shuffle join**：两边按 key 重分区，网络+磁盘开销大