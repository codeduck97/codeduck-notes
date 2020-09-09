## Supplier接口

### Supplier接口源码

> 该接口是一个提供者的意思，只有一个 get（） 方法
>
> 当传入一个泛型T对象，则可使用 get（）方法返回 该对象实例的引用

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

**创建对象consumer**

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

**使用Supplier接口获取对象的引用**

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

