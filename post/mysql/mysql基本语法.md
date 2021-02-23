# MySQL基本操作
## 查询语句执行顺序
一般的查询语句

```
select 输出 from 获取数据 where 过滤 group by 分组 having 过滤 order by 排序 limit 限定个数
```

执行顺序

`from`->`where`->`group by`->`select`->`having`->`order by`->`limit`
* having是对分组后的结果进行过滤
* where后语句不能带聚合函数函数

### group by

根据group by后面的字段进行分组，相同的分为一组，输出的结果是每个分组的第一行数据

```
mysql> select * from test;
+----+------+----------+
| id | name | class_id |
+----+------+----------+
|  1 | A    |        1 |
|  2 | B    |        1 |
|  3 | C    |        2 |
|  4 | D    |        2 |
|  5 | E    |        1 |
|  6 | F    |        1 |
|  7 | NULL |        2 |
+----+------+----------+
mysql> select count(1), class_id from test group by class_id;
+----------+----------+
| count(1) | class_id |
+----------+----------+
|        4 |        1 |
|        3 |        2 |
+----------+----------+
```
count(1)可以理解为新增了一列tmp，它的值都是1，然后统计每个分组中1的个数

**count(1)和count(name)的区别？**

如果name中有null，那么count(name)不记录null，这是唯一的区别

```
mysql> select count(name), count(1) from test group by class_id;
+-------------+----------+
| count(name) | count(1) |
+-------------+----------+
|           4 |        4 |
|           2 |        3 |
+-------------+----------+
```
group by后面跟多个字段，结果是什么呢？
```
mysql> select * from test group by class_id,name;
+----+------+----------+
| id | name | class_id |
+----+------+----------+
|  1 | A    |        1 |
|  2 | B    |        1 |
|  5 | E    |        1 |
|  6 | F    |        1 |
|  7 | NULL |        2 |
|  3 | C    |        2 |
|  4 | D    |        2 |
+----+------+----------+
```
可以看出并不是先对class_id进行分组，然后再对name分组；而是将class_id和name作为一体进行分组
```
mysql> select * from (select * from test group by class_id) as T group by name;
+----+------+----------+
| id | name | class_id |
+----+------+----------+
|  1 | A    |        1 |
|  3 | C    |        2 |
+----+------+----------+
```
先对class_id进行分组，再对结果按照name进行分组

group一般跟聚合函数联合使用，聚合函数主要包括sum，max，min，avg，group-concat
## 表的增删改
### 创建表
```
CREATE TABLE `nano_order` (
  `id` bigint UNSIGNED NOT NULL AUTO_INCREMENT COMMENT 'id',
  `nano_number` varchar(32) NOT NULL DEFAULT '' COMMENT 'nano单号',
  `tracking_no` varchar(32) NOT NULL DEFAULT '' COMMENT '物流单号',
  `delivery_way` varchar(20) NOT NULL DEFAULT '' COMMENT '物流方式',
  `order_name` varchar(24) NOT NULL DEFAULT '' COMMENT '订单号',
  `create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT 'ERP端的创建时间 UTC',
  `update_time` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT 'ERP端的更新时间 UTC',
  `sp_name` varchar(32) NOT NULL DEFAULT '' COMMENT '出库单',
  PRIMARY KEY (`id`) USING BTREE,
  KEY `idx_order_name` (`order_name`) USING BTREE,
  KEY `idx_nano_order` (`nano_number`) USING BTREE,
  KEY `idx_sp_name` (`sp_name`) USING BTREE,
  KEY `idx_update_time` (`update_time`) USING BTREE,
  UNIQUE KEY `uidx_tracking_no_delivery_way` (`tracking_no`,`delivery_way`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT 'nano_order表';
```

### 修改表
```
update sale_order set shipping_state="Punjab",shipping_city="Jallandhur" where order_name="SO275146782";
```

### 插入
```
INSERT INTO cargo_proxy(cargo_proxy, email_collection, warehouse, report_type_collection)
VALUES ('盛凡', "sandy@sfanair.com", 'hangzhou2',"summary_from_manifest"), 
('盛凡', "sandy@sfanair.com", 'dongguan2', "summary_from_manifest")
```

### 清空表
```
truncate table cargo_proxy
```

### 更改默认值
```
alter table carrier_company alter column status set default 1;
```

### 增加索引
```
alter table seller_address_registration DROP INDEX address_id,add unique index idx_addr_id_delivery_way(address_id,delivery_way)
###
alter table order_nano_rel add index idx_carrier_order(carrier_order),add index idx_order_name(order_name);
```



## case when

查询

```
select 字段1, 字段2,       
    case 字段3     
    when 值1 then 新值       
    when 值2 then 新值
    else 新值
    end as 重新命名字段3的名字       
from table      
where ……      
order by …… 
```

or

```
select 字段1, 字段2,       
    case 
    when 条件1 then 值1
    when 条件2 then 值2
    else 值3
    end as 重新命名字段3的名字       
from table      
where ……      
order by …… 
```

更新

```
update table  
set 字段1=case     
    when 条件1 then 值1       
    when 条件2 then 值2      
    else 值3      
    end     
where ……
```

用例

```
mysql> select * from Course;
+------+--------+------+
| CId  | Cname  | TId  |
+------+--------+------+
| 01   | 语文   | 02   |
| 02   | 数学   | 01   |
| 03   | 英语   | 03   |
+------+--------+------+
```

把Tid替换

```mysql
select Cid,Cname,
case 
when Tid="02" 
then "05" 
else "06" 
end as new_Tid 
from Course;
```
## if和ifnull
`if(expr, value1, value2)`：如果expr为True，返回value1，否则返回value2

`ifnull(value1, value2)`：如果value1为null，返回value2，否则放回value1

## limit和offset
select语句查询出来的结果它们可能是很多条数据，为了返回第一行或者指定前面几行，这时候就需要使用limit关键字来获取查询结果了。

EG:
```mysql
SELECT * FROM products LIMIT 0,8;
SELECT * FROM products LIMIT 8 OFFSET 0; (在mysql 5以后支持这种写法)
```
解释：
* “0”: 代表数据获取的起始位置(0代表第一条记录,以此递增)
* “8”: 期望获取的记录条数
表示查询的结果从第一条记录开始获取获，获取的记录条数不超过8条

这里为什么是不超过8条呢? 因为可能数据库的总记录是小于8条的此时就是以实际的记录数为最终的记录数。

当获取的记录是从第一条开始则可以省略书写起始位置 “0”
```mysql
SELECT * FROM products LIMIT 8;
```

