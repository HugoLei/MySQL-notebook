# 针对数据对象的 log 有两种思路：记录最新的数据值，记录操作

# 物理日志：记录数据的最新值
![](/assets/page 修改.png)
如图：


```
物理日志的格式如下
Page 42: 367,2;before:'Ke';after:'ca'
格式说明：
Page 42 // 页号
367 // offset
2 // length
before:'Ke' // old value
after:'ca' // new value
```

# 逻辑日志：记录操作
如上图：


```
逻辑日志格式如下
update(6,'Kemera'=>'camera')
格式说明：
update // 操作类型
6 // 更新的记录的 id
'Kemera' // old value
'camera' // new value
```

# 物理日志 vs 逻辑日志
||日志大小|数据依赖|恢复|
|:--|:--|:--|
|物理日志|大|无|直接通过日志覆盖，且幂等|
|逻辑日志|小|依赖初始数据||

逻辑日志的问题：
1. 一条逻辑日志可能对应多个操作，如 insert 会更新多个索引，会有部分操作成功，部分操作失败的情况，这时候页内数据不一致
2. 逻辑日志没有记录初始值，所以无法恢复
> 数据快照提供初始数据 + 逻辑日志，无法恢复吗？

# InnoDB 日志方式（物理到页+页内逻辑）
Innodb日志方式

采用逻辑与物理相结合 物理到Page Page内部是逻辑的（space id, page no, operation code, data）
虽然采取此方式可以解决部分执行问题 但是数据一致性还是无法解决（因为对于page内的信息使用逻辑日志进行记录，所以当出现坏块时，并不能解决）

日志格式：
Page 42: update(6,'Kemera'=>'camera')
Page // 物理页地址
update // 逻辑操作



