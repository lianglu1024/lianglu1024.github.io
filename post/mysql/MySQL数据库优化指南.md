# MySQL数据库优化指南

## 一、SQL优化

### 1.1 查询优化

1. SELECT语句指定具体字段名称，杜绝使用SELECT * 读取全部字段。尤其是当表中存在TEXT/BLOB 大列时就会是灾难，可能本来不需要读取这些列，但因为偷懒写成 SELECT * 导致内存buffer pool被这些“垃圾“数据把真正需要缓冲起来的热点数据给洗出去了

2. SELECT语句避免使用UNION，推荐使用UNION ALL，并且子句个数限制在5个以内。因为UNION ALL不需要去重，节省数据库资源，提高性能

3. SELECT … WHERE … IN (…)，IN值不要过多，限制在500以内，会增加底层扫描，影响查询效率。因为哪怕是基于索引的条件过滤，如果优化器意识到总共需要扫描的数据量超过30%时，就会直接改表执行计划为全表扫描，不再使用索引

4. SELECT查询时必须加上limit限制查询行数，避免慢查询

5. 生产环境禁止使用hint，hint是用来强制改变MySQL执行计划，如force index、ignore key、straight join等，但随着数据量变化我们无法保证自己当初的预判是正确的，因此要充分相信MySQL优化器

6. WHERE条件中等号左右两边的字段类型必须一致，否则无法利用索引

7. WHERE子句中禁止只使用全模糊的LIKE条件进行查找，必须有其他等值或范围查询条件

8. 索引列禁止使用函数或表达式，否则无法利用索引。如where length(name)=’Admin’或where user_id+2=10023

9. or语句慎用

10. 使用EXPLAIN判断SQL语句是否合理使用了索引，尽量避免Extra列出现Using FileSort，Using Temporary

11. 建议不要使用“like%value”的形式，因为MySQL仅支持最左前缀索引，即 like '%value'不走索引而like 'value%'是可以走一部分索引的

12. 关联优化

    1. 不建议使用子查询，建议将子查询SQL拆开结合程序多次查询，或使用join来代替子查询
    2. 多表join不要超过3个表
    3. 多表join时，要把过滤性最大（不一定是数据量最小，而是指加了where条件后过滤性最大的那个）的表选为驱动表
    4. 如果join之后有排序，排序字段只有在属于驱动表的情况下，才能利用驱动表的索引完成排序
    5. join的关联字段在不同表中的类型和命名要一致

13. order by、group by优化

    1. 减少使用order by，和业务沟通能不排序就不排序或将排序放到程序端去做。order by、group by、distinct这些语句都比较耗费CPU
    2. order by、group by、distinct尽量利用索引直接检索出排序好的数据
    3. 包含了order by、group by、distinct这些查询的语句，必须加limit，限制行数控制在1000以内
    4. group by 时会按照一定规则进行排序，如果业务上对顺序没有要求，可以加order by null提高查询效率
    5. 多字段联合索引情况下，where中过滤条件的字段顺序无需和索引一致，但如果有排序、分组则必须一致，比如联合索引idx(a,b,c)
       1. 下面的sql都可以完整用到索引：
          - SELECT ... WHERE b = ? AND c = ? AND a = ?; --注意到，WHERE中字段顺序并没有和索引字段顺序一致
          - SELECT ... WHERE b = ? AND a = ? AND c = ?;
          - SELECT ... WHERE a = ? AND b IN (?, ?) AND c = ?;
          - SELECT ... WHERE a = ? AND b = ? ORDER BY c;
          - SELECT ... WHERE a = ? AND b IN (?, ?) ORDER BY c;
          - SELECT ... WHERE a = ? ORDER BY b, c;
          - SELECT ... ORDER BY a, b, c; -- 可利用联合索引完成排序
       2. 下面的sql只能用到部分索引，或者可利用ICP特性（ICP（index condition pushdown）是MySQL 5.6的新特性，其机制会让索引的其他部分也参与过滤，减少引擎层和server层之间的数据传输和回表请求，通常情况下可大幅提升查询效率）：
          - SELECT ... WHERE b = ? AND a = ?; -- 只能用到 (a, b) 部分
          - SELECT ... WHERE a IN (?, ?) AND b = ?; -- EXPLAIN显示只用到 (a, b) 部分索引，同时有ICP
          - SELECT ... WHERE (a BETWEEN ? AND ?) AND b = ?; -- EXPLAIN显示只用到 (a, b) 部分索引，同时有ICP
          - SELECT ... WHERE a = ? AND b IN (?, ?); -- EXPLAIN显示只用到 (a, b) 部分索引，同时有ICP
          - SELECT ... WHERE a = ? AND (b BETWEEN ? AND ?) AND c = ?; -- EXPLAIN显示用到 (a, b, c) 整个索引，同时有ICP
          - SELECT ... WHERE a = ? AND c = ?; -- EXPLAIN显示只用到 (a) 部分索引，同时有ICP
          - SELECT ... WHERE a = ? AND c >= ?; -- EXPLAIN显示只用到 (a) 部分索引，同时有ICP
       3. 下面的几个sql完全用不到索引：
          - SELECT ... WHERE b = ?;
          - SELECT ... WHERE b = ? AND c = ?;
          - SELECT ... WHERE b = ? AND c = ?;
          - SELECT ... ORDER BY b;
          - SELECT ... ORDER BY b, a;

14. 分页优化

    1. 分页查询，当limit起点较高时，可先用过滤条件进行过滤。如select a,b,c from t1 limit 1000,20；优化为: select a,b,c from t1 where id>1000 limit 20;

    2. 分页limit优化，采用覆盖索引和延迟关联的思想进行优化，改写前后sql查询时间从18秒缩短到1秒

       原sql：select id, awb_no, cf_info_pricing_id, lm_info_pricing_id, available_pricing_time,
       main_contractor_id,logistics_business_id, lm_id, batch_adj_no, negotiation_time
       from reconcile_result
       WHERE 1 = 1 AND lm_id = 4
       AND main_contractor_id = 3
       AND recon_status = 80
       AND available_pricing_time >= '2019-10-01 00:00:00'
       AND available_pricing_time <= '2019-10-07 23:59:59'
       AND is_deleted = 0
       LIMIT 10000, 10;

       改写sql：select [a.id](http://a.id/), a.awb_no, a.cf_info_pricing_id, a.lm_info_pricing_id, a.available_pricing_time,
       a.main_contractor_id,a.logistics_business_id, a.lm_id, a.batch_adj_no, a.negotiation_time
       from reconcile_result a,
       (select id
       from reconcile_result
       WHERE 1 = 1 AND lm_id = 4
       AND main_contractor_id = 3
       AND recon_status = 80
       AND available_pricing_time >= '2019-10-01 00:00:00'
       AND available_pricing_time <= '2019-10-07 23:59:59'
       AND is_deleted = 0
       LIMIT 10000, 10) b
       where [a.id](http://a.id/)=[b.id](http://b.id/);

       查询时间比较：
       ![](https://tva1.sinaimg.cn/large/007S8ZIlly1gdwq6it4abj315z0dgq7z.jpg)

       ![](https://tva1.sinaimg.cn/large/007S8ZIlly1gdwq6srbg4j315t0el0xq.jpg)

### 1.2 DML优化

1. INSERT语句需要指定具体字段名称，禁止写成INSERT INTO table VALUES(…)
2. 使用批量提交语句INSERT INTO table VALUES(),(),().. 插入行不要超过2000个，一次性提交过多的记录会导致线上I/O紧张，出现慢查询，引起主从同步延迟
3. 事务中批量更新数据必须控制数量，进行必要的sleep，做到少量多次
4. 事务涉及的表必须全部是INNODB表，否则一旦失败不会全部回滚
5. 禁止在业务的更新类SQL语句中使用join，比如update t1 join t2…
6. 事务中INSERT|UPDATE|DELETE|REPLACE语句操作的行数控制在2000以内，以及WHERE子句中IN列表的传参个数控制在500以内
7. UPDATE、DELETE语句需要根据WHERE条件添加索引
8. 批量操作数据时，需要控制事务处理间隔时间，进行必要的sleep，一般建议值5-10秒
9. 禁用procedure、function、trigger、views、event、外键约束，因为它们会消耗数据库资源，降低数据库集群可扩展性。推荐都在程序端实现
10. 禁止使用关联子查询，如update t1 set … where name in(select name from user where…);效率极其低下
11. 禁止联表更新语句，如update t1,t2 where [t1.id](http://t1.id/)=[t2.id](http://t2.id/)…
12. 禁用insert into …on duplicate key update… 原因如下：a. 在高并发环境下，容易发生死锁 b.会造成自增主键id不连续，假设原来有数据300条，用duplicate key插入100条数据，其中前99条都是和原来有重叠的，只有最后一条是新增的，那么最后一条id会从400开始
