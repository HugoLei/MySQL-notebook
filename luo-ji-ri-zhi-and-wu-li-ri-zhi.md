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


