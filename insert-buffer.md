# Insert Buffer 为了解决什么问题？
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

### 辅助索引：非聚集，且非 unique
1. 若主键是递增的，那在主索引上还是顺序 insert
2. 非唯一的辅助索引 insert，因为是B+树，在叶节点上的插入是`随机的`

