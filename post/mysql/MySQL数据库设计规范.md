# MySQL数据库设计规范

# 一、建库规范

1. 库命名规范：
   1. 库名称必须控制在25个字符以内
   2. 库名与应用名称尽量一致，库的命名规则必须契合所属业务的特点
   3. 库名用小写（尽量不要使用除下划线、小写英文字母之外的其他字符，如果要用下划线，应该尽量保持一致的风格）
2. 字符集规范：创建数据库时必须显示指定字符集，建议使用utf8mb4字符集：create database xxxxxx default CHARACTER SET utf8mb4;
   MySQL 5.5.3之后增加utf8mb4编码，utf8mb4是utf8的一个扩展。许多新类型的字符，例如emoji这种类型的符号，utf8不支持存储，但utf8mb4支持。所以，设计数据库时如果想要允许用户使用特殊符号，最好使用utf8mb4编码来存储，使得数据库有更好的兼容性
   可参考官方说明：https://dev.mysql.com/doc/refman/5.5/en/charset-unicode-utf8mb4.html

# 二、建表规范

1. 表命名规范：
   1. 表命名做到专业、简洁、见名知意，多使用专业词汇命名，不使用拼音，以使用方便记忆、描述性强的可读性名称为第一准则，应尽量避免使用缩写或代码来命名
   2. 用小写（尽量不要使用除下划线、小写英文字母、数字之外的其他字符，如果要用下划线，应该尽量保持一致的风格）
   3. 命名不要过长（应尽量少于25个字符）
   4. 禁止使用数字开头，禁止使用关键字或保留字，可参考官方说明：https://dev.mysql.com/doc/refman/5.7/en/keywords.html
   5. 同一个应用(或领域)下的表需要有相同的前缀名称，如：acticity_share、acticity_order、activity_item；同一个数据库下有不同的应用模块，则可以考虑对表名用不同的前缀标识
   6. 备份表命名时加上时间标识
   7. 相关模块的表名与表名之间尽量体现join的关系，如user表和user_login表
   8. 表命名加上“业务名称_表的作用”，如：alipay_task / force_project / trade_config
2. 表注释规范：
   1. 每个表建立时必须加上表描述（如COMMENT='履约单商品表'），方便参考理解、维护管理
3. 存储引擎规范：
   1. 若没有特殊要求，存储引擎均采用默认的InnoDB（ENGINE=InnoDB）
4. 数据表存储格式规范：
   1. row_format没有特殊需求时默认即可，不需要指定。row_format=Dynamic/Compressed只有在innodb_file_format=barracuda的情况下才支持，如果强制设置了，后续再对表进行DDL操作时会产生警告：#1 Execute(Warning, Code 1478):InnoDB: ROW_FORMAT=DYNAMIC requires innodb_file_format > Antelope. #2 Execute(Warning, Code 1478):InnoDB: assuming ROW_FORMAT=COMPACT
5. 表字符集规范：
   1. 若没有特殊要求，统一设置为utf8mb4，即CHARSET = utf8mb4。关联查询时，若字符集不一致，可能会导致索引失效
6. 主键规范：
   1. 每个表建立时必须设置主键id（PRIMARY KEY (`id`)），自增长（AUTO_INCREMENT）、步长为1，类型为整型（根据需要选择int或bigint。一般情况下业务表使用bigint类型，防止数据量增长后自增主键值不够用；对于数据量不会增长很多的配置表，可使用int类型）

# 三、字段规范

1. 字段命名规范：

   1. 字段命名要做到专业、简洁，做到见名知意，多使用专业词汇命名，不使用拼音，以使用方便记忆、描述性强的可读性名称为第一准则，应尽量避免使用缩写或代码来命名
   2. 用小写（尽量不要使用除下划线、小写英文字母之外的其他字符，如果要用下划线，应该尽量保持一致的风格）
   3. 命名不要过长（应尽量少于25个字符）
   4. 禁止使用数字开头，禁止使用mysql关键字
   5. 注意字段类型的一致性、命名的一致性，同一个字段在不同的表中也应是相同的类型或长度，方便大家理解（如用户ID，都用user_id，就不需要再造一个member_id；如创建时间统一命名为create_time，更新时间统一命名为update_time）

2. 字段注释规范：

   1. 字段注释必须加上（如 product varchar(100) NOT NULL DEFAULT '' COMMENT '商品名称'），字段取值范围含义、枚举常量等注释必须加上，方便参考理解、维护管理
   2. 字段注释规范：修改字段含义或对字段表示的状态追加时，需要及时更新字段注释

3. 字段默认值规范：

   1. 尽量将字段设置成NOT NULL DEFAULT xxx。可为NULL的列使得索引、索引统计和值都比较复杂，InnoDB使用单独的位（bit）存储NULL值，当可为NULL的列被索引时，每个索引记录需要一个额外的字节，所以会占用更多的存储空间，在MyISAM引擎中，NULL值会使索引失效。另外，使用null值之后，sql的复杂度变大，会使优化器更难以优化SQL，如下sql所示，如果box_type有默认值，则sql不需要用or来关联：

      ```
      SELECT
          idle_sku.sku_id,
          sum(1) as stocked_sum
      FROM
          idle_sku
          LEFT JOIN box_sku on box_sku.id = idle_sku.box_id
      where
          idle_sku.status not in ('IDLE_TRASH')
          and (
              box_sku.box_type = 1
              or box_sku.box_type is null
          )
      group by
          idle_sku.sku_id;
      ```

   2. 在对字段进行count()统计时，值为NULL的不会被count统计进去，即当字段值为NULL时，count(*)和count(字段)值是不一样的

   3. 字段默认值规范：当前字段为NULL时，更改字段时不必要再NOT NULL DEFAULT xxx，因为数据量过多，改动成本过大，可能会对磁盘IO造成过多压力，阻塞其他进程

4. 字段顺序规范：

   1. 表的字段顺序很重要，从前到后，按照字段的重要性和使用频率排列，create_time、update_time、remark等字段在最后。按字段的分类归集排列，如金额相关的字段在一块，时间相关的字段在一块等。以方便查看、排查问题（注：增加字段时不采用after和first关键字，原因是数仓增量采集数据时无法感知 after的位置，如果在表中间位置加字段，会使数仓同步数据失败）

5. 字段拆分规范：

   1. 考虑使用垂直分区。比如，我们可以把大字段或使用不频繁的字段分离到另外的表中，这样做可以减少表的大小，让表执行得更快。我们还可以把一个频繁更新的字段放到另外的表中，因为频繁更新的字段会导致MySQL Query Cache里相关的结果集频繁失效，可能会影响性能。需要留意的一点是，垂直分区的目的是为了优化性能，但如果将字段分离了到分离表后，又经常需要建立连接，那可能就会得不偿失了，所以，我们要确保分离的表不会经常进行连接，这时，用程序进行连接是一个可以考虑的办法

# 四、字段数据类型规范

1. 设计字段有一个基本的原则，保小不保大，也就是一般情况应该尽量使用可以正确存储数据的最小数据类型，更小的数据类型执行速度通常更快，因为它们占用更小的磁盘、内存和CPU缓存，处理时需要的CPU周期也更小

2. 整型：

   1. 在MySQL中支持的5个主要整数类型是tinyint、smallint、mediumint、int、bigint，下表列出了各种数值类型以及它们的允许范围和占用的内存空间：

      | 类型      | 字节 | 最小值~最大值（有符号）                              | 最小值~最大值（无符号）      |
      | :-------- | :--- | :--------------------------------------------------- | :--------------------------- |
      | tinyint   | 1    | -128~127                                             | 0~255                        |
      | smallint  | 2    | -32 768~32 767                                       | 0~65 535                     |
      | mediumint | 3    | -8 388 608~8 388 607                                 | 0~16 777 215                 |
      | int       | 4    | -2 147 483 648~2 147 483 647                         | 0~4 294 967 295              |
      | bigint    | 8    | -9 223 372 036 854 775 808~9 223 372 036 854 775 807 | 0~18 446 744 073 709 551 615 |

   2. 建议使用unsigned（无符号类型）存储非负值，这样数值范围可以扩大一倍，例如tinyint的取值范围为-127~128，unsigned tinyint的取值范围为0~255

   3. 建议使用无符号整型存储IPV4（IP地址），可以使用INET_ATON()、INET_NTOA()函数进行转换，基本上没必要使用char类型来存储。例如，使用INET_ATON('172.16.23.16')将IP转化成整数值2886735632，使用INET_NTOA(2886735632)可得到IP值172.16.23.16

   4. 整型定义中不添加显示长度的值，比如使用INT，而不是INT(4)

   5. 使用更短小的列，比如短整型，整型列的执行速度往往更快。年龄、性别、状态、删除标记等枚举类型：可以用 tinyint 来存放，它只占用 1 个字节，unsigned tinyint 可以表示 0~255 的范围，基本够用

   6. 手机号：通常我们在存储手机号时，喜欢用 varchar 类型。假设是 11 位的手机号，用 utf8 编码。如果使用字符串存储，每位需要 3 个字节，一共需要 11*3=33 个字节；如果使用 bigint，只需要 8 个字节

   7. 能用整型的就用整型而不用字符型，如下反面示例中状态值完全可以用整型替代，从数据库读取之后再做释义即可：

      1. `status` varchar(32) NOT NULL DEFAULT 'IDLE_UNALLOCATED' COMMENT '状态: IDLE_UNALLOCATED,IDLE_TO_SHELVE, IDLE_SHELVED, IDLE_OCCUPIED, IDLE_TO_OFF_SHELF,IDLE_TRASH, IDLE_FIN'

3. 浮点型：

   1. 存储精确浮点数时必须使用DECIMAL替代FLOAT和DOUBLE
   2. 价格、金额等数值采用decimal类型，精度高，float 和 double 可能会存在精度损失的问题

4. 字符类型：

   1. char 和 varchar 是我们常用的字符类型。char(N) 用来记录固定长度的字符，长度最大为255，比指定长度大的值将被截短，而比指定长度小的值将会用空格进行填补。varchar(N) 用来保存可变长度的字符，只存储字符串实际需要的长度，varchar使用额外的1～2字节来存储值的长度，如果列的最大长度小于或等于255则使用1字节，否则就是用2字节。char 和 varchar 占用的字节数，根据数据库的编码格式不同而不同。Latin1 占用 1 个字节，gbk 占用 2 个字节，utf8 占用 3 个字节
   2. 用法方面，如果存储的内容是可变长度的，如家庭住址、用户描述等就可以用 varchar；如果内容是固定长度的，例如：UUID（36 位）、MD5 加密串（32 位）等就可以使用 Char 存放；如果存储的字符串长度几乎相等，使用 char ；如果存储的字符串长度差别大，使用varchar
   3. 字符串存储能用varchar就不要用text，varchar长度不要超过5000，存储搜索性能均高于text，text查询时会产生临时磁盘文件，性能差。如果字符串长度超过5000需要使用text类型存储，则建议独立出来一张表，用主键来对应，避免影响其它字段索引效率。MySQL把text值当作一个独立的对象处理，存储引擎在存储时通常会做特殊处理，当text值过大时，InnoDB会使用专门的外部存储区域来进行存储，此时每个值在行内需要1-4个字节存储一个指针，然后在外部存储区域存储实际的值
   4. 慷慨是不明智的，最好的策略是只分配真正需要的空间。如使用varchar(5)和varchar(200)存储'hello'的空间开销是一样的，但是varchar(200)会消耗更多的内存，因为MySQL通常会分配固定大小的内存块来保存内部值，尤其是使用内存临时表进行排序或操作时会特别糟糕，在利用磁盘临时表进行排序时也同样糟糕。
   5. 字段数据类型的选择要方便以后扩展。一般扩展性有限的常量类型，建议用整型，性能高、占空间少，比如状态字段；一般不确定扩展性的类型，建议用字符型，可以添加业务规则，一个字段可以同时表示多种含义
   6. 在VARCHAR(N)中，N表示的是字符数而不是字节数，比如VARCHAR(255)，最大可存储255个汉字。需要根据实际的宽度来选择N。此外，N应尽可能地小，因为在MySQL的一个表中，所有的VARCHAR字段的最大长度是65535个字节，进行排序和创建临时表一类的内存操作时，会使用N的长度申请内存（对于这一点，MySQL 5.7后有了改进）
   7. 字符型详解参考链接：[字符类型详解](http://wiki.yuceyi.com/pages/viewpage.action?pageId=43959039)

5. JSON类型：

   1. 存储JSON字符串时选用JSON类型，而不要选varchar、text类型存储
   2. JSON类型的意义：
      1. 保证了JSON数据类型的强校验，JSON数据列会自动校验存入此列的内容是否符合JSON格式，非正常格式则报错，而varchar类型和text等类型不存在这种机制
      2. 更优化的存储格式，存储在JSON列中的JSON数据会被转成内部特定的存储格式，允许快速读取
      3. MySQL同时提供了一组操作JSON类型数据的内置函数
      4. 可以基于JSON格式的特征支持修改特定的键值（即不需要把整条内容拿出来放到程序中遍历然后寻找替换再塞回去，MySQL内置的函数允许你通过一条SQL语句就能搞定）

6. 时间类型：

   1. 存储年时使用year类型，存储日期时使用date类型，存储时间时使用time类型，date保存精度到天，格式为：YYYY-MM-DD，如2019-11-07
   2. 时间类型的精度统一到毫秒级别，毫秒的格式类似为：2019-11-07 10:58:27.257，定义方法：datetime(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3)，datetime(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3)
   3. 建议使用datetime类型存储日期时间，因为它的可用范围比timestamp更大，mysql 5.6.4以后物理存储上仅比timestamp多1个字节，整体性能上的损失并不大
   4. datetime和timestamp有下面几个小区别，需要注意：
      1. 区别一：存储数据方式不同
         - timestamp 是转化成 utc 时间进行存储，查询时转化为客户端时间返回
      2. 区别二：可存储的时间范围不同
         - timestamp 为'1970-01-01 00:00:01.000000' 到'2038-01-19 03:14:07.999999'
         - datetime为'1000-01-01 00:00:00.000000'到'9999-12-31 23:59:59.999999'
      3. 区别三：占用存储空间不同
         - MySQL 5.6.4之前：datetime 占用 8 个字节，timestamp 占用 4 个字节
         - MySQL 5.6.4开始：datetime 占用 5 个字节+小数秒存储，timestamp 占用 4 个字节+小数秒存储
      4. 区别四：受时区影响不同
         - 当更改时区参数time_zone时，timestamp会随时区改变而改变，datetime不会改变
      5. 区别五：索引效率不同
         - timestamp更轻量，索引相对更快

7. text、blob类型：

   1. 尽可能不要使用text、blob类型
   2. 不要在数据库中使用VARBINARY或BLOB存储图片及文件等。MySQL并不适合大量存储这种类型的文件

五、索引规范

1. 索引命名规范：

   1. 主键索引名为 pk_字段名；唯一索引名为 uniq_字段名；普通索引名则为 idx_字段名（pk_ 即 primary key；uniq_ 即 unique key；idx_ 即 index）
   2. 用小写，命名不要过长

2. 主键规范：每个表必须设置主键id（PRIMARY KEY (`id`)），字段类型必须为整型、自增长（AUTO_INCREMENT）、步长为1（根据需要选择int或bigint。一般情况下业务表使用bigint类型，防止数据量增长后自增主键值不够用；对于数据量不会增长过多的配置表，可使用int类型）

3. 外键规范：禁止使用外键，约束逻辑在应用层面解决，外键影响性能，并且在并发操作时容易引起死锁

4. 索引数量规范：单张表的索引数量建议控制在5个以内

5. 普通索引规范：

   1. 选择性低的字段不适合单独建立索引（如状态值status、性别等），即使设计了索引也未必会生效，查询时mysql优化器可能仍然选择全表扫描，这样的索引反而浪费存储空间，影响增删改的效率
      这里需要引入一个概念，索引的选择性。索引的选择性是指，不重复的索引值和数据表的记录总数的比值。索引的选择性越高则查询效率越高，因为选择性高的索引可以让 MySQL 在查找时过滤掉更多的行
   2. UPDATE、DELETE语句需要根据WHERE条件添加索引
   3. 使用EXPLAIN判断SQL语句是否合理使用了索引，尽量避免Extra列出现Using FileSort，Using Temporary
   4. 合理地利用覆盖索引。由于覆盖索引一般常驻于内存中，因此可以大大提高查询速度
   5. 对长度过长的VARCHAR字段（比如网页地址）建立索引时，需要增加散列字段，对VARCHAR使用散列算法时，散列后的字段最好是整型，然后对该字段建立索引
   6. 存储域名地址时，可以考虑采用反向存储的方法，比如把[news.sohu.com](http://news.sohu.com/)存储为com.sohu.news，方便在其上构建索引和进行统计

6. 联合索引规范：

   1. 建议联合索引中的字段数量不超过5个
   2. 经常用的列优先（最左前缀原则）、离散度高的列优先（离散度高原则）、宽度小的列优先（最少空间原则），另外还是要结合索引的原理结构和具体使用的业务场景
   3. 索引字段的顺序需要考虑字段唯一值的个数，一般选择性高的字段放在前面
   4. ORDER BY、GROUP BY、DISTINCT的字段需要放在联合索引的后面，也就是说，联合索引的前面部分用于等值查询，后面的部分用于排序
   5. 合理创建联合索引，联合合索引(a,b,c)可以用于“where a=?”、“where a=?and b=?”、“where a=?and b=?and c=?”等形式，但对于“where a=?”的查询，可能会比仅仅在a列上创建单列索引查询要慢，因此需要在空间和效率上达成平衡
   6. 把范围条件放到联合索引的最后，WHERE条件中的范围条件（BETWEEN、<、<=、>、>=）会导致后面的条件使用不了索引

7. 前缀索引规范：

   1. 对于超过20个长度的字符串列，可以考虑创建前缀索引
   2. 或者一个字段的前几个字符的选择性跟整个字段的选择性差不多，这时候考虑前缀索引，节省空间且提高效率，例如邮箱地址xxxx@[clubfactory.com](http://clubfactory.com/)，取前几位姓名做前缀索引即可
   3. 缺点是排序无法使用前缀索引

8. 时间索引规范：每个业务相关的表在建表时必须加上创建时间索引和更新时间索引，以方便数据清理转移、数仓同步数据

   最后的最后，sql进行explain后再提测是一种美德

# 六、分表规范

1. MySQL单表数据量在300w-500w左右时性能最佳；每张表的数据量控制在5000万以下，数据量超过5000万考虑水平拆分，单表字段值过多考虑垂直拆分
2. 对于可预见的短期内会形成大表的，在建表时即可考虑分表，以免后期被动
3. 推荐使用CRC32求余（或者类似的算术算法）进行分表，表名后缀使用数字，数字必须从0开始并等宽，比如散100张表，后缀则是从00-99
4. 使用时间分表，表名后缀必须使用固定的格式，比如按日分表为user_20110101
5. 深入理解业务，根据业务特性、查询需求来分表，如按年、季度、月分，如按user_id取模、seller_id末尾数字来分等

# 七、其他规范

1. 禁止使用函数、触发器、存储过程等数据库对象，这些会将业务逻辑和数据库耦合在一起，MySQL的存储过程、触发器和函数中可能会存在一些Bug，维护起来不方便、出现问题难以排查

# 八、建表示例

```
`CREATE` `TABLE` ``air_route` (`` ```id` ``bigint``(20) unsigned ``NOT` `NULL` `AUTO_INCREMENT COMMENT ``'主键'``,`` ```air_route_code` ``varchar``(32) ``NOT` `NULL` `DEFAULT` `''` `COMMENT ``'航线简码'``,`` ```air_route_middle` ``varchar``(1024) ``NOT` `NULL` `DEFAULT` `''` `COMMENT ``'经停信息'``,`` ```origin_country_name` ``varchar``(128) ``NOT` `NULL` `DEFAULT` `''` `COMMENT ``'出发国'``,`` ```origin_country_code` ``varchar``(32) ``NOT` `NULL` `DEFAULT` `''` `COMMENT ``'出发国简码'``,`` ```origin_air_line_code` ``varchar``(64) ``NOT` `NULL` `DEFAULT` `''` `COMMENT ``'出发港航司简码'``,`` ```origin_port_code` ``varchar``(32) ``NOT` `NULL` `DEFAULT` `''` `COMMENT ``'出发港简码'``,`` ```dest_country_name` ``varchar``(128) ``NOT` `NULL` `DEFAULT` `''` `COMMENT ``'目的国'``,`` ```dest_country_code` ``varchar``(32) ``NOT` `NULL` `DEFAULT` `''` `COMMENT ``'目的国简码'``,`` ```dest_port_code` ``varchar``(32) ``NOT` `NULL` `DEFAULT` `''` `COMMENT ``'目的港简码'``,`` ```air_space_type` tinyint(4) unsigned ``DEFAULT` `NULL` `COMMENT ``'舱位类型'``,`` ```air_route_status` tinyint(4) ``NOT` `NULL` `DEFAULT` `'2'` `COMMENT ``'状态，1. 使用中 2. 暂停使用'``,`` ```create_time` datetime(3) ``NOT` `NULL` `DEFAULT` `CURRENT_TIMESTAMP``(3) COMMENT ``'创建时间'``,`` ```update_time` datetime(3) ``NOT` `NULL` `DEFAULT` `CURRENT_TIMESTAMP``(3) ``ON` `UPDATE` `CURRENT_TIMESTAMP``(3) COMMENT ``'更新时间'``,`` ```is_deleted` tinyint(4) ``NOT` `NULL` `DEFAULT` `'0'` `COMMENT ``'逻辑删除'``,`` ``PRIMARY` `KEY` `(`id`),`` ``UNIQUE` `KEY` ``uniq_air_route_code` (`air_route_code`),`` ``KEY` ``idx_air_route_status` (`air_route_status`)``) ENGINE = InnoDB ``DEFAULT` `CHARSET = utf8mb4 COMMENT = ``'航线表'``;`
```
