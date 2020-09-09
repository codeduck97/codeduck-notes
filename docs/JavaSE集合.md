[TOC]

---

# Java集合框架

## Collection接口

> 集合类的基本接口是Collection接口。该接口有两个基本方法：

```
public interface Collection<E>
{
    ...
    
    boolean add(E element);
    Iterator<E> iterator();
    
    ...
}
```

- add 方法用于向集合中添加元素。如果添加元素确实改变了集合就返回true, 如果集合没有发生变化就返回false
- iterator 方法用于返回一个实现了Iterator 接口的对象。可以使用这个迭代器对象依次访问集合中的元素

------

## 迭代器

> Iterator 接口包含4 个方法

```
public interface Iterator<E>
{
    E next();
    boolean hasNext();
    void remove();
    default void forEachRemaining(Consumer<? super E> action);
}
```

- next 方法，可以逐个访问集合中的每个元素，如果到达集合末尾，则会抛出NoSuchElementException异常，应配合hasNext方法使用
- hasNext方法，如果迭代器对象还有多个供访问的元素， 这个方法就返回true

> 査看集合中的所有元素，则请求一个迭代器，并在hasNext 返回true 时反复地调用next 方法

```
Collection<String> c = ...;
Iterator<String> iter = c.iterator();

while(iter.hasNext())
{
    String element = iter.next();
    ...
}
```

**PS：**如果对正在被迭代的集合进行结构上的改变（即对该集合使用add、remove或clear方法），那么迭代器就不再合法（并且在其后使用该迭代器时将会有 Concurrent ModificationException异常被抛出）。

为避免迭代器准备给出某一项作为下一项（next item）而该项此后或者被删除，或者也许一个新的项正好插入该项的前面这样一些讨厌的情形，有必要记住上述法则。

这意味着，只有在需要立即使用一个迭代器的时候，我们才应该获取迭代器。然而，如果迭代器调用了它自己的 remove方法，那么这个迭代器就仍然是合法的。这是有时候我们更愿意使用迭代器的 remove方法。

- **使用for each循环遍历集合**

```
Collection<String> c = ...;
foreach(String element : c)
{
    ...
}
    
```

- **JavaSE 8中，可以调用forEachRemaining 方法并提供一个lambda表达式（它会处理一个元素）。将对迭代器的每一个元素调用这个lambda 表达式**

```
Collection<String> c = new ArrayList<>();

c.add("e1");
c.add("e2");
c.add("e3");
c.add("e4");

Iterator<String> iter = c.iterator();

iter.forEachRemaining(element -> System.out.println(element));
```

**PS：元素被访问的顺序取决于集合类型 **

​	如果对ArrayList 进行迭代， 迭代器将从索引0 开始， 每迭代一次， 索引值加 1

​	如果访问HashSet 中的元素， 每个元素将会按照某种随机的次序出现

- Java迭代器，查找操作与位置变更是紧密相连的。查找一个元素的唯一方法是调用next，而在执行查找操作的同时，迭代器的位置随之向前移动
- 应该将Java 迭代器认为是位于两个元素之间。当调用next 时， 迭代器就越过下一个元素，并返回刚刚越过的那个元素的引用
- Iterator 接口的remove 方法将会删除上次调用next 方法时返回的元素，如果想要删除指定位置上的元素， 必须越过这个元素

![img](https://images.cnblogs.com/cnblogs_com/code-duck/1798082/o_200702134401image-20200606213226474.png)

```
Iterator<String> iter = c.iterator();

iter.next();
iter.remove();//next与remove方法相互依赖，必须先越过元素，然后将返回的元素删除
```

------

## 集合框架中的接口

 **List 是一个有序集合（ordered collection），元素会增加到容器中的特定位置。可以采用两种方式访问元素：使用迭代器访问， 或者使用一个整数索引来访问。后一种方法称为随机访问（random access )，因为这样可以按任意顺序访问元素**

> List接口继承了Collection接口，因此它包含Collection接口的所有方法，外加其他一些方法

![img](https://images.cnblogs.com/cnblogs_com/code-duck/1798082/o_200706025557image-20200706103907568.png)

 

**Set接口等同于Collection接口，不过其方法的行为有更严谨的定义。集（ set )的 add 方法不允许增加重复的元素。要适当地定义集的equals 方法：只要两个集包含同样的元素就认为是相等的。hashCode 方法的定义要保证包含相同元素的两个集，会得到相同的散列码**

 **Queue 队列接口指出可以在队列的尾部添加元素，在队列的头部删除元素，并且可以査找队列中元素的个数。**

![img](https://images.cnblogs.com/cnblogs_com/code-duck/1798082/o_200702134405image-20200606213511022.png)

------

# 具体的集合

> 以Map 结尾的类之外， 其他类都实现了Collection 接口，而以Map 结尾的类实现了Map 接口

![img](https://images.cnblogs.com/cnblogs_com/code-duck/1798082/o_200702134418image-20200607111856187.png)

![img](https://images.cnblogs.com/cnblogs_com/code-duck/1798082/o_200702134424image-20200607111926189.png)

## 数组列表

> List是有序的Collection，使用此接口能够精确的控制每个元素插入的位置。用户能够使用索引（元素在List中的位置，类似于数组下标）来访问List中的元素
>
> 访问元素的方式有：
>
> ​	1、使用迭代器顺序访问，
>
> ​	2、使用get 和 set 方法随机访问。
>
> 后者不适合链表，但是非常适合数组和数组列表 ( ArrayList 为List接口的实现类)

- ArrayList的数据结构
  - 其底层数据结构就是一个数组，数组元素的类型为Object类型
- ArrayList的线性安全问题
  - 对ArrayList进行添加元素的操作的时候是分两个步骤进行的
  - 第一步先在object[size]的位置上存放需要添加的元素
  - 第二步将size的值增加1
  - 因此在多线程的环境下不能保证具有原子性，为非线性安全
- 优点
  - ArrayList底层以数组实现，是一种随机访问模式，再加上它实现了RandomAccess接口，因此查找也就是get的时候非常快
  - 实现了Cloneable接口，能被克隆
  - ArrayList实现了Serializable接口，因此它支持序列化，能够通过序列化传输
  - ArrayList在**顺序**添加一个元素的时候非常方便，只是往数组里面添加了一个元素而已
  - 支持快速随机访问，实际上就是通过下标序号进行快速访问
  - 可以自动扩容，默认为每次扩容为原来的1.5倍
- 缺点
  - 插入和删除元素的效率不高，插入(删除)一个元素需要将后面(前面)的所有元素进行移动
  - 根据元素下标查找元素需要遍历整个元素数组
  - 非线性安全(多线程环境下可以考虑用Collections.synchronizedList(List l)函数返回一个线程安全的ArrayList类，也可以使用concurrent并发包下的CopyOnWriteArrayList类)

**在需要动态数组时，可能会使用Vector类，但是为什么会用ArrayList取代Vector类呢?**

> **Vector：** 基于数组（Array）的List，其实就是封装了数组所不具备的一些功能方便我们使用，所以它难易避免数组的限制，同时性能也不可能超越数组

- Vector 类的所有方法都是同步的。可以由两个线程安全地访问一个Vector 对象
- 如果由一个线程访问Vector，代码要在同步操作上耗费大量的时间
- ArrayList 方法不是同步的，因此，建议在不需要同步时使用ArrayList， 而不要使用Vector。

------

## 链表

>  数组和数组列表（ArrayList）都有一个重大的缺陷。这就是从数组的中间位置删除一个元素要付出很大的代价，其原因是数组中处于被删除元素之后的所有元素都要向数组的前端移动，在数组中间的位置上插入一个元素也是如此

 **链表（linked list）解决了这个问题。尽管数组在连续的存储位置上存放对象引用，但链表却将每个对象存放在独立的结点中。每个结点还存放着序列中下一个结点的引用。在 Java 程序设计语言中，所有链表实际上都是双向链接的（doubly linked），即每个结点还存放着指向前驱结点的引用**

![img](https://images.cnblogs.com/cnblogs_com/code-duck/1798082/o_200702134433image-20200607112514755.png)

> 下列代码实现向链表中添加3个元素，然后删除第二个元素

```
List<String> staff = new LinkedList<>(); // LinkedList implements List
staff.add("Amy");
staff.add(MBobH)；
staff.add("Carl");
Iterator iter = staff.iterator();
String first = iter.next();// visit first element
String second = iter.next(); "visit second element
iter.remove(); // remove last visited element
```

- 链表是一个有序集合，LinkedList.add 方法将对象添加到链表的尾部，但常常需要将元素添加到链表中间
- 迭代器非常适合对自然有序的集合使用，Iterator的子接口ListIterator对链表有更好的操作

> Listlterator 接口有两个方法，可以用来反向遍历链表

```
E previous();
boolean hasPrevoius();
```

- Add 方法在迭代器位置之前添加一个新对象

> 下列代码在第一个元素后，第二个元素前添加"Juliet"，此时迭代器位置位于Amy和Bob之间

```
List<String> staff = new LinkedList();
staff.add("Amy");
staff.add("Bob");
staff.add("Carl");
ListIterator<String> iter = staff.listlterator();
iter.next();// skip past first element
iter.add("juliet") ;
```

- set 方法是用一个新元素取代调用next 或previous 方法返回的上一个元素

> 下列代码用一个新值取代链表的第一个元素

```
ListIterator<String> iter = list.listlterator();
String oldValue = iter.next() ; // returns first element
iter.set (newValue) ; // sets first element to newValue
```

- 链表不支持快速地随机访问。如果要查看链表中第 n 个元素，就必须从头开始，越过 n-1 个元素
- 在程序需要采用整数索引访问元素时，通常不选用链表

> 虽然效率不高，但是LinkedList类还是提供了一个用来访问某个特定元素的get方法，**此种访问方式效率极低**

```
LinkedList<String> list = ...;
String obj = list.get(n);
```

 **链表总结：**

> ArrayList在表的两端进行添加和删除都是常数时间的操作，由此 Linkedlist更提供了方法 adafirst和 removefirst、adalat和removelast、以及 getFirst和 getLast等以有效地添加、删除和访问表两端的项

- 使用链表的唯一理由是尽可能地减少在列表中间插人或删除
- 避免使用以整数索引表示链表中位置的所有方法。如果需要对集合进行随机访问， 就使用数组或ArrayList

**java.util.List<E>**

| 方法                                                     | 描述                                                         |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| ListIterator<E> listIterator ( )                         | 返回一个列表迭代器， 以便用来访问列表中的元素。              |
| ListIterator <E> listIterator( int index )               | 返回一个列表迭代器， 以便用来访问列表中的元素， 这个元素是第一次调用next 返回的给定索引的元素 |
| void add( int i , E element )                            | 在给定位置添加一个元素                                       |
| void addAll( int i , Col 1ection<? extends E> elements ) | 将某个集合中的所有元素添加到给定位置                         |
| E remove( int i )                                        | 删除给定位置的元素并返回这个元素                             |
| E get ( int i )                                          | 获取给定位置的元素                                           |
| E set ( int i , E element )                              | 用新元素取代给定位置的元素， 并返回原来那个元素              |
| int indexOf (Object element )                            | 返回与指定元素相等的元素在列表中第一次出现的位置， 如果没有这样的元素将返回-1 |
| int lastlndexOf(Object element)                          | 返回与指定元素相等的元素在列表中最后一次出现的位置， 如果没有这样的元素将返回-1 |

**java.util.Listlterator<E>**

| 方法                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| void add(E newElement) | 在当前位置前添加一个元素                                     |
| void set(E newElement) | 用新元素取代next 或previous 上次访问的元素，如果在next 或previous 上次调用之后列表结构被修改了， 将拋出一个异常 |
| boolean hasPrevious()  | 当反向迭代列表时， 还有可供访问的元素， 返回true             |
| E previous()           | 返回前对象。如果已经到达了列表的头部， 謝te出一hNoSuchElementException 异常 |
| int nextlndex()        | 返回下一次调用next 方法时将返回的元素索引                    |
| int previousIndex()    | 返回下一次调用previous 方法时将返回的元素索引                |

**java.util.LinkedList<E>**

| 方法                                          | 描述                                                |
| --------------------------------------------- | --------------------------------------------------- |
| LinkedList()                                  | 构造一个空链表                                      |
| LinkedList(Col 1ection<? extends E> elements) | 构造一个链表， 并将集合中所有的元素添加到这个链表中 |
| void addFirst(E element)                      |                                                     |
| void addLast(E element)                       | 将某个元素添加到列表的头部或尾部                    |
| E getFirst()                                  |                                                     |
| E getLast()                                   | 返回列表头部或尾部的元素                            |
| E removeFirst()                               |                                                     |
| E removeLast()                                | 删除并返回列表头部或尾部的元素                      |

## 散列集

> 数组和链表在集合中包含很多元素时，查找某一个元素需要耗费很多时间。而散列表（hash table）可以快速的查找到所需的对象

- 散列表为每个对象计算一个整数，称为散列码（hashcode)， 散列码是由对象的实例域产生的一个整数

> 在 Java中，散列表用链表数组实现。每个列表被称为桶（ bucket)，要想査找表中对象的位置，就要先计算它的散列码，然后与桶的总数取余，所得到的结果就是保存这个元素的桶的索引

![img](https://images.cnblogs.com/cnblogs_com/code-duck/1798082/o_200702134414image-20200607102107049.png)

​	**散列表**

- 在JavaSE 8 中， 桶满时会从链表变为平衡二叉树

> 散列表可以用于实现几个重要的数据结构。其中最简单的是set 类型。set 是没有重复元素的元素集合。set 的add 方法首先在集中查找要添加的对象，如果不存在，就将这个对象添加进去

**Java 集合类库提供了一个HashSet类，它实现了基于散列表的集**

- 可以用add 方法添加元素
- 散列集迭代器将依次访问所有的桶，访问顺序是随机的
- 只有不关心集合中元素的顺序时才应该使用HashSet

- ***java.util.HashSet<E>\***

  | 方法                                               | 描述                                                    |
  | -------------------------------------------------- | ------------------------------------------------------- |
  | HashSet ( )                                        | 构造一个空散列表                                        |
  | HashSet (Col 1ection<? extends E> elements )       | 构造一个散列集， 并将集合中的所有元素添加到这个散列集中 |
  | HashSet ( int initialCapacity）                    | 构造一个空的具有指定容量（桶数）的散列集                |
  | HashSet ( int initialCapacity , float 1oadFactor ) | 构造一个具有指定容量和装填因子                          |

- ***java.lang.Object\***

  | 方法            | 描述                                                         |
  | --------------- | :----------------------------------------------------------- |
  | int hashCode( ) | 返回这个对象的散列码。散列码可以是任何整数， 包括正数或负数。equals 和hashCode的定义必须兼容，即如果x.equals(y) 为true, x.hashCode() 必须等于y.hashCode()。 |

------

## 树集

> TreeSet 类与散列集十分类似，不过，它比散列集有所改进。树集是一个有序集合( sorted collection )，每次将一个元素添加到树中时，都被放置在正确的排序位置上。因此，迭代器总是以排好序的顺序访问每个元素

- 有序的结构付出的代价就是，将一个元素添加到树中要比添加到散列表中慢
- 如果树中包含n 个元素， 査找新元素的正确位置平均需要log2 n 次比较。
- TreeSet 类实现了**NavigableSet 接口**。这个接口增加了几个便于定位元素以及反向遍历的方法

- ***java.util.TreeSet<E>\***

| 方法                                       | 描述                                              |
| ------------------------------------------ | ------------------------------------------------- |
| TreeSet()                                  |                                                   |
| TreeSet(Comparator<? super E> comparator)  | 构造一个空树集                                    |
| TreeSet(Col 1ection<? extends E> elements) |                                                   |
| TreeSet(SortedSet<E> s)                    | 构造一个树集， 并增加一个集合或有序集中的所有元素 |

- ***java.util.SortedSet<E>\***

| 方法                               | 描述                             |
| ---------------------------------- | -------------------------------- |
| Comparator ? super E> comparator ) | 返回用于对元素进行排序的比较器   |
| E first()                          |                                  |
| E last()                           | 返回有序集中的最小元素或最大元素 |

- ***java.util.NavigableSet<E>\***

| 方法                             | 描述                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| E higher(E value)                |                                                              |
| E 1ower(E value)                 | 返回大于value 的最小元素或小于value 的最大元素，如果没有这样的元素则返回null |
| E ceilling(E value)              |                                                              |
| E floor(E value)                 | 返回大于等于vaiue 的最小元素或小于等于value 的最大元素， 如果没有这样的元素则返回null |
| E pollFirst()                    |                                                              |
| E poll Last                      | 删除并返回这个集中的最大元素或最小元素， 这个集为空时返回null |
| Iterator<E> descendingIterator() | 返回一个按照递减顺序遍历集中元素的迭代器                     |

## 队列与双端队列

> 队列可以让人们有效地在尾部添加一个元素，在头部删除一个元素。有两个端头的队列，即双端队列，可以让人们有效地在头部和尾部同时添加或删除元素，但不支持在队列中间添加元素

- 在Java SE 6 中引人了 Deque 接口，并由 ArrayDeque 和LinkedList 类实现。这两个类都提供了双端队列，而且在必要时可以增加队列的长度。

- ***java.utii.Queue<E>\***

| 方法                      | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| boolean add(E element )   |                                                              |
| boolean offer(E element ) | 如果队列没有满，将给定的元素添加到这个双端队列的尾部并返回true |
| E remove( )               |                                                              |
| E poll ( )                | 假如队列不空， 删除并返回这个队列头部的元素                  |
| E element( )              |                                                              |
| E peek( )                 | 如果队列不空，返回这个队列头部的元素， 但不删除              |

- ***java.util.Deque<E>\***

| 方法                           | 描述                                        |
| ------------------------------ | ------------------------------------------- |
| void addFirst(E element )      |                                             |
| void addLast(E element )       |                                             |
| boolean offerFirst(E element ) |                                             |
| boolean offerLast( E element ) | 将给定的对象添加到双端队列的头部或尾部      |
| E removeFirst( )               |                                             |
| E removeLast( )                |                                             |
| E pollFirst( )                 |                                             |
| E pollLast（ ）                | 如果队列不空，删除并返回队列头部的元素      |
| E getFirst（）                 |                                             |
| E getLast( )                   |                                             |
| E peekFirst（）                |                                             |
| E peekLast( )                  | 如果队列非空，返回队列头部的元素， 但不删除 |

- ***java.util.ArrayDeque<E>\***

| 方法                             | 描述                                              |
| -------------------------------- | ------------------------------------------------- |
| ArrayDeque(）                    |                                                   |
| ArrayDeque( int initialCapacity) | 用初始容量16 或给定的初始容量构造一个无限双端队列 |

## 优先级队列

> 优先级队列（ priority queue ) 中的元素可以按照任意的顺序插入，却总是按照排序的顺序进行检索

例如：无论何时调用remove 方法，总会获得当前优先级队列中最小的元素。然而，优先级队列并没有对所有的元素进行排序

> **优先级队列的数据结构：堆（heap），堆是一个可以自我调整的二叉树，对树执行添加（add)和删除（remore) 操作，可以让最小的元素移动到根，而不必花费时间对元素进行排序**

- ***java.util.PriorityQueue\***

| 方法                                                        | 描述                                               |
| ----------------------------------------------------------- | -------------------------------------------------- |
| PriorityQueue()                                             |                                                    |
| PriorityQueue(int initialCapacity)                          | 构造一个用于存放Comparable 对象的优先级队列        |
| Pr1orityQueue(int initialCapacity, Comparator ? super E> c) | 构造一个优先级队列，并用指定的比较器对元素进行排序 |

# 映射

> 集是一个集合，它可以快速地查找现有的元素。
>
> 但是，要查看一个元素， 需要有要查找元素的精确副本。这不是一种非常通用的査找方式。
>
> 通常， 我们知道某些键的信息，并想要查找与之对应的元素。
>
> **映射（map) 数据结构就是为此设计的。映射用来存放键/ 值对**

## 映射的基本操作

> Java类库为映射提供了两个通用的实现：**HashMap和TreeMap**，他们都实现了**Map接口**

- 散列映射**对键**进行散列，其优点：操作快
- 树映射**用键**的整体顺序对元素进行排序，并将其组织成搜索树，其优点：元素有序排列

```
Map<String, Employee> staff = new HashMap<>();// HashMap implements Map
Employee harry = new Employee ("Harry Hacker");
staff.put("987-98-9996", harry);
```

- key在映射中值唯一
- remove 方法用于从映射中删除给定键对应的元素
- size 方法用于返回映射中的元素数
- 用 forEach 方法可以提供一个接收键和值的 lambda 表达式（新特性）

```
scores•forEach((k, v) -> System.out .println("key=" + k + ", value:" + v)) ;
```

- ***java.util.Map<K,V>\***

| 方法                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| V get(Object key)                                            | 获取与键对应的值，如果在映射中没有这个对象则返回null         |
| default V getOrDefault(Object key, V defaultValue)           | 获得与键关联的值； 返回与键关联的对象， 或者如果未在映射中找到这个键， 则返回defaultValue |
| V put(K key, V value)                                        | 将键与对应的值关系插入到映射中。如果这个键已经存在， 新的对象将取代与这个键对应的旧对象 |
| void putAll (Map<? extends K , ? extends V> entries)         | 将给定映射中的所有条目添加到这个映射中                       |
| boolean containsKey(Object key)                              | 如果在映射中已经有这个键， 返回true                          |
| boolean containsValue(Object value)                          | 如果映射中已经有这个值， 返回true                            |
| default void forEach(BiConsumer<? super K ,? super V> action) | 对这个映射中的所有键/ 值对应用这个动作                       |

- ***java.util.HashMap<K,V>\***

| 方法                                           | 描述                                                         |
| ---------------------------------------------- | ------------------------------------------------------------ |
| HashMap()                                      |                                                              |
| HashMap(int initialCapacity)                   |                                                              |
| HashMap(int initialCapacity, float loadFactor) | 用给定的容量和装填因子构造一个空散列映射（装填因子是一个0.0 〜1.0 之间的数值,这个数值决定散列表填充的百分比。一旦到了这个比例， 就要将其再散列到更大的表中 )，默认的装填因子是0.75 |

- ***java.util.TreeMap<K,V>\***

| 方法                                                 | 描述                                                         |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| TreeMap()                                            | 为实现Comparable 接口的键构造一个空的树映射                  |
| TreeMap(Comparator<? super K> c)                     | 构造一个树映射， 并使用一个指定的比较器对键进行排序          |
| TreeMap(Map<? extends K, ? extends V> entries)       | 构造一个树映射， 并将某个映射中的所有条目添加到树映射中      |
| TreeMap(SortedMap<? extends K, ? extends V> entries) | 构造一个树映射， 将某个有序映射中的所有条目添加到树映射中，并使用与给定的有序映射相同的比较器 |
| Comparator ? super K> comparator()                   | 返回对键进行排序的比较器。如果键是用Comparable 接口的compareTo 方法进行比较的，返回null |
| K firstKey() / K 1 astKey()                          | 返回映射中最小元素和最大元素                                 |

## 更新映射项

> 待续...

## 映射视图

> ​	集合框架不认为映射本身是一个集合，不过， 可以得到映射的视图（ View ) 这是实现了Collection 接口或某个子接口的对象

- 有 3 种视图集合：
  - 键集
  - 值集合 ( 不是一个集 )
  - 键 / 值对集

```
Set<K> keySet() //返回映射中所有键的一个集视图，可以从这个集中删除元素，它们将从映射中删除，但是不能增加任何元素
    
Collection<V> values()//返回映射中所有值的一个集合视图，可以从这个集中删除元素，键和相关联的值将从映射中删除， 但是不能增加任何元素
    
Set<Map.Entry<K,V>> entrySet() //返回Map.Entry 对象（映射中的键/ 值对）的一个集视图，可以从这个集合中删除元素， 所删除的值及相应的键将从映射中删除， 不过不能增加任何元素
```

- keySet 不是HashSet 或TreeSet ,而是实现了Set接口的另外某个类的对象, Set 接口扩展了 Collection 接口,因此,可以像使用集合一样使用 keySet

## 弱散列映射

> 待续...

## 链接散列集与映射

> ​	LinkedHashSet 和 LinkedHashMap 类用来记住插人元素项的顺序。这样就可以避免在散歹IJ表 中的项从表面上看是随机排列的，当条目插入到表中时，就会并入到双向链表中

![image-20200607100138213](http://null/)

​	**连接散列表**

```
Map<String, Employee>staff = new LinkedHashMap<>();
staff.put ("144-25-5464", new Employee("Amy Lee")) ;
staff.put ("567-24-2546", new Employee("Harry Hacker")) ;
staff.put ("157-62-7935", new Employee("Gary Cooper")) ;
staff.put ("456-62-5527", new Employee("Francesca Cruz"));

//使用staff.keySet().iterator()进行键的枚举
staff.keySet().iterator().forEachRemaining(action -> System.out.println(action));
# 运行结果:
144-25-5464
567-24-2546
157-62-7935
//staff.values().iterator()可以对值进行枚举
```

> 链接散列映射将用访问顺序， 而不是插入顺序， 对映射条目进行迭代。每次调用get 或put, 受到影响的条目将从当前的位置删除， 并放到条目链表的尾部（只有条目在链表中的位置会受影响， 而散列表中的桶不会受影响。一个条目总位于与键散列码对应的桶中）

- 要项构造这样一个的散列映射表， 请调用

```
LinkedHashMap<K, V>(initialCapacity, loadFactor, true)
```

- ***java.util.LinkedHashSet<E>\***

  | 方法                                                 | 描述                                       |
  | ---------------------------------------------------- | ------------------------------------------ |
  | LinkedHashSet()                                      |                                            |
  | LinkedHashSet(int initialCapacity)                   |                                            |
  | LinkedHashSet(int initialCapacity, float 1oadFactor) | 用给定的容量和填充因子构造一个空链接散列集 |

- ***java.util.LinkedHashMap<E>\***

| 方法                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| LinkedHashMap()                                              |                                                              |
| LinkedHashMap(int initialCapacity）                          |                                                              |
| LinkedHashMap(int initialCapacity, float loadFactor)         |                                                              |
| LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder) | 用给定的容量、填充因子和顺序构造一个空的链接散列映射表。accessOrder 参数为true 时表示访问顺序， 为false 时表示插入顺序 |
| protected boolean removeEldestEntry(Map.Entry<K, V > eldest) | 如果想删除eldest 元素， 并同时返回true, 就应该覆盖这个方法，eldest 参数是预期要删除的条目，这个方法将在条目添加到映射中之后调用。其默认的实现将返回false。即在默认情况下，旧元素没有被删除 |

## 枚举集与映射

> EnumSet 是一个枚举类型元素集的高效实现。由于枚举类型只有有限个实例， 所以EnumSet 内部用位序列实现。如果对应的值在集中， 则相应的位被置为1

- EnumSet 类没有公共的构造器。可以使用静态工厂方法构造这个集：

```
enum Weekday { MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY };
EnumSet<Weekday> always = EnumSet.allOf(Weekday.class);
EnumSet<Weekday> never = EnumSet.noneOf(Weekday.class);
EnumSet<Weekday> workday = EnumSet.range(Weekday.MONDAY, Weekday.FRIDAY);
EnumSet<Weekday> mwf = EnumSet.of(Weekday.MONDAY, Weekday.WEDNESDAY, Weekday.FRIDAY);
```

- EnumMap 是一个键类型为枚举类型的映射。它可以直接且高效地用一个值数组实现

```
EnumMap<Weekday, Employee> personlnCharge = new EnumMap<>(Weekday.class);
```

- ***java.util.EnumSet<E extents Enum<E>>\***

| 方法                                                         | 描述                                                      |
| ------------------------------------------------------------ | --------------------------------------------------------- |
| static < E extends Enum< E>> EnumSet< E> allOf(Class<E> enumType) | 返回一个包含给定枚举类型的所有值的集                      |
| static < E extends Enum< E>> EnumSet< E > noneOf(Class< E > enumType) | 返回一个空集，并有足够的空间保存给定的枚举类型所有的值    |
| static < E extends Enum< E >> EnumSet< E > range(E from, E to) | 返回一个包含from -to 之间的所有值（包括两个边界元素）的集 |
| static < E extends Enum< E>> EnumSet< E > of(E value)        |                                                           |
| static < E extends Enum< E>> EnumSet< E > of(E value, E... values) | 返回包括给定值的                                          |

## 标识散列映射

> ​	类 IdentityHashMap 中键的散列值不是用hashCode 函数计算的， 而是用System.identityHashCode 方法计算的，这是Object.hashCode 方法根据对象的内存地址来计算散列码时所使用的方式。在对两个对象进行比较时， IdentityHashMap 类使用==, 而不使用equals。

- 不同的键对象， 即使内容相同， 也被视为是不同的对象。在实现对象遍历算法（如对象串行化）时， 这个类非常有用， 可以用来跟踪每个对象的遍历状况。