## JDBC
JDBC：Java DataBase Connectivity

JDBC的本质是什么？JDBC是SUN公司制定的一套接口（interface）
接口都有调用者和实现者。面向接口调用、面向接口写实现类，这都是面向接口编程。

为什么要面向接口编程呢？
解耦合：降低程序的耦合度，提供程序的扩展力。多态机制就是非常典型的面相抽象编程
建议：
```java
Animal a = new Cat();
Animal a = new Dog();

public void feed(Animal a){// 面向父类型编程}
```
不建议
```java
Cat a = new Cat();
Dog a = new Dog();

public void feed(Cat a){}
public void feed(Dog a){}
```
因此SUN要提供一套JDBC接口给各大厂家去实现，我们只需面向接口去写代码就可以了，减少开发成本。

![JDBC.png](https://upload-images.jianshu.io/upload_images/6853111-2c14ac2f8de5e815.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)


### JDBC编程六步（需要熟记）
1、注册驱动（作用：告诉Java程序，即将要连接的是哪个厂家的数据库）

2、获取连接（表示JVM的进程和数据库进程之间的通道打开了，这属于进程之间的通信，重量级，使用完之后一定要关闭）

3、获取数据库操作对象（专门执行sql语句的对象）

4、执行sql语句（DDL，DQL，DML...）

5、处理查询结果集（只有当第四步执行的是select语句的时候，才有这第五步处理查询结果集）

6、释放资源（使用完资源之后一定要关闭资源）

第一种注册方式
```java
import java.sql.*;

public class JDBCTest01 {

    public static void main(String[] args) throws SQLException {
        // 1.注册驱动
        Driver driver = new com.mysql.jdbc.Driver();  // 多态，父类型引用指向子类对象
        DriverManager.registerDriver(driver);
        // 2.获取连接
        String url = "jdbc:mysql://127.0.0.1:3306/crashcourse";
        String user = "root";
        String password = "root";
        Connection conn = DriverManager.getConnection(url, user, password);  // 多态
        // com.mysql.jdbc.JDBC4Connection@42110406
        System.out.println(conn);
        // 3.获取连接库操作对象（Statement专门执行sql语句的）
        Statement stmt = conn.createStatement();
        // 4.执行sql
        String sql = "select * from products";
        ResultSet rs = stmt.executeQuery(sql);
        System.out.println(rs.getFetchSize());
        // 5.释放资源
        stmt.close();
        conn.close();
    }
}
```

第二种注册方式（建议使用）
* class文件加载到内存时，会执行静态代码块
* 将驱动和数据库的连接信息放到配置文件中

新建jdbc.properties文件
```java
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://127.0.0.1:3306/crashcourse
user=root
password=root
```
具体操作：
```java
import java.sql.*;
import java.util.ResourceBundle;

public class JDBCTest02 {

    public static void main(String[] args) throws Exception {
        // 1.注册驱动
        // 使用资源绑定器绑定属性配置文件
        ResourceBundle bundle = ResourceBundle.getBundle("jdbc");
        String driver = bundle.getString("driver");
        String url = bundle.getString("url");
        String user = bundle.getString("user");
        String password = bundle.getString("password");
        Class.forName(driver);
        // 2.获取连接
        Connection conn = DriverManager.getConnection(url, user, password);
        // com.mysql.jdbc.JDBC4Connection@42110406
        System.out.println(conn);
    }
}
```
`Class.forName(driver)`的内部执行原理就是加载时执行了com.mysql.jdbc.Driver里的静态代码块
```
public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    public Driver() throws SQLException {
    }

    static {
        try {
            DriverManager.registerDriver(new Driver());
        } catch (SQLException var1) {
            throw new RuntimeException("Can't register driver!");
        }
    }
}
```

查询结果集ResultSet
方法：
* getString(int a)：更加列的索引取值（默认从第一列开始），返回的是字符串
* getString(String column)：根据列名取值
* getInt(String column)：如果数据库里的字段存的是int，可以用该方法返回int类型
```java
        // 4.执行sql
        String sql = "select * from products";
        ResultSet rs = stmt.executeQuery(sql);
        boolean flag1 = rs.next();  // 将游标指向第一行（初始在第0行）
        // 5.处理结果集
        System.out.println(rs.getString(1));  // 获取第一列数据
        System.out.println(rs.getInt(2));  // 获取第二列数据
 
        while (rs.next()){
            System.out.println(rs.getString(1));
        }
```
### SQL注入
在拼接sql语句的时候，传入了恶意的代码，导致返回了预料之外的结果，如：
```java
        String id = "1";
        String userName = "'123@qq.com' or 1=1";
        String sql = "select * from user where id=" + id + " and user_name=" + userName;
        System.out.println(sql);  // select * from user where id=1 and user_name='123@qq.com' or 1=1
```
当执行完这条sql之后，会将user这张表里的所有数据都查出来。

Statement和PreparedStatement
* Statement的缺陷：容易导致SQL注入；使用PreparedStatement，可以对sql语句进行预编译。
* Statement是编译一次执行一次，PreparedStatement是编译一次，可执行N次，因为下次我只要传值过去就行了。PreparedStatement效率高一些。
* PreparedStatement会在编译阶段做类型的安全检查


```java
        Connection conn = DriverManager.getConnection(url, user, password);）
        String sql = "select * from products where vend_id=?";
        // 执行到此次，会发送sql语句框子给DBMS，然后DBMS进行sql语句的预编译
        PreparedStatement ps = conn.prepareStatement(sql);
        ps.setInt(1, 1001);
        ResultSet rs = ps.executeQuery();
        System.out.println(rs);

        while (rs.next()){
            System.out.println(rs.getString(1));
        }
```
注意：**防止SQL注入的功能是MySQL内部机制提供的，不是我们写的Java类去实现的**。底层原理是使用MySQL的预编译
1.执行预编译语句，例如：prepare showUsersByLikeName from 'select * from user where username like ?';
2.设置变量，例如：set @username='%小明%';
3.执行语句，例如：execute showUsersByLikeName using @username;

详细了解底层原理请参考：[JDBC：深入理解PreparedStatement和Statement](https://blog.csdn.net/Marvel__Dead/article/details/69486947)

### JDBC事务
jdbc中的事务是自动提交的，即执行一条DML语句则自动提交一次。但在实际的业务中，通常都是N条DML语句一起提交的（如银行转账功能）
conn.setAutoCommit(false)：开启事务
conn.commit()：提交事务
conn.rollback()：回滚事务
```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.util.ResourceBundle;

public class JDBCTest03 {
    public static void main(String[] args) throws Exception{
        ResourceBundle bundle = ResourceBundle.getBundle("jdbc");
        String driver = bundle.getString("driver");
        String url = bundle.getString("url");
        String user = bundle.getString("user");
        String password = bundle.getString("password");
        Class.forName(driver);
        // 获取连接
        Connection conn = DriverManager.getConnection(url, user, password);
        try{
            conn.setAutoCommit(false);
            String sql = "update products set prod_name=? where prod_id=?";
            PreparedStatement ps = conn.prepareStatement(sql);
            ps.setString(1, "xxx");
            ps.setString(2, "FB");
            int count = ps.executeUpdate();
//            String s = null;  制造异常
//            s.toString();
            conn.commit();

        }catch (Exception e){
            System.out.println("回滚数据");
            conn.rollback();
        }finally {
            conn.close();
        }
    }
}

```

