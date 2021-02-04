# mybatis Available parameters问题

引入我司自己的parent依赖之后，发布到测试环境，当读取数据库的时候报下面的错误：

```
Caused by: org.apache.ibatis.binding.BindingException: Parameter 'countryId' not found. Available parameters are [arg1, arg0, param1, param2]
```

原因是Mapper接口里的方法参数名前没有加`@Param`注解。可是我以前没加也是可以的呀？

## 获取方法参数名

在java8里，按下面的方式获取方法的参数名，最终得到的是`arg0`

```java
public class Demo01 {

    public void test(Long id){}

    public static void main(String[] args) throws NoSuchMethodException {
        Method method = Demo01.class.getMethod("test", Long.class);
        for (Parameter parameter : method.getParameters()) {
            System.out.println(parameter.getName());  // arg0
        }
    }
}
```

如果想获取test方法的参数名id，需要编译源码的时候加上`- parameters`

```
javac - parameters Demo01
```

这个时候生成的class文件里的参数名就是id了

## 原因追溯

如果我们引入的是spring boot parent依赖包

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.0.6.RELEASE</version>
</parent>
```

会默认帮我们引入编译插件，其中

```xml
<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <parameters>true</parameters>
    </configuration>
</plugin>
```

这样maven编译打包的时候就会加上parameters参数进行编译。如果你不手动指定这个选项，那么就没法获取到方法参数名，只能在参数名前加`@Param`

