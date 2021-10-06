# PG
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
### 索引分类
1. B-tree: 最常用索引，适合等值和范围查询
2. Hash: 处理简单的等值查询
3. Gist: 不是单独的索引类型，而是一种架构，这种架构之下，可以实现很多不同的索引策略
4. SP-Gist： space-partioned Gist ,空间分区索引
5. GIN: 反转索引，可以处理包含多个键的值，如数组等。
```select * from contact where phone @> array['134256789'::varchar(32)]```

### 并发创建索引

通过并发创建索引，降低对现网用户操作冲击
```create index concurrently idx_name on table_name(xxx) ```

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
