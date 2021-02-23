# 问题
表scores
```
+------+--------+-------+
| name | stage  | score |
+------+--------+-------+
| A    | 基础   |     1 |
| B    | 基础   |     2 |
| C    | 基础   |     2 |
| A    | 中级   |     2 |
| B    | 中级   |     3 |
| C    | 中级   |     1 |
| A    | 高级   |     2 |
| B    | 高级   |     2 |
| C    | 高级   |     4 |
+------+--------+-------+
```
需要转成
```
+------+--------+--------+---------+
| name | 基础   | 中级   | 高级    |
+------+--------+--------+---------+
| A    |      1 |      2 |       2 |
| B    |      2 |      3 |       2 |
| C    |      2 |      1 |       4 |
+------+--------+--------+---------+
```
# 解答

分析：
```
mysql> select name,
    -> case when stage="基础" then score else null end as "基础",
    -> case when stage="中级" then score else null end as "中级",
    -> case when stage="高级 " then score else null end as "高级 "
    -> from scores;
+------+--------+--------+---------+
| name | 基础   | 中级   | 高级    |
+------+--------+--------+---------+
| A    |      1 |   NULL |    NULL |
| B    |      2 |   NULL |    NULL |
| C    |      2 |   NULL |    NULL |
| A    |   NULL |      2 |    NULL |
| B    |   NULL |      3 |    NULL |
| C    |   NULL |      1 |    NULL |
| A    |   NULL |   NULL |       2 |
| B    |   NULL |   NULL |       2 |
| C    |   NULL |   NULL |       4 |
+------+--------+--------+---------+
```
结果：
```mysql
select name,
max(case when stage="基础" then score else null end) as "基础",
max(case when stage="中级" then score else null end) as "中级",
max(case when stage="高级 " then score else null end) as "高级 "
from scores group by name;
```
参考：https://www.bilibili.com/video/BV15t41177pv?p=14
