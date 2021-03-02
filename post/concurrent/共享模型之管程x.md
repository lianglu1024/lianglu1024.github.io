## synchronized

synchronized语法

```java
synchronized(对象)  //对象（block，线程2(block)
{
  临界区
}
```

方法上的synchronized

```java
class Test{
  public synchronized void test(){
    
  }
}
等价于
class Test{
  public void test(){
    synchronized(this){
      
    }
  }
}
```

```java
class Test{
  public synchronized static void test(){
    
  }
}
等价于
class Test{
  public static void test(){
    synchronized(Test.class){
      
    }
  }
}
```

## 常见线程安全类

* String
* Integer
* StringBuffer
* Random
* Vector
* Hashtable
* java.util.concurrent包下的类

这里说它们是线程安全的是指，**多个线程调用它们同一个实例的方法时**，是线程安全的。也可以理解为

```java
Hashtable table = new Hashtable();
new Thread(()->table.put("key","value1");).start()
new Thread(()->table.put("key","value2");).start()
```

