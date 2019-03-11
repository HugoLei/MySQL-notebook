# Insert Buffer 为了解决什么问题？
insert有两种情况：
* 顺序 insert
* 随机 insert

Insert Buffer 为了提升顺序 insert 的性能。

# InnoDB 中的顺序 insert
> insert分两步，先插入聚集索引，再插入 secondary index

### 递增主键聚集索引
InnoDB 中主键是行唯一标识符。
若是递增主键，并且未指定主键值。（会有其他主键？）
则在主索引（是个聚集索引）上的 insert 其实是顺序 insert

### 非聚集的辅助索引
