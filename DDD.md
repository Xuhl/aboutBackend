# DDD 
## 应用架构
应用架构分层：`interface`、`Application`、`Domain`、`infrusture` 四层

## 领域对象
`DO`：Data Object
`DP`: Domain primitives
'DTO': Data trans object
'Entity' : entity

## Repository 设计
封装`Entity`,避免直接操作底层DAO 
面对`多聚合根`的情况，可以通过`change-tracking` 技术，采用snapshot，追踪对象的变化
