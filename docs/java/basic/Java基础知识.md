# 1. Java基础知识

## 1.1 Java基本概念

## 1.2 Java语法

## 1.3 基本数据类型

## 1.4 Java方法

# 2. Java面向对象

## 类和对象

## 面向对象的三大特征

## 修饰符

# 3. Java核心技术

## 反射机制

### 反射机制概述

反射机制允许程序在运行期间借助反射获取任何类的内部信息，并且能直接操作任意对象的内部属性和方法。

具体来说：当JVM完成类加载后（[可参考](https://codeduck97.github.io/codeduck-notes/#/docs/java/jvm/6.%E8%99%9A%E6%8B%9F%E6%9C%BA%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6)），方法区中就会产生类的Class对象（一个类对应一个Class对象），该Class对象包含该类的全部信息。而实例化后的对象存在于Java堆中，每个实例化对象都会存储Class对象的内存地址（[可参考](https://codeduck97.github.io/codeduck-notes/#/docs/java/jvm/2.Java%E5%AF%B9%E8%B1%A1%E7%9A%84%E8%A7%A3%E6%9E%84)），如下图所示:point_down:

![image-20210103193756813](images/image-20210103193756813.png)

**反射的具体操作为：**通过实例化后的对象（man或women）获取对应的Class全部信息（Person信息），进而获取到类的全限定名。

**反射可实现的功能如下：**

- 在运行时判断任意一个对象所属的类；
- 在运行时构造任意一个类的对象；
- 在运行时判断任意一个类所具有的成员变量和方法；
- 在运行时获取泛型信息；
- 在运行时调用任意一个对象的成员变量和方法；
- 在运行时处理注解；
- 生成动态代理；

### 反射常用的类

- Java.lang.Class；

- Java.lang.reflect.Constructor；

- Java.lang.reflect.Field；

- Java.lang.reflect.Method；

- Java.lang.reflect.Modifier；

### 反射的基本使用

通过反射来调用类的构造器、属性、方法。

- 创建Person类；

```java
public class Person {

    // 私有属性
    private String name;

    public Integer age;

    public Person() {
    }
    
    // 私有构造器
    private Person(String name) {
        this.name = name;
    }

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

	// 省略set和get方法……

    public void say() {
        System.out.println("hello, nice to meet you!");
    }

    private void sayHobby(String hobby) {
        System.out.println("my hobbies are " + hobby);
    }

    @Override
    public String toString() {
        return "Person[" +
                "name='" + name + '\'' +
                ", age=" + age +
                ']';
    }
}
```

- 通过反射调用Person类中的`public`构造器，属性，方法；

```java
public void test01() throws Exception{
    Class clazz = Person.class;

    // 创建 Person 对象
    Constructor cons = clazz.getConstructor(String.class, Integer.class);
    Object obj = cons.newInstance("zs", 11);
    System.out.println(obj);    // Person[name='zs', age=11]

    // 调用对象指定的属性
    Field age = clazz.getField("age");
    age.set(obj, 22);
    System.out.println(obj);    // Person[name='zs', age=22]

    // 调用对象指定的方法
    Method say = clazz.getMethod("say" );
    say.invoke(obj);            // hello, nice to meet you!
}
```

- 通过反射调用Person类中的`private`构造器，属性，方法；

```java
@Test
public void test02() throws Exception {
    Class clazz = Person.class;

    // 调用私有构造器
    Constructor cons = clazz.getDeclaredConstructor(String.class);
    // 对所有属性设置访问权限  当类中的成员变量为private时 必须设置此项
    cons.setAccessible(true);

    // 创建实例对象
    Object codeduck = cons.newInstance("codeduck");
    System.out.println(codeduck);       // Person[name='codeduck', age=null]

    // 调用私有属性
    Field name = clazz.getDeclaredField("name");
    name.setAccessible(true);
    name.set(codeduck, "jason");
    System.out.println(codeduck);       // Person[name='jason', age=null]

    // 调用私有方法
    Method sayHobby = clazz.getDeclaredMethod("sayHobby", String.class);
    sayHobby.setAccessible(true);
    sayHobby.invoke(codeduck, "sing-jump-rap-ball");   // my hobbies are sing-jump-rap-ball
}
```

通过上述代码可以发现，反射的使用打破了Java代码的封装性。

### java.lang.Class 类

Class对象的实例其实就是将Class对应的静态字节流存储结构加载到方法区内，而生成的一个运行时数据结构。可以通过以下几种方式来获取方法区的Class对象。**PS**：有关类的加载[可参考](https://codeduck97.github.io/codeduck-notes/#/docs/java/jvm/6.%E8%99%9A%E6%8B%9F%E6%9C%BA%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6)

```java
@Test
public void test03() throws ClassNotFoundException {
    // 1. 调用运行时类的属性: .class
    Class<Person> clazz1 = Person.class;
    System.out.println(clazz1);     // class com.jason.javabase.reflection.Person

    // 2. 通过运行时类的对象调用 getClass()方法
    Person person = new Person();
    Class<? extends Person> clazz2 = person.getClass();
    System.out.println(clazz2);     // class com.jason.javabase.reflection.Person

    // 3. 调用 Class 的静态方法: forName(String classPath) 
    Class<?> clazz3 = Class.forName("com.jason.javabase.reflection.Person");
    System.out.println(clazz3);     // class com.jason.javabase.reflection.Person
    
    // 4. 使用类加载器ClassLoader
    ClassLoader classLoader = TestReflection.class.getClassLoader();
    Class<?> clazz4 = classLoader.loadClass("com.jason.javabase.reflection.Person");
    System.out.println(clazz4);     // class com.jason.javabase.reflection.Person
    
    // 判断四个Class引用是否指向同一个Class对象
    System.out.println(clazz1 == clazz2); // true
    System.out.println(clazz2 == clazz3); // true
    System.out.println(clazz3 == clazz4); // true
}
```

其中第三种方式的获取使用较多，众多开发框架中使用此方法获取Class实例对象。

**Class类的其他方法：**

- getFields（）：获取当前运行时类及其父类中声明为public访问权限的属性；
- getDeclareFields（）：获取当前运行时类中声明的所有属性（不包含父类中声明的属性）；
- getMethods（）：获取当前运行时类及其所有父类中声明为public权限的方法；
- getDeclareMethods（）：获取当前运行时类中声明的所有方法（不包含父类中声明的方法）；
- getConstructor（）：获取当前运行时类中声明的public权限的构造器；
- getDeclareConstructor（）：获取当前运行时类中声明的所有构造器；
- getSuperclass（）：获取运行时类的父类；
- getGenericSuperclass（）：获取运行时类的带泛型的父类；
- getInterfaces（）：获取运行时类实现的接口；
- getPackage（）：获取运行时类所在的包；
- getAnnotations（）：获取运行时类声明的注解；

**Class实例可以是哪些数据结构？**

```java
Class c1 = Object.class;
Class c2 = Comparable.class;
Class c3 = String[].class;
Class c4 = int[][].class;
Class c5 = ElementType.class;
Class c6 = Override.class;
Class c7 = int.class;
Class c8 = void.class;
Class c9 = Class.class;
```

**使用ClassLoader加载配置文件：**

- 在src目录下创建配置文件jdbc.properties

```
name=codeduck
password=123123
```

- 测试加载

```java
@Test
public void test01() throws Exception{
    Properties properties = new Properties();

    // 1. 使用IO流读取配置文件信息
    FileInputStream in = new FileInputStream("src//jdbc.properties");
    // properties.load(in);

    // 2. 通过类加载器获取
    ClassLoader classLoader = TestClassLoader.class.getClassLoader();
    InputStream is = classLoader.getResourceAsStream("jdbc.properties");

    properties.load(is);
    String name = properties.getProperty("name");
    String pw = properties.getProperty("password");
    System.out.println(name + "," + pw);        // codeduck,123123
}
```

### 通过反射创建运行时类的对象

```java
@Test
public void test01() throws Exception {
    Class<Person> clazz = Class.forName("com.jason.javabase.reflection.Person");

    /**
         * 使用此方法创建对应的运行时类的对象，内部调用了运行时类的空参构造方法
         * 因此
         *   1. 运行时类必须提供空参的构造器，
         *   2. 空参的构造器的访问权限能够对外暴露（public）
         * 下列方法使用较多！而调用有参构造器创建对象使用较少
         */
    Person person = clazz.newInstance();
    System.out.println(person);		// Person[name='null', age=null]
}
```

### 反射应用——动态代理

#### 代理的基本概念

**代理模式的定义：**由于某些原因需要给某对象提供一个代理以控制对该对象的访问。这时，访问对象不适合或者不能直接引用目标对象，代理对象作为访问对象和目标对象之间的中介。

**动态代理：**指客户通过代理类来调用其它对象的方法，并且是在程序运行时根据需要动态创建目标类的代理对象。

**代理模式的主要优点有：**

- 代理模式在客户端与目标对象之间起到一个中介作用和保护目标对象的作用；
- 代理对象可以扩展目标对象的功能；
- 代理模式能将客户端与目标对象分离，在一定程度上降低了系统的耦合度，增加了程序的可扩展性

#### 代理模式的结构

1. 抽象主题（Subject）类：通过接口或抽象类声明真实主题和代理对象实现的业务方法。
2. 真实主题（Real Subject）类：实现了抽象主题中的具体业务，是代理对象所代表的真实对象，是最终要引用的对象。
3. 代理（Proxy）类：提供了与真实主题相同的接口，其内部含有对真实主题的引用，它可以访问、控制或扩展真实主题的功能。

根据代理的创建时期，代理模式分为静态代理和动态代理。

- 静态：由程序员创建代理类或特定工具自动生成源代码再对其编译，在程序运行前代理类的 .class 文件就已经存在了；
- 动态：在程序运行时，运用反射机制动态创建而成；

#### 动态代理的实现

```java
// 抽象主题
interface Human{

    void getGender(String gender);

    void saySomething(String talk);
}

// 真实主题: 被代理类
class Liming implements Human{

    @Override
    public void getGender(String gender) {
        System.out.println("I am " + gender);
    }

    @Override
    public void saySomething(String talk) {
        System.out.println(talk);
    }
}

/**
 * 代理类——实现动态代理:
 *      1. 如何根据加载到内存的被代理类，动态的创建一个代理类及其对象
 *      2. 通过代理类对象调用方法时，如何动态的去调用被代理类中的同名方法
 */

class ProxyFactory{

    // 调用此方法，返回一个代理类的对象，解决问题一
    public static Object getProxyInstance(Object obj) {
        MyInvocationHandler handler = new MyInvocationHandler();

        // InvocationHandler绑定代理对象
        handler.bind(obj);

        // 返回代理对象实例
        return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(), handler);
    }
}

class MyInvocationHandler implements InvocationHandler {

    private Object obj;

    public void bind(Object obj) {
        this.obj = obj;
    }

    /**
     * desc: 通过代理类的对象，调用方法 xxx() 时，就会自动的调用如下方法 invoke()
     * <p>
     *
     * @param proxy 代理对象
     * @param method 代理对象 的 xxx() 方法
     * @param args xxx() 方法 的参数
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        // method: 即为代理类对象调用的方法，此方法作为被代理类对象要调用的方法
        Object returnValue = method.invoke(obj, args);

        // 返回上述方法的返回值
        return returnValue;
    }
}

public class TestProxy {
    public static void main(String[] args) {
        Liming liming = new Liming();
        Human proxyInstance = (Human) ProxyFactory.getProxyInstance(liming);
        proxyInstance.saySomething("I am Liming`s proxyInstance");  // I am Liming`s proxyInstance
        proxyInstance.getGender("male");    // I am male
    }
}
```

## 多线程

# 4. Java8 新特性

## Lambda 表达式

Lambda允许把函数作为一个方法的参数（函数作为参数传递进方法中）。使用 Lambda 表达式可以使代码变的更加简洁紧凑。

### Lambda语法

Java8中引入了一个新的操作符 “—>” 该操作符称为箭头操作符 或 Lambda操作符。箭头操作符将Lambda表达式拆分为两部分：

- 左侧：Lambda表达式的参数列表（可以想象成，是上面定义的接口中抽象方法参数的列表）；
- 右侧：Lambda表达式中，所需要执行的功能，即Lambda体（需要对抽象方法实现的功能），若右侧只有一行代码，则可省略{}，否则必须添加{}；

### 基本使用

```java
@Test
public void test01() {

    // 不使用Lambda
    new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("hello world");
        }
    }).start();

    // 使用Lambda允许把函数作为一个方法的参数
    new Thread(() -> {
        System.out.println("hi world");
    }).start();
    
    // 使用匿名内部类，重写Intger的 compare方法
    Comparator<Integer> c = new Comparator<Integer>() {
        @Override
        public int compare(Integer o1, Integer o2) {
            return Integer.compare(o1, o2);
        }
    };

    // 使用有参Lambda表达式
    Comparator<Integer> comparator = (x, y) -> Integer.compare(x, y);
    System.out.println(comparator.compare(1, 2));   // -1
}
```

## 函数式接口

### 四大核心函数式接口

1. Consumer<T>：消费型接口，包含方法 void accept（T t）表示对类型为T的对象应用操作；
2. Supplier<T> ：供给型接口，包含方法 T get（）返回类型为T的对象；
3. Function<T,R>：函数型接口，包含方法 R apply（T t）操作T类型对象，返回R类型对象；
4. Predicate<T>：断言型接口，包含方法 boolean test（T t）确定T类型对象是否满足约束条件；

### Consumer 接口

测试用例一

```java
@Test
public void test01(){
    // 创建字符串对象
    StringBuilder sb = new StringBuilder("sb字符串后面将会跟随####");
    
    // 声明函数对象 consumer
    Consumer<StringBuilder> consumer = (str) -> str.append("大SB");
    
    // 调用Consumer.accept()方法接收参数
    consumer.accept(sb);
    
    System.out.println(sb.toString());	// sb字符串后面将会跟随####大SB
}
```

测试用例二

```java
@Test
public void test01() {
    whoAmI("codeduck", new Consumer<String>() {     // I am codeduck
        @Override
        public void accept(String name) {
            System.out.println("I am " + name);
        }
    });

    // 使用Lambda表达式
    whoAmI("jason", (name) -> System.out.println("I am " + name));  // I am jason
}

public void whoAmI(String name, Consumer<String> consumer) {
    consumer.accept(name);
}
```

### Supplier 接口

```java
@Test
public void test01(){
    List<Integer> nums = getNums(5, () -> {
        Random random = new Random();
        return random.nextInt(10);
    });
    nums.forEach(n -> {
        System.out.print(n + "\t");     // 生成5个随机数，且随机数范围为 0 - 9
    });
}

public List<Integer> getNums(Integer n, Supplier<Integer> supplier) {
    List<Integer> list = new ArrayList<>();
    for (int i = 0; i < n; i++) {
        list.add(supplier.get());
    }
    return list;
}
```

### Function 接口

```java
@Test
public void test01() {
    int len = strLen("adc", (x) -> x.length());
    System.out.println("String length is " + len);  // String length is 3
}

public int strLen(String s, Function<String,Integer> function){
    return function.apply(s);
}
```

### Predicate 接口

```java
@Test
public void test01(){
    List<String> list = Arrays.asList("codeduck", "jason", "ls", "ww", "zs");
    List<String> rs = strPredict(list, (x) -> x.length() > 3);
    rs.forEach(System.out::println);	// codeduck jason
}

/**
 * 将满足条件的字符串，放入到集合中
 */
public static List<String> strPredict(List<String> list, Predicate<String> predicate) {
    List<String> result = new ArrayList<>();
    list.forEach(item -> {
        if(predicate.test(item)) {
            result.add(item);
        }
    });
    return result;
}
```

## 方法的引用

**方法的引用：**通过类或对象的引用，使用`::`来调用类中的静态方法或实例对象中的方法。

### 对象::实例方法

```java
@Test
public void test01() {
    Person person = new Person("zs",12);
    Supplier<String> supplier = person::getName;
    System.out.println(supplier.get());     // zs
}
```

### 类::静态方法

```java
@Test
public void test02() {

    // Lambda表达式
    Comparator<Integer> comparator = (x, y) -> Integer.compare(x, y);

    // 方法引用
    Comparator<Integer> comparator2 = Integer::compare;
}
```

### 类::new

```java
@Test
public void test03() {

    // Lambda表达式
    Supplier<Person> supplier01 = ()->new Person("zs",12);


    // 方法的引用调用无参构造器
    Supplier<Person> supplier02  = Person::new;
    Person person = supplier02.get();

    // 方法的引用调用有参构造器
    Function<Integer,Person> supplier03  = Person::new;
    Person person1 = supplier03.apply(22);
    System.out.println(person1.getAge());       // 22
}
```

### 数组类型::new

```java
@Test
public void test04() {

    // Lambda表达式创建指定容量的String数组
    Function<Integer, String[]> function = (x) -> new String[x];
    String[] s = function.apply(20);

    // 数组引用方式创建指定容量大小的String数组
    Function<Integer, String[]> function1= String[]::new;
    String[] str = function1.apply(10);
}
```

## Stream（流）

### 何为 Stream

[引自](https://www.runoob.com/java/java8-streams.html)Stream（流）是一个来自数据源的元素队列并支持聚合操作

- 元素是特定类型的对象，形成一个队列。 Java中的Stream并不会存储元素，而是按需计算。因此使用集合存储数据（内存方面），使用流计算数据（CPU方面）；
- **数据源** 流的来源。 可以是集合，数组，I/O channel， 产生器generator 等；
- **聚合操作** 类似SQL语句一样的操作， 比如filter, map, reduce, find, match, sorted等；

 Stream操作有两个基础的特征：

- **Pipelining**: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)；
- **内部迭代**： 以前对集合遍历都是通过Iterator或者For-Each的方式，显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了内部迭代的方式， 通过访问者模式(Visitor)实现；

### 流的创建方法

在 Java 8 中, 集合接口有两个方法来生成流：

- **stream()** − 为集合创建串行流。
- **parallelStream()** − 为集合创建并行流。

#### 通过集合生成流

```java
// 通过Collection集合的 stream() 或者 parallelStream() 获取流。
List<String> list = new ArrayList<>();

// 返回一个顺序流
Stream<String> stream =  list.stream();

// 返回一个并行流
Stream<String> parallelStream = list.parallelStream();
```

#### 通过数组生成流

```java
String[] s = new String[3];

// 通过Arrays 中的静态方法，获取数组流
Stream<String> stream = Arrays.stream(s);
```

#### 通过Stream生成流

```
Stream<Integer> stream = Stream.of(1, 2, 3);
```

#### 生成无限流

```
Stream<Integer> stream = Stream.iterate(0, x -> x + 2 );
```

### Stream 的操作

**中间操作：**一个中间操作链，对数据源的数据进行处理；

**终止操作：**执行中间操作链，并产生结果；

多个中间操作可以连接起来形成一个流水线，除非流水线上触发终止操作，否者中间操作不会执行任何的处理，而在终止操作时一次性全部处理，这样被称为惰性求值。

#### Stream 数据处理

- filter（Predicate p）：接收Lambda表达式，从流中过滤某些元素；
- distinct（）：筛选，通过流所生成的hashCode（）和equals（）去除重复元素；
- limit（long maxSize）：截断流，使其元素不超过给定数量；
- skip（long n）：跳过元素，返回一个扔掉了前n个元素的流，若流中元素不足n个，则返回一个空流；

```java
List<Person> persons = Arrays.asList(
    new Person("zs",13),
    new Person("codeduck",23),
    new Person("jason",33),
    new Person("ls",43),
    new Person("ww",53)
);

// Person[name='zs', age=13]
// Person[name='codeduck', age=23]
persons.stream().filter(x -> x.getAge() < 40).limit(2).forEach(System.out::println);

// Person[name='ls', age=43]
// Person[name='ww', age=53]
persons.stream().skip(3).forEach(System.out::println);
```

#### Stream 映射

- map（Function f），将元素转换成其它形式或提取信息，接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新元素；

- flatMap（Function f） 接收一个函数作为参数，将流中的每个值都转换成另一个流，然后把所有流连接成一个流；
- mapToDouble（ToDoubleFunction f）接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的DoubleStream；
- mapToLong（ToLongFunction f）接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的LongStream；
- mapToInt（ToIntFunction f）接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的IntStream；

```java
@Test
public void test04(){
    List<String> names = Arrays.asList("codeduck", "jason", "zs");

    // 输出 CODEDUCK JASON ZS
    names.stream().map(x -> x.toUpperCase()).forEach(System.out::println);

    // 将String流转为Character流
    // 输出 codeduckjasonzs
    names.stream().flatMap(TestStream::fromStringToStream).forEach(System.out::print);
}

public static Stream<Character> fromStringToStream(String str) {
    List<Character> list = new ArrayList<>();
    for (Character c : str.toCharArray()) {
        list.add(c);
    }
    return list.stream();
}
```

#### Stream 排序

排序方法可分为：

- sorted()：自然排序
- sorted(Comparator com)：定制排序

```java
    @Test
    public void test05() {
        List<Integer> list = Arrays.asList(4, 1, 2, 4, 5);
        // 自然排序
        list.stream().sorted().forEach(System.out::println);

        List<Person> persons = Arrays.asList(
                new Person("zs",13),
                new Person("codeduck",23),
                new Person("jason",13),
                new Person("ls",19),
                new Person("ww",15)
        );
        
        // 制定排序规则
        persons.stream().sorted((x,y) -> {
            if (x.getAge() == y.getAge()) 
                return x.getName().compareTo(y.getName());
            else 
                return Integer.compare(x.getAge(), y.getAge());
        }).forEach(System.out::println);
    }
```

运行结果：

```
1
2
4
4
5
Person[name='jason', age=13]
Person[name='zs', age=13]
Person[name='ww', age=15]
Person[name='ls', age=19]
Person[name='codeduck', age=23]
```

#### Stream 终止操作

执行下列操作后，Stream流就会进行终止执行。

- allMatch：检查是否匹配所有元素；
- anyMatch：检查是否至少匹配一个元素；
- noneMatch：检查是否一个都没匹配；
- findFirst：返回第一个元素；
- findAny：返回当前流中任意一个元素；
- count：返回流中元素的个数；
- max：返回当前流中最大值；
- min：返回当前流中最小值；
- forEach：内部迭代；

#### Stream 归约

- reduce（T identity，BinaryOperator b）将流中元素反复结合起来，得到一个值，返回T

- reduce（BinaryOperator b）将流中元素反复结合起来，得到一个值，返回Optional<T>

```java
@Test
public void test06() {
    List<Integer> list = Arrays.asList(1, 2, 3, 4, 5 ,6 ,7 ,8 , 9, 10);
    // 按照下面的规则进行累加操作

    // reduce()归约 第一个参数为初始值，第二参数为运算函数
    Integer sum = list.stream().reduce(0, (x, y) -> x + y); 

    // 直接传入运算函数
    Optional<Integer> reduce = list.stream().reduce((x, y) -> x + y);

    System.out.println(sum);            // 55
    System.out.println(reduce.get());   // 55
}
```

#### Stream 收集

- collect（Collector c）将流转换成其它形式，接收一个Collector接口实现，用于给Stream中元素做汇总的方法；

Collector接口实现方法的实现决定了如何对流执行收集操作（如收集到List，Set，Map）。另外Collectors实用类提供了很多静态方法，可以方便地创建常用收集器实例。

```java
    @Test
    public void test07() {
        
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5 ,6 ,7 ,8 , 9, 10);

        // 转为set集合
        Set<Integer> set = list.stream().collect(Collectors.toSet());

        // 转为List集合
        List<Integer> collect = list.stream().collect(Collectors.toList());

        // 转为ArrayList集合
        ArrayList<Integer> arrayList = list.stream().collect(Collectors.toCollection(ArrayList::new));

        // 转为Map集合
        Map<String, Integer> map = list.stream().collect(Collectors.toMap(x -> "key" + x, y -> y));

        // 求和
        Integer sum = list.stream().collect(Collectors.summingInt(Integer::intValue)); // 55
    }
```

## Optional 类

Optional 类是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。

Optional 是个容器：它可以保存类型T的值，或者仅仅保存null。Optional提供很多有用的方法，这样我们就不用显式进行空值检测。

Optional 类的引入很好的解决空指针异常。

### 常用方法

- Optional.of：创建一个Optional实例
- Optional.empty：创建一个空的Optional实例
- Optional.ofNullable：若t不为null，创建optional实例，否者创建一个空实例
- isPresent：判断是否包含值
- orElse(T other)：如果存在该值，返回值， 否则返回 other。
- orElseGet(Supplier other )：如果存在该值，返回值， 否则触发 other，并返回 other 调用的结果。
- map(Function f)：如果有值对其处理，返回处理后的Optional，否则返回Optional.empty()
- flatMap(Function mapper)：与map类似，要求返回值必须是Optional

### 基本使用

```java
@Test
public void test01(){
    Person person = null;

    Optional<Person> optionalPerson = Optional.ofNullable(person);

    System.out.println(optionalPerson); 		// Optional.empty

    Person codeduck = new Person("codeduck", 22);
    Optional<Person> optional = Optional.ofNullable(codeduck);
    
    System.out.println(optional);   			// Optional[Person[name='codeduck', age=22]]
    System.out.println(optional.get()); 		// Person[name='codeduck', age=22]
    System.out.println(optional.isPresent());   // true
}
```

