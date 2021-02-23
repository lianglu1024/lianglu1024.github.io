# MySQL数据库操作规范

## 一、DQL

1. 手动查询数据尽量在从库上进行，除非对数据实时性要求很高或无实例从库可查主库，每个查询sql末尾必须加limit限制查询行数在1000以内，禁止全表扫描
2. 线上环境禁止使用hint强制改变MySQL执行计划，如force index、ignore key、straight join等
3. SELECT … WHERE … IN (…)，IN值不能过多，限制在100以内，否则会增加底层扫描，影响查询效率
4. WHERE条件中等号左右两边的字段类型必须一致，否则无法利用索引
5. WHERE子句中禁止只使用全模糊的LIKE条件进行查找，必须有其他等值或范围查询条件
6. 索引列禁止使用函数或表达式，否则无法利用索引。如where length(name)='Admin'或where user_id+2=10023
7. SELECT语句避免使用UNION，推荐使用UNION ALL，并且子句个数限制在5个以内。UNION ALL不需要去重，节省数据库资源，提高性能
8. or语句慎用
9. 建议使用合理的分页技术以提高操作的效率，分页查询当limit起点较高时，可先用过滤条件进行过滤。如select a,b,c from t1 limit 1000,20；优化为: select a,b,c from t1 where id>1000 limit 20;
10. 不建议使用子查询，建议将子查询SQL拆开结合程序多次查询，或使用join来代替子查询
11. 多表join不要超过3个表，在多表join中，尽量选取结果集较小的表作为驱动表来join其他表
12. 减少使用order by，和业务沟通能不排序就不排序或将排序放到程序端去做。order by、group by、distinct这些语句都比较耗费CPU
13. order by、group by、distinct尽量利用索引直接检索出排序好的数据，且必须加limit，限制行数控制在1000以内
14. 在设计面向消费者业务技术方案时，底层DB不允许有join操作，如果实在需要必须满足两个前提：1 业务量不可能大；2 缓存机制

## 二、DML

1. INSERT/UPDATE/DELETE语句禁止一次性操作大量数据，必须严格控制数量，单次操作不超过2000，进行必要的sleep，做到少量多次，一次性提交过多的记录会导致线上I/O紧张
2. INSERT语句需要指定具体字段名称，禁止写成INSERT INTO t1 VALUES(…)
3. 禁用insert into …on duplicate key update… /insert ignore……，在高并发环境下，容易发生交叉死锁
4. UPDATE/DELETE禁止使用关联子查询，如update t1 set … where name in(select name from user where…)
5. UPDATE/DELETE禁止联表更新，如update t1,t2 where [t1.id](http://t1.id/)=[t2.id](http://t2.id/)…
6. UPDATE/DELETE禁止使用join，比如update t1 join t2…
7. UPDATE/DELETE禁止使用模糊查询like

## 三、DDL

1. 禁用procedure、function、trigger、views、event、外键约束，因为它们会消耗数据库资源，降低数据库集群可扩展性。推荐都在程序端实现
2. 添加索引前第一步先确认需要加索引的字段是否已经存在索引，禁止添加重复的索引降低性能、浪费资源
3. 清空表数据请使用truncate，而不用delete，delete速度慢、会造成主从延迟、且产生大量binlog
4. 新建库、新建表、增加字段、添加索引务必参考《[MySQL数据库设计规范](http://wiki.yuceyi.com/pages/viewpage.action?pageId=38644839)》
