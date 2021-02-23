# 子查询
[子查询](https://www.cnblogs.com/zhuiluoyu/p/5822481.html)
# 非相关子查询和相关子查询执行流程
## 非相关子查询
先看一个非相关子查询到sql语句。

需求：查询学生表student和学生成绩表grade中成绩为70分的学生的基本信息。
```mysql
select t.sno,t.sname,t.sage,t.sgentle,t.sbirth,t.sdept 
from student t where t.sno 
in (select f.sno from garde f where f.score=70)
```
这个sql语句的执行时是简单的，

1、在grade表中找出成绩为70的学生学号sno,再将该学号返回到父查询作为where子句的条件。

2、在student表中找到该学号学生的其他基本信息。

**注：非相关子查询流程是先子查询，然后再父查询**

## 相关子查询
所谓相关子查询，是指求解相关子查询不能像求解普通子查询那样，一次将子查询求解出来，然后求解父查询。相关子查询的内层查询由于与外层查询有关，因此必须反复求值。

下面看相关子查询的sql语句。

需求：在学生表student和学生成绩表grade找出参加了“计算机基础”课程并且分数在80分以上的所有学生信息。
```mysql
select t.sno,t.sname,t.sage,t.sgentle,t.sbirth,sdept 
from student t 
where 80<=(select f.score from grade f where f.sno=t.sno and f.cname='计算机基础')
```
该子查询的执行流程：

1、 先从父查询的student表中取出第一条记录的sno值，进入子查询中，比较其where子句的条件“where f.sno=t.sno and f.cname=’计算机基础’”，符合则返回score成绩。

2、 返回父查询，判断父查询的where子句条件80<=返回的score，如果条件为true，则返回第1条记录。

3、 从父查询的student表中取出第2条数据，重复上述操作，直到所有父查询中的表中记录取完为止。

**注：相关子查询流程是先父查询，然后再子查询，相关子查询的特点就是子查询里引用了父查询的表**

总结：

对比这两个查询的sql执行过程可以看出，相关子查询和非相关子查询的不同点在于，相关子查询依赖于父查询，父查询和子查询是有联系的，尤其在子查询的where语句中更是如此。明白了他们的执行过程，再去看相关子查询的代码，一下子就明白了。

案例1：[简单易懂教你学会SQL关联子查询](https://zhuanlan.zhihu.com/p/41844742)

案例2：[184.部门工资最高的员工](https://github.com/lianglu1024/helloworld/blob/master/mysql/sql%E6%9F%A5%E8%AF%A2%E7%BB%83%E4%B9%A0/184%E9%83%A8%E9%97%A8%E5%B7%A5%E8%B5%84%E6%9C%80%E9%AB%98%E7%9A%84%E5%91%98%E5%B7%A5.md)

# exists和in的区别
exists的用法：
```
select * from tables_name where [not] exists(select..);
```
exists后面的子查询被称做相关子查询，它是不返回列表的值的。只是返回一个ture或false的结果(这也是为什么子查询里是 "select 1"的原因，当然也可以select任何东西)。其运行方式是先运行主查询一次，再去子查询里查询与其对应的结果，如果是ture则输出，反之则不输出。再根据主查询中的每一行去子查询里去查询。

**Exists执行顺序如下：** 
* 首先执行一次外部查询 
* 对于外部查询中的每一行分别执行一次子查询，而且每次执行子查询时都会引用外部查询中当前行的值。 
* 使用子查询的结果来确定外部查询的结果集。（如果外部查询返回100行，SQL就将执行101次查询，一次执行外部查询，然后为外部查询返回的每一行执行一次子查询。但实际上，SQL的查询优化器有可能会找到一种更好的方法来执行相关子查询，而不需要实际执行101次查询。）

**IN的执行过程如下：**
* 首先运行子查询，获取子结果集
* 主查询再去结果集里去找符合要求的字段列表，符合要求的输出，反之则不输出。
