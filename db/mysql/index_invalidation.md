# MySQL 索引失效问题

问：有一个用户表，对age字段加了索引，运行select * from user where age = 10，解释器提示该语句未使用索引，可能是什么情况


## MySQL索引失效原因

### 1. 查询条件中有or

```
SELECT name,age,address FROM user where name = '光头强' or age=9
```

### 2. like查询以'%'开头

```
SELECT name,age,address FROM user where name like '%头强' 
```

如果想让以`%`开头仍然使用索引，则需要使用覆盖索引，即只查询带索引字段的列

```
SELECT name FROM user where name like '%头强' 
```

### 3. 对查询的列上有运算或者函数的

```
SELECT name,age,address FROM user where substr(name,-2)='头强'
```

### 4. 如果列类型是字符串，那一定要在条件中将数据使用引号引用起来,否则不使用索引

```
explain SELECT name,age,address FROM user where name = 10
```

mysql有个类型转换规则就是将“字符转成数字”，所以以上sql就等价于这样：

```
SELECT name,age,address FROM user where cast(name as signed)= 10
```

### 5. 左连接查询或者右连接查询查询关联的字段编码格式不一样

### 6. 如果mysql估计使用全表扫描要比使用索引快,则不使用索引

### 7. 连接查询中，按照优化器顺序的第一张表不会走索引

### 8. 如果查询中没有用到联合索引的第一个字段，则不会走索引

### 9. 不适合键值较少的列（重复数据较多的列）

假如索引列TYPE有5个键值，如果有1万条数据，那么 WHERE TYPE = 1将访问表中的2000个数据块。

再加上访问索引块，一共要访问大于200个的数据块。

如果全表扫描，假设10条数据一个数据块，那么只需访问1000个数据块，既然全表扫描访问的数据块

少一些，肯定就不会利用索引了。

----
## 参考资料
https://yq.aliyun.com/articles/388463

https://www.jianshu.com/p/3ccca0444432