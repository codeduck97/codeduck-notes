

# Java8新特性

## Supplier接口

### 源码分析

该接口是一个提供者的意思，只有一个 get（） 方法，当传入一个泛型T对象，则可使用 get（）方法返回 该对象实例的引用。

```java
@FunctionalInterface
public interface Supplier<T> {
 
    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

### Supplier接口实战



创建Consumer类

```java
public static class Consumer {
        private String name;
        public Consumer() {
 
        }
        public Consumer(String name) {
            super();
            this.name = name;
        }
        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
        }
}
```



使用Supplier接口获取对象的引用

```java
// 1、创建String类型的实例，并由supplier引用
Supplier<String> supplier = String::new;
System.out.println(supplier.get());	//  ""

// 2、创建Consumer对象的实例，并由supplier引用
Supplier<Consumer> supplierCon = Consumer::new;

// 使用supplier.get()方法返回该实例的引用
Consumer consumer = supplierCon.get();
consumer.setName("我是消费者");

System.out.println(consumer.getName()); // 我是消费者
```



## Consumer接口

### Consumer接口实战

实践一：Comsumer函数式接口

```java
/**
 * Consumer函数式接口测试
 */
@Test
public void testFunction01(){
    // 创建字符串对象
    StringBuilder sb = new StringBuilder("sb字符串后面将会跟随####");
    // 声明函数对象 consumer
    Consumer<StringBuilder> consumer = (str) -> str.append("大SB");
    // 调用Consumer.accept()方法接收参数
    consumer.accept(sb);
    System.out.println(sb.toString());
}
```

运行结果：

```
sb字符串后面将会跟随####大SB
```



实践二：BiConsumer函数式接口

```java
@Test
public void testFunction02(){
    // 创建字符串对象
    StringBuilder sb = new StringBuilder();
    // 声明函数对象 consumer
    BiConsumer<String,String> consumer = (str1, str2) -> {
        // 拼接字符串
        sb.append(str1);
        sb.append(str2);
    };
    // 调用Consumer.accept()方法接收参数
    consumer.accept("我是参数01","，我是参数02。我们被BiConsumer.accept(T,V)接收并处理了");
    System.out.println(sb);
}
```

运行结果：

```
我是参数01,我是参数02。我们被BiConsumer.accept(T,V)接收并处理了
```



### BiConsumer接口源码分析

```java
package sourcecode.analysis;

import java.util.Objects;

/**
 * to operate via side-effects.
 * 本函数接口特征:
 * 1.输入参数2个.
 * 2.无输出结果
 * 3.本函数接口和Consumer函数接口唯一区别:
 * 4.和其它函数接口不同的是:BiConsumer接口的操作是通过其副作用而完成的.
 * 5.本函数接口功能方法:accept(t,u)
 *
 * @param <T> 第一个操作参数类型
 * @param <U> 第二个操作参数类型
 *
 * @see java.util.function.Consumer
 * @since 1.8
 */
@FunctionalInterface
public interface BiConsumer<T, U> {

    /**
     * 本方法的调用,会对输入参数执行指定的行为
     * @param t 第一个输入参数
     * @param u 第二个输入参数
     */
    void accept(T t, U u);

    /**
     * andThen方法,会执行两次Consumer接口的accept方法.两次执行顺序上,先对输入参数执行accept()方法;然后
     * 再对输入参数执行一次after.accept()方法.(注意:两次均为对输入参数的操作,after操作并不是对第一次accept结果的操作)
     * 这两次任何一次accept操作出现问题,都将抛异常到方法调用者处.
     * 如果执行accept这一操作出现异常,fater操作将不会执行.
     * @return 一个按顺序执行的组合的{BiConsumer} 操作后面跟着{@code after}操作
     */
    default BiConsumer<T, U> andThen(BiConsumer<? super T, ? super U> after) {
        Objects.requireNonNull(after);

        return (l, r) -> {
            accept(l, r);
            after.accept(l, r);
        };
    }
}

```



### MybatisPlus中的函数式接口的调用分析



```java
/**
 * 获取 SqlStatement
 *
 * @param sqlMethod ignore
 * @return ignore
 */
protected String sqlStatement(SqlMethod sqlMethod) {
    return SqlHelper.table(entityClass).getSqlStatement(sqlMethod.getMethod());
}

@Override
public boolean saveBatch(Collection<User> entityList, int batchSize) {
    // 获取insert插入语句：INSERT INTO xxx VALUES xxx
    String sqlStatement = sqlStatement(SqlMethod.INSERT_ONE);
    
    // 执行批量删除操作
    // 参数一：entity集合
    // 参数二：执行次数
    // 参数三 : 接受两个参数的函数接口并执行方法 sqlSession.insert(sqlStatement,user)方法
    return executeBatch(entityList, batchSize,
                        (sqlSession, user) -> sqlSession.insert(sqlStatement,user));
}
```

```java
/**
 * 执行批量操作
 *
 * @param list      数据集合
 * @param batchSize 批量大小
 * @param consumer  执行方法
 * @param <E>       泛型
 * @return 操作结果
 * @since 3.3.1
 */
protected <E> boolean executeBatch(Collection<E> list, int batchSize, BiConsumer<SqlSession, E> consumer) {
    Assert.isFalse(batchSize < 1, "batchSize must not be less than one");
    return !CollectionUtils.isEmpty(list) && executeBatch(sqlSession -> {
        int size = list.size();
        int i = 1;
        for (E element : list) {
            
            // comsumer对象调用方法accept接受参数 sqlSession和user对象
            // 然后执行sqlSession.insert(sqlStatement,user)方法
            consumer.accept(sqlSession, element);
            
            if ((i % batchSize == 0) || i == size) {
                // 刷新批处理语句：最终执行insert插入语句
                sqlSession.flushStatements();
            }
            i++;
        }
    });
}
```





## 方法的引用

### 概念及使用

方法的引用：**通过类或对象的引用，使用`::`来调用类中的静态方法或实例对象中的方法**

下面，我们在 Car 类中定义了 4 个方法作为例子来区分 Java 中 4 种不同方法的引用。

```java
package com.runoob.main;
 
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
 
class Car {
    //Supplier是jdk1.8的接口，这里和lamda一起使用了
    public static Car create(final Supplier<Car> supplier) {
        return supplier.get();
    }
 
    public static void collide(final Car car) {
        System.out.println("Collided " + car.toString());
    }
 
    public void follow(final Car another) {
        System.out.println("Following the " + another.toString());
    }
 
    public void repair() {
        System.out.println("Repaired " + this.toString());
    }
}
```



其语法有

**构造器引用：**它的语法是Class::new，或者更一般的`Class< T >::new` 实例如下：

```java
final Car car = Car.create( Car::new );
final List< Car > cars = Arrays.asList( car );
```

**静态方法引用：**它的语法是`Class::static_method` 实例如下：

```java
cars.forEach( Car::collide );
```

**特定对象的方法引用：**它的语法是`instance::method `实例如下：

```
final Car police = Car.create( Car::new );
cars.forEach( police::follow );
```

**特定类的任意对象的方法引用：**它的语法是`Class::method`实例如下：

```java
@Test
public void test03(){
    List names = new ArrayList();

    names.add("Google");
    names.add("Runoob");
    names.add("Taobao");
    names.add("Baidu");
    names.add("Sina");

    // 使用::实现对特定类的方法
    names.forEach(System.out::println);
}
```

在下列中 consumer 是对 println()方法的引用，使用上述基本用法则是:

```java
Consumer<String> consumer = (String x) -> {
  synchronized (this) {
      print(x);
      newLine();
	}
}

// 使用accept()方法接收参数
consumer.accept(x)		// 然后执行方法体函数
```

以上参考：https://www.runoob.com/java/java8-new-features.html



### forEach源码分析

```java
// 参考下列源码有
// 1、action是对PrintStream.println()方法的引用
default void forEach(Consumer<? super T> action) {
    Objects.requireNonNull(action);
    
    for (T t : this) {
        // 2、接受参数 t ，调用方法println(String x)并执行
        action.accept(t);
    }
}
```

**System类**

```java
public final class System {
	public final static PrintStream out = null;
}
```

**public class PrintStream.println**

```java
public void println(String x) {
    synchronized (this) {
        print(x);
        newLine();
    }
}
```

