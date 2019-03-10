# Note



MySQL 是单进程多线程



MySQL 存储引擎基于表，不是数据库



OLTP（Online Transaction Processing）在线事务处理（如 MySQL）



MySQL 最重要的特征：插件式存储引擎



MySQL 有很多种存储引擎（可自己开发），特性各异，有的支持全文检索，有的支持事务等



InnoDB 支持 OLTP，促使 MySQL 的商业化蓬勃发展



# ACID 事务



atomicity：原子性，要么全部执行，要么全部不执行。



consistency：一致性，事务执行必须保证系统状态一致，如转账，这边减10块，那边就加10块。



isolation：隔离性，事务与事务的执行是隔离的（比如串行化，就保证事务是隔离执行的）



durability：持久性，事务对数据的更改持久保存，不会被无故回滚



# 缓冲池



### LRU （Least Recent Used最少最近使用）更新算法



基本原理：频繁使用的放在前部，最少使用的放在尾部，清理时先清理尾部



### InnoDB LRU 改进

##### 预读和扫描对缓存的影响

1. 预读对缓存的影响

1. 如果预读的页也放在 LRU 头部的话，会把真正的 hot 数据挤出缓存，另一方面这些预读的页后续可能根本就不会再访问。

2. 扫描对缓存的影响

1. 扫描大量更新缓存，但是扫描时数据往往只用一次，结果真正 hot 的数据被挤出缓存



### midpoint 策略



!\[\]\(/assets/innodb-buffer-pool-list.png\)



LRU 列表被 midpoint 分成两部分，前半部分是 young 列表，后半部分是 old 列表。midpoint 的位置默认是当前列表的5/8处。

##### midpoint解决预读对缓存的影响

* 当从磁盘读取新页时，先插入到 midpoint 位置（这个新页可能是SQL 直接要用的，也可能是依据局部性原理预读的）

* 当真正使用一个old 区域的缓存页时，把该页转成 young，也既从 old 拿出来放在 young 的头部。

* 真正使用一个 old 区域的页时才发生 old-young 的转换

* 如果是预读的，则没有这步操作，也既预读的页一直在 old，不会影响 young



##### midpoint + innodb_old_blocks_time 解决扫描对缓存的影响

* 从磁盘读取的页先放在 midpoint 位置

* 刚从磁盘读取的页有两种情况：

* 立马会被 SQL 访问（cache miss）

* 预读

* 若立马被 SQL 访问，该页会被从 old 放入到 young 的头部

* 但是这个转换有个条件：该页被第一次 SQL 访问后，还需要在 old 待一段时间innodb_old_blocks_time，这个时间之后才会发生 old 到 young 头部的转换。

* 在这段时间里，SQL 对该页的访问还在 old 区

* 若时间还没到，这个页就在 old 区里被清理了



> innodb\_old\_blocks\_time有效地缓解了扫描对缓存的影响，因为一个页必须在 old 区里待足够的时间，才能移动到 young 里，而扫描时页很快就会被踢出 old，因此可以减少 scan 对缓存的影响。



# 伙伴算法



存储空间管理算法，目的是减少碎片



