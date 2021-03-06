# PG
## SQL 执行顺序
 sql 按如下顺序执行
```
(7)select (8) DISTICT(11) <SELECT LIST>
(1) from <left table>
(3) <join type> JOIN <right table>
(2) on <join condition>
(4) where <where condition>
(5) group by <group by list>
(6) having <having condition>
(9) order by <order by list>
(10) limit <limit offset>
```
## 约束
- 检查约束,  age int CHECK(age >=0 and age<= 150)
- 非空约束 , age int not null
- 唯一约束,  name varchar(20) UNIQUE
- 主键 ,
- 外键
## 分区表
  PG 的分区表建立在集成表的机制上，通过数据分区，可以获得收益如下：
  - 删除历史数据更快，直接drop 掉分区表就可以了
  - 分区之后，对应查询可以集中在少数分区上，索引可以直接缓存在内存中，查询性能极大的提升
  - 很少用到的数据可以放到慢速介质上，或者日落
  
  分区表使用步骤：
 1. 创建父表
 2. 创建子表，作为分区表
 3. 在分区表上增加约束，定义每个分区允许的键值
 4. 对每个分区表，增加关键字段的索引
 5. 定义触发器或者规则，把插入主表的数据重定向到分区表上。（规则的话，每次插入都会检查，存在性能损耗）
 6. 确保constraint_exclusion 是打开的
## 索引
**索引原理：** https://www.cnblogs.com/ricklz/p/12813080.html
### 索引分类
1. B-tree: 最常用索引，适合等值和范围查询
2. Hash: 处理简单的等值查询
3. Gist: 不是单独的索引类型，而是一种架构，这种架构之下，可以实现很多不同的索引策略
4. SP-Gist： space-partioned Gist ,空间分区索引
5. GIN: 反转索引，可以处理包含多个键的值，如数组等。
```select * from contact where phone @> array['134256789'::varchar(32)]```

### 组合索引
- 在PG 数据库中，组合索引需要将选择性好的列放在前面。因为当没有INDEX_SKIP_SCAN 技术，当查询条件缺失选择性好
### 位图索引
当根据非聚集索引，范围查询时，如果直接去回表读取数据，就会产生大量随机性的IO ，为了避免频繁的回表造成的随机IO，读取完非聚集索引上符合条件的key值之后，对key值对应的聚集索引键排序，然后根据排序后的聚集索引键顺序地回表，从而避免大量的随机性IO。因为MySQL的Innodb表都是聚集表，那么排序后，是顺序性的映射到聚集索引的page，从而避回表过程中的随机性IO。
在PG数据库中，采用同样的原理来解决此问题。pg 中bitmap scan的作用就是通过建立位图的方式，将回表过程中对标访问随机性IO的转换为顺行性行为，从而减少查询过程中IO的消耗。
- oracle 数据库中，可以物化位图索引
- pg 中的位图索引是查询过程总动态创建，执行完之后就会丢弃掉

### 并发创建索引

通过并发创建索引，降低对现网用户操作冲击
```create index concurrently idx_name on table_name(xxx) ```

### 多版本控制
通常实现多版本控制有两种方式：
- 写入新数据时，把旧数据挪到回滚段中，其他人读取数据时，从回滚段中再把数据读取出来
- 写新数据时，旧数据不删除，而是把新数据插入
PG 采用的是第二种方式。
在多版本控制过程中，PostgreSQL主要就是通过t_xmin，t_xmax，cmin和cmax，ctid，t_infomask来唯一定义一个元组（t_xmin，t_xmax，cmin和cmax，ctid实际上也是一个表的隐藏的标记字段）
- t_xmin 存储的是产生这个元组的事务ID，可能是insert或者update语句
- t_xmax 存储的是删除或者锁定这个元组的事务ID
- t_cid 包含cmin和cmax两个字段，分别存储创建这个元组的Command ID和删除这个元组的Command ID
- t_xvac 存储的是VACUUM FULL 命令的事务ID

PG 实现了HOT （HEAP ONLY tuple） 机制，。它存在的目的就是消除表非索引列更新对索引影响，它必须满足两个条件
1. 索引字段的值不变。(其中任意一个索引字段的值发生了变化，则所有索引都需要新增版本)  
2. 新的版本与旧的版本在同一个HEAP PAGE中。  

### 事务隔离
4个事务隔离级别：
- 读未提交
- 读已提交
- 重复度
- 串行化

3个异常情况：
- 脏读
- 不可重复度
- 幻读

### 锁
https://www.modb.pro/db/26462

锁分表锁和行锁，其中表锁分为8种模式

最普通的共享锁
- Share 读
- Exclusive

多版本读写锁
- Access share ， 只与Access Exclusive 模式冲突
- Access Exclusive，最强的写锁，DROP TABLE|TRUNCATE TABLE|VACUUM FULL |REINDEX|CLUSTER 会请求这个锁

意向锁，处理表锁和行锁之间的关系
- Row share， SELECT FOR UPDAE 或者SELECT FOR SHARE 请求该锁
- Row Exclusive，INSERT|UPDATE|DELETE 请求自动加上这个锁

更严格的意向锁，意向锁间不会参数冲突，意向锁和排他锁之间也不会产生冲突
- Share update Exclusive，create index concurrently 和 不带FULL 的vacuum
- Share Row Exclusive  ，PG 内部未使用


## 常用的SQL 
1. 查询长事务

``` select * from pg_stat_activity where state<>'idle' and pg_backend_pid() != pid and (backend_xid is not null or backend_xmin is not null ) and extract(epoch from (now() - xact_start))  > 60s order by  xact_start;```
