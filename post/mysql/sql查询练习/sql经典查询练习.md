# mysql查询语句练习
## 数据表创建

学生表 Student
```
create table Student(
SId varchar(10) comment "学生编号",
Sname varchar(10) comment "学生姓名",
Sage datetime comment "出生年月",
Ssex varchar(10) comment "学生性别"
);
insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-12-20' , '男');
insert into Student values('04' , '李云' , '1990-12-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-01-01' , '女');
insert into Student values('07' , '郑竹' , '1989-01-01' , '女');
insert into Student values('09' , '张三' , '2017-12-20' , '女');
insert into Student values('10' , '李四' , '2017-12-25' , '女');
insert into Student values('11' , '李四' , '2012-06-06' , '女');
insert into Student values('12' , '赵六' , '2013-06-13' , '女');
insert into Student values('13' , '孙七' , '2014-06-01' , '女');
```
科目表 Course
```
create table Course(
CId varchar(10) comment "课程编号",
Cname nvarchar(10) comment "课程名称",
TId varchar(10) comment "教师编号"
);
insert into Course values('01' , '语文' , '02');
insert into Course values('02' , '数学' , '01');
insert into Course values('03' , '英语' , '03');
```
教师表Teacher
```
create table Teacher(
TId varchar(10) comment "教师编号",
Tname varchar(10) comment "教师姓名"
);
insert into Teacher values('01' , '张三');
insert into Teacher values('02' , '李四');
insert into Teacher values('03' , '王五');
```
成绩表 SC
```
create table SC(
SId varchar(10) comment "学生编号",
CId varchar(10) comment "课程编号",
score decimal(18,1) comment "分数"
);
insert into SC values('01' , '01' , 80);
insert into SC values('01' , '02' , 90);
insert into SC values('01' , '03' , 99);
insert into SC values('02' , '01' , 70);
insert into SC values('02' , '02' , 60);
insert into SC values('02' , '03' , 80);
insert into SC values('03' , '01' , 80);
insert into SC values('03' , '02' , 80);
insert into SC values('03' , '03' , 80);
insert into SC values('04' , '01' , 50);
insert into SC values('04' , '02' , 30);
insert into SC values('04' , '03' , 20);
insert into SC values('05' , '01' , 76);
insert into SC values('05' , '02' , 87);
insert into SC values('06' , '01' , 31);
insert into SC values('06' , '03' , 34);
insert into SC values('07' , '02' , 89);
insert into SC values('07' , '03' , 98);
```
表数据：
```
mysql> select * from Course;
+------+--------+------+
| CId  | Cname  | TId  |
+------+--------+------+
| 01   | 语文   | 02   |
| 02   | 数学   | 01   |
| 03   | 英语   | 03   |
+------+--------+------+
mysql> select * from Student;
+------+--------+---------------------+------+
| SId  | Sname  | Sage                | Ssex |
+------+--------+---------------------+------+
| 01   | 赵雷   | 1990-01-01 00:00:00 | 男   |
| 02   | 钱电   | 1990-12-21 00:00:00 | 男   |
| 03   | 孙风   | 1990-12-20 00:00:00 | 男   |
| 04   | 李云   | 1990-12-06 00:00:00 | 男   |
| 05   | 周梅   | 1991-12-01 00:00:00 | 女   |
| 06   | 吴兰   | 1992-01-01 00:00:00 | 女   |
| 07   | 郑竹   | 1989-01-01 00:00:00 | 女   |
| 09   | 张三   | 2017-12-20 00:00:00 | 女   |
| 10   | 李四   | 2017-12-25 00:00:00 | 女   |
| 11   | 李四   | 2012-06-06 00:00:00 | 女   |
| 12   | 赵六   | 2013-06-13 00:00:00 | 女   |
| 13   | 孙七   | 2014-06-01 00:00:00 | 女   |
+------+--------+---------------------+------+
mysql> select * from SC;
+------+------+-------+
| SId  | CId  | score |
+------+------+-------+
| 01   | 01   |  80.0 |
| 01   | 02   |  90.0 |
| 01   | 03   |  99.0 |
| 02   | 01   |  70.0 |
| 02   | 02   |  60.0 |
| 02   | 03   |  80.0 |
| 03   | 01   |  80.0 |
| 03   | 02   |  80.0 |
| 03   | 03   |  80.0 |
| 04   | 01   |  50.0 |
| 04   | 02   |  30.0 |
| 04   | 03   |  20.0 |
| 05   | 01   |  76.0 |
| 05   | 02   |  87.0 |
| 06   | 01   |  31.0 |
| 06   | 03   |  34.0 |
| 07   | 02   |  89.0 |
| 07   | 03   |  98.0 |
+------+------+-------+
mysql> select * from Teacher;
+------+--------+
| TId  | Tname  |
+------+--------+
| 01   | 张三   |
| 02   | 李四   |
| 03   | 王五   |
+------+--------+
```

## 练习

1. 查询" 01 "课程比" 02 "课程成绩高的学生的信息及课程分数
   * 1.1 查询同时存在" 01 "课程和" 02 "课程的情况
   * 1.2 查询存在" 01 "课程但可能不存在" 02 "课程的情况(不存在时显示为 null )
   * 1.3 查询不存在" 01 "课程但存在" 02 "课程的情况
2. 查询平均成绩大于等于 60 分的同学的学生编号和学生姓名和平均成绩
```mysql
select st.Sid, Sname, average 
from Student as st 
right join 
(select Sid, avg(score) as average from SC group by Sid having average >=60) as tmp 
on st.Sid = tmp.Sid;
```
3. 查询在 SC 表存在成绩的学生信息
```mysql
select * from Student where Sid in (select distinct(Sid) from SC);
```
4. 查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩(没成绩的显示为 null )
```mysql
# 方式一
select st.SId, st.Sname, tmp.sum_score as "总成绩", tmp.sum_course as "选课总数" 
from Student as st 
left join 
(select SId, sum(score) as sum_score,count(CId) as sum_course from SC group by SId) tmp 
on st.SId = tmp.SId;
# 方式二
select st.SId, st.Sname, count(CId) as "选课总数", sum(Score) as "总成绩" 
from Student st left join SC s1 
on st.SId = s1.SId 
group by st.SId;

```
5. 查询「李」姓老师的数量
```mysql
select count(1) from Teacher where Tname like "李%";
```
6. 查询学过「张三」老师授课的同学的信息
```mysql
select S.* from Teacher T join Course C on T.TId = C.TId and T.Tname = "张三" 
join SC on SC.CId = C.CId 
join Student S on S.SId = SC.SId;
```
7. 查询没有学全所有课程的同学的信息
```mysql
select st.* from Student st 
left join SC sc 
on st.SId = sc.SId 
group by st.SId 
having count(st.SId) < (select count(1) from Course);
```
8. 查询至少有一门课与学号为" 01 "的同学所学相同的同学的信息
```mysql
select st.* from Student st where SId in 
(
select distinct SId from SC where CId in 
(select CId from SC where SId = "01") 
and SId != "01");
```
9. 查询和" 01 "号的同学学习的课程 完全相同的其他同学的信息
10. 查询没学过"张三"老师讲授的任一门课程的学生姓名
```mysql
select * from Student st where st.SId not in (
select sc.SId from SC sc 
left join Course c on sc.CId = c.CId 
left join Teacher t on t.TId = c.TId
where t.Tname = "张三" group by sc.SId);
```
11. 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩
```python
select st.Sname, avg(score) as "平均成绩" from SC 
join Student st 
on st.SId = SC.SId 
group by SC.SId 
having count(if(SC.score < 60,1,null)) >= 2;
```
12. 检索" 01 "课程分数小于 60，按分数降序排列的学生信息
13. 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩
14. 查询各科成绩最高分、最低分和平均分：
    以如下形式显示：课程 ID，课程 name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率
    及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90
    要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列
15. 按各科成绩进行排序，并显示排名， Score 重复时保留名次空缺
    15.1 按各科成绩进行排序，并显示排名， Score 重复时合并名次
16. 查询学生的总成绩，并进行排名，总分重复时保留名次空缺
    16.1 查询学生的总成绩，并进行排名，总分重复时不保留名次空缺
17. 统计各科成绩各分数段人数：课程编号，课程名称，[100-85]，[85-70]，[70-60]，[60-0] 及所占百分比
18. 查询各科成绩前三名的记录
19. 查询每门课程被选修的学生数
20. 查询出只选修两门课程的学生学号和姓名
21. 查询男生、女生人数
```mysql
select count(Ssex) from Student group by Ssex;
```
22. 查询名字中含有「风」字的学生信息
```mysql
select * from Student where Sname like "%风%"
```
23. 查询同名同性学生名单，并统计同名人数
24. 查询 1990 年出生的学生名单
```mysql
select * from Student where year(Sage) = "1990";
```
25. 查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列
26. 查询平均成绩大于等于 85 的所有学生的学号、姓名和平均成绩
27. 查询课程名称为「数学」，且分数低于 60 的学生姓名和分数
28. 查询所有学生的课程及分数情况（存在学生没成绩，没选课的情况）
29. 查询任何一门课程成绩在 70 分以上的姓名、课程名称和分数
30. 查询不及格的课程
31. 查询课程编号为 01 且课程成绩在 80 分以上的学生的学号和姓名
32. 求每门课程的学生人数
33. 成绩不重复，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩
34. 成绩有重复的情况下，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩
35. 查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩
36. 查询每门功成绩最好的前两名
37. 统计每门课程的学生选修人数（超过 5 人的课程才统计）。
38. 检索至少选修两门课程的学生学号
39. 查询选修了全部课程的学生信息
40. 查询各学生的年龄，只按年份来算
```mysql
select Sname, (year(now()) - year(Sage)) as year from Student;
```
41. 按照出生日期来算，当前月日 < 出生年月的月日则，年龄减一
42. 查询本周过生日的学生
43. 查询下周过生日的学生
44. 查询本月过生日的学生
45. 查询下月过生日的学生
```mysql
select * from Student where month(Sage)-month(now()) = 1;
```
