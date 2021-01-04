# Java8 新特性

## 1. Lambda 表达式

Lambda允许把函数作为一个方法的参数（函数作为参数传递进方法中）。使用 Lambda 表达式可以使代码变的更加简洁紧凑。

### 1.2 Lambda语法

Java8中引入了一个新的操作符 “—>” 该操作符称为箭头操作符 或 Lambda操作符。箭头操作符将Lambda表达式拆分为两部分：

- 左侧：Lambda表达式的参数列表（可以想象成，是上面定义的接口中抽象方法参数的列表）；
- 右侧：Lambda表达式中，所需要执行的功能，即Lambda体（需要对抽象方法实现的功能），若右侧只有一行代码，则可省略{}，否则必须添加{}；

### 1.3 基本使用

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

## 2. 函数式接口

### 2.1 四大核心函数式接口

1. Consumer<T>：消费型接口，包含方法 void accept（T t）表示对类型为T的对象应用操作；
2. Supplier<T> ：供给型接口，包含方法 T get（）返回类型为T的对象；
3. Function<T,R>：函数型接口，包含方法 R apply（T t）操作T类型对象，返回R类型对象；
4. Predicate<T>：断言型接口，包含方法 boolean test（T t）确定T类型对象是否满足约束条件；

### 2.2 Consumer 接口

代表了接受一个输入参数并且无返回的操作。

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

### 2.2 Supplier 接口

无参数，返回一个结果。

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

### 2.3 Function 接口

接受一个输入参数T，返回一个结果R。

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

### 2.4 Predicate 接口

接受一个输入参数，返回一个布尔值结果。

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

## 3. 方法的引用

**方法的引用：**通过类或对象的引用，使用`::`来调用类中的静态方法或实例对象中的方法。

### 3.1 对象::实例方法

```java
@Test
public void test01() {
    Person person = new Person("zs",12);
    Supplier<String> supplier = person::getName;
    System.out.println(supplier.get());     // zs
}
```

### 3.2 类::静态方法

```java
@Test
public void test02() {

    // Lambda表达式
    Comparator<Integer> comparator = (x, y) -> Integer.compare(x, y);

    // 方法引用
    Comparator<Integer> comparator2 = Integer::compare;
}
```

### 3.3 类::new

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

### 3.4 数组类型::new

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

## 4. Stream（流）

### 4.1 何为 Stream

[引自](https://www.runoob.com/java/java8-streams.html)Stream（流）是一个来自数据源的元素队列并支持聚合操作

- 元素是特定类型的对象，形成一个队列。 Java中的Stream并不会存储元素，而是按需计算。因此使用集合存储数据（内存方面），使用流计算数据（CPU方面）；
- **数据源** 流的来源。 可以是集合，数组，I/O channel， 产生器generator 等；
- **聚合操作** 类似SQL语句一样的操作， 比如filter, map, reduce, find, match, sorted等；

 Stream操作有两个基础的特征：

- **Pipelining**: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)；
- **内部迭代**： 以前对集合遍历都是通过Iterator或者For-Each的方式，显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了内部迭代的方式， 通过访问者模式(Visitor)实现；

### 4.2 流的创建方法

在 Java 8 中, 集合接口有两个方法来生成流：

- **stream()** − 为集合创建串行流。
- **parallelStream()** − 为集合创建并行流。

#### 4.2.1 通过集合生成流

```java
// 通过Collection集合的 stream() 或者 parallelStream() 获取流。
List<String> list = new ArrayList<>();

// 返回一个顺序流
Stream<String> stream =  list.stream();

// 返回一个并行流
Stream<String> parallelStream = list.parallelStream();
```

#### 4.2.2 通过数组生成流

```java
String[] s = new String[3];

// 通过Arrays 中的静态方法，获取数组流
Stream<String> stream = Arrays.stream(s);
```

#### 4.2.3 通过Stream生成流

```
Stream<Integer> stream = Stream.of(1, 2, 3);
```

#### 4.2.4 生成无限流

```
Stream<Integer> stream = Stream.iterate(0, x -> x + 2 );
```

### 4.3 Stream 的操作

**中间操作：**一个中间操作链，对数据源的数据进行处理；

**终止操作：**执行中间操作链，并产生结果；

多个中间操作可以连接起来形成一个流水线，除非流水线上触发终止操作，否者中间操作不会执行任何的处理，而在终止操作时一次性全部处理，这样被称为惰性求值。

#### 4.3.1 Stream 数据处理

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

#### 4.3.2 Stream 映射

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

#### 4.3.3 Stream 排序

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

#### 4.3.4 Stream 终止操作

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

#### 4.3.5 Stream 归约

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

#### 4.3.6 Stream 收集

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

## 5. Optional 类

Optional 类是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。

Optional 是个容器：它可以保存类型T的值，或者仅仅保存null。Optional提供很多有用的方法，这样我们就不用显式进行空值检测。

Optional 类的引入很好的解决空指针异常。

### 5.1 常用方法

- Optional.of：创建一个Optional实例
- Optional.empty：创建一个空的Optional实例
- Optional.ofNullable：若t不为null，创建optional实例，否者创建一个空实例
- isPresent：判断是否包含值
- orElse(T other)：如果存在该值，返回值， 否则返回 other。
- orElseGet(Supplier other )：如果存在该值，返回值， 否则触发 other，并返回 other 调用的结果。
- map(Function f)：如果有值对其处理，返回处理后的Optional，否则返回Optional.empty()
- flatMap(Function mapper)：与map类似，要求返回值必须是Optional

### 5.2 基本使用

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

