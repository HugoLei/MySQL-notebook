# Insert Buffer 为了解决什么问题？
insert有两种情况：
* 顺序 insert
* 随机 insert

Insert Buffer 为了提升顺序 insert 的性能。

# InnoDB 中的顺序 insert
### 递增主键聚集索引
InnoDB 中主键是行唯一标识符。
通常是递增主键。（会有其他主键？）
因此在主索引（是个聚集索引）上的 insert 其实是顺序 insert