# Insert Buffer 为了解决什么问题？（优化随机写）
insert有两种情况：
* 顺序 insert
* 随机 insert

Insert Buffer 为了提升`随机` insert 的性能。

# InnoDB 中的顺序 insert
> insert分两步，先插入聚集索引，再插入 secondary index

### 主索引：递增主键聚集索引
InnoDB 中主键是行唯一标识符。
若是递增主键，并且未指定主键值。（若主键指定了值或者是自定义主键，则是随机写）
则在主索引（是个聚集索引）上的 insert 其实是顺序 insert

### 辅助索引：非聚集，且not unique
1. 若主键是递增的，那在主索引上还是顺序 insert
2. 非唯一的辅助索引 insert，因为是B+树，在叶节点上的插入是`随机的`

> “较为顺序”：在辅助索引上按照时间递增插入，非完全随机，较为顺序

# Insert Buffer
* 辅助索引
* not unique

上述条件下的 insert/update，写逻辑如下：

1. 若对应的索引节点在缓存中，则直接写入
2. 若不在缓存中，则先放入 Insert Buffer 中。（而不是去从 disk 读对应的页）
3. 按照一定规则 merge InsertBuffer + 索引节点（通常是将多个 insert/update合并在一个操作中）

> 因为是先写在 Insert Buffer 中，因此存在丢失的风险

### 辅助索引：unique
unique 索引，因为要判断唯一性，这个判断过程肯定要随机读 B+ 树的节点。所以这种情况使用 Insert Buffer 无意义。

# 实现原理
使用 B+树缓存，一棵全局 Insert Buffer B+ 树。
非叶节点：
* 表 ID + 辅助索引页的 offset

叶节点：
* 表 ID + 辅助索引页的 offset
* 插入时的顺序
* 要放到辅助索引上的内容

Insert Buffer Bitmap：
* 标记某个辅助索引的页是否有insert buffer
* 标记某个辅助索引的页的剩余空间

merge：
* 辅助索引页被读取到缓冲池（如 select 时读取该页，检查 bitmap，看是否有 insert buffer 需要合并，合并时将多个操作一次 merge）
* bitmap 追踪到该页已无足够可用空间（强制读取对应的页，进行 merge）
* Master Thread 周期性 merge（但非全量 merge）


