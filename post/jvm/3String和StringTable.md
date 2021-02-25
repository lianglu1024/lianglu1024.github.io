# String和StringTable

StringTable就是一个HashTable，key就是字符串，value是字符串对象。在1.8之后是存在堆内存里。

由于字符串是程序中使用最频繁的，所以使用到的字符串会缓存到StringTable里，提高效率。

![image-20210224193447774](https://tva1.sinaimg.cn/large/008eGmZEly1gnyush5kgjj313s0qgtib.jpg)

记住：

* **使用双引号括起来的字符串是存在StringTable中的**
* **用双引号括起来的字符串是不可变的，也就是"abc"自出生到死亡不可变**
* 使用`new String("a") + new String("b")`拼接生成的ab是放在堆里的

```java
public class Test {
    // 这两行代码底层在方法区字符串常量池中创建了3个字符串对象
    public static void main(String[] args) {

        String s1 = "abc";
        String s2 = s1 + "xyz";
    }
}
```

