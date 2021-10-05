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

## 常用的SQL 
1. 查询长事务

``` select * from pg_stat_activity where state<>'idle' and pg_backend_pid() != pid and (backend_xid is not null or backend_xmin is not null ) and extract(epoch from (now() - xact_start))  > 60s order by  xact_start;```
