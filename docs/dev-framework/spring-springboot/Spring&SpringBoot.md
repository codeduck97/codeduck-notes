# Spring框架的组成部分

下图对应的是 Spring4.x 版本。目前最新的5.x版本中 Web 模块的 Portlet 组件已经被废弃掉，同时增加了用于异步响应式处理的 WebFlux 组件。

![image-20201216191730698](https://jason-01.oss-cn-hangzhou.aliyuncs.com/public/image/markdown/image-20201216191730698.png)

- **Spring Core：** Spring 其他所有的功能都需要依赖于该类库。主要提供 IOC 依赖注入功能。
- **Spring Aspects** ： 该模块为与AspectJ的集成提供支持。
- **Spring AOP** ：提供了面向切面的编程实现。
- **Spring JDBC** : Java数据库连接。
- **Spring JMS** ：Java消息服务。
- **Spring ORM** : 用于支持Hibernate等ORM工具。
- **Spring Web** : 为创建Web应用程序提供支持。
- **Spring Test** : 提供了对 JUnit 和 TestNG 测试的支持。

Spring 官网列出的 Spring 的 6 个特征:

- **核心技术** ：依赖注入(DI)，AOP，事件(events)，资源，i18n，验证，数据绑定，类型转换，SpEL。
- **测试** ：模拟对象，TestContext框架，Spring MVC 测试，WebTestClient。
- **数据访问** ：事务，DAO支持，JDBC，ORM，编组XML。
- **Web支持** : Spring MVC和Spring WebFlux Web框架。
- **集成** ：远程处理，JMS，JCA，JMX，电子邮件，任务，调度，缓存。
- **语言** ：Kotlin，Groovy，动态语言。

# Spring IOC 与 AOP

## IOC

IoC（Inverse of Control 控制反转）是一种**设计思想**，就是 **将原本在程序中手动创建对象的控制权，交由Spring框架来管理。** IoC 在其他语言中也有应用，并非 Spring 特有。 **IoC 容器（IoC Container）是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个Map（key，value）,Map 中存放的是各种对象。**

[参考](https://www.zhihu.com/question/23277575/answer/169698662)可知**依赖倒置原则（Dependency Inversion Principle ）**、**控制反转( Inversion of Control )**、**依赖注入（Dependency Injection）**、**控制反转容器（IoC Container）**之间的关系如下：

![image-20201216202424481](https://jason-01.oss-cn-hangzhou.aliyuncs.com/public/image/markdown/image-20201216202424481.png)

**依赖倒置原则**（Dependence Inversion Principle）是程序要依赖于抽象接口，不要依赖于具体实现。

**控制反转**实现的常见方式**依赖注入**：通过控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体将其所依赖的对象的引用传递给它。也可以说，依赖被注入到对象中。

将对象之间的相互依赖关系交给 IoC 容器来管理，并由 IoC 容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。 **IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。**

手动创建多依赖实例过程：

![image-20201216204741045](images/image-20201216204741045.png)

IoC容器帮助创建多依赖实例过程：

![image-20201216204425679](https://jason-01.oss-cn-hangzhou.aliyuncs.com/public/image/markdown/image-20201216204425679.png)

与手动创建实例不同的是，IoC容器使用依赖注入实现多依赖实例的创建。

## AOP

AOP(Aspect-Oriented Programming:面向切面编程)能够将那些与业务无关，**却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码**，**降低模块间的耦合度**，并**有利于未来的可拓展性和可维护性**。

在软件开发中，分布于应用多处的功能被称为 **横切关注点**，例如：日志、安全、事务管理等。将这些横切关注点与业务逻辑相分离（持久层数据的处理等）正是**面向切面编程（AOP）**所要解决的。将横切关注点模块化为特殊的类时，这些类被成为**切面**。

![image-20201217114357220](https://jason-01.oss-cn-hangzhou.aliyuncs.com/public/image/markdown/image-20201217114357220.png)

先了解一下AOP的专业术语：

- **通知**（Advice）:  定义了**切面是什么、如何执行**以及**何时使用**（Before、After、After-returning、After-throwing、Around）。
- **连接点**（Joinpoint）:  我们的应用可能需要对数以千计的时机应用通知，这些时机被称为连接点，它表示应用执行过程中能够插入切面的一个点，这个点可以是方法的调用、异常的抛出。**在 Spring AOP 中，连接点就是类中的所有方法。**其他AOP框架（AspectJ和Jboss）支持字段和构造器的连接点。
- **切点**（PointCut）:  一个切面不需要通知应用的所有连接点，这时切点可以帮我们缩小切面所通知连接点的范围。它定义了在何处应用通知。
- **切面**（Aspect）: 切面是通知和切点的结合。通知和切点共同定义了切面的全部内容——他是什么、在何时、何处完成其功能。
- **引入**（Introduction）：引入允许我们向现有的类添加新的方法或者属性。
- **织入**（Weaving）:  织入是将切面应用到目标对象来创建新的代理对象（Proxy Object）的过程。切面在指定的连接点被织入到目标对象中，可在目标对象的生命周期中进行织入。

下图为在一个或多个连接点上，将切面的功能（通知）织入到程序的执行过程中。总的来说：通知包含了需要用于多个应用对象的横切行为；连接点是程序执行过程中能够应用通知的所有点；切点定义了通知被应用的具体位置（在哪些连接点）。**其中关键的概念是切点定义了哪些连接点会得到通知**。

![image-20201217152632816](https://jason-01.oss-cn-hangzhou.aliyuncs.com/public/image/markdown/image-20201217152632816.png)

**Springboot中定义日志切面**

```java
/**
 * 切面 = 通知 + 切点
 * 	 切点: 定义哪些类的哪些方法需要拦截
 * 	 通知: 定义拦截方法后要做什么
 */
@Component  // 将对象交给IOC容器管理
@Aspect // 声明当前类是一个切面
public class LogAspect {

    /**
     * 定义切点： @Pointcut(“ 匹配表达式 ”)
     * 			匹配表达式实际上是指定哪些类的方法需要被拦截
     *
     * <p>
     * AOP 切点表达式规则概述
     * 	1. 执行任意公共方法: execution(public *(..))
     * 	2. 执行任意的set方法: execution(* set*(..))
     * 	3. 执行com.xxx.service包下的任意类的任意方法：
     * 		execution(* com.xxx.service.*.*(..))
     * 	4. 执行com.xxx.service包 及其 子包 下的任意类的任意方法：
     * 		execution(* com.xxx.service..*.*(..))
     * <p>
     *
     * 注：表达式中第一个 * 代表方法的修饰范围，
     * 		可选值有：private protected public (* 代表所有范围)
     */
    @Pointcut("execution(* com.duck.code.*.service.*.*(..))")
    public void pointcut() {
        // do nothing
    }

    /**
     * 前置通知: 将通知应用到定义的切点
     * 目标类方法执行前 执行该通知
     */
    @Before(value = "pointcut()")
    public void doBefore() {
        System.out.println("前置通知...");
    }

    /**
     * 目标类方法（无异常）执行后，执行后置通知
     */
    @AfterReturning(value = "pointcut()")
    public void doAfterReturn() {
        System.out.println("后置通知...");
    }

    /**
     * 目标类方法（无异常或有异常）执行后，执行最终通知
     */
    @After(value = "pointcut()")
    public void doAfter() {
        System.out.println("最终通知...");
    }

    /**
     * 目标类方法发生异常，执行异常通知
     */
    @AfterThrowing(value = "pointcut()")
    public void doThrow() {
        System.out.println("异常通知...");
    }


    /**
     * 方法执行前后 通过环绕通知定义响应处理
     * 需要通过显示调用对应的方法，否则无法访问指定方法 joinPoint.proceed()
     *
     * @param joinPoint: 连接点
     * @throws Throwable
     */
    @Around(value = "pointcut()")
    public Object doAround(ProceedingJoinPoint joinPoint) {

        System.out.println("前置通知...");

        Object object = null;
        try {
            object = joinPoint.proceed();
            System.out.println("后置通知... =>" + joinPoint.getTarget());
        } catch (Throwable throwable) {
            throwable.printStackTrace();
            System.out.println("异常通知...");
        } finally {
            System.err.println("最终通知...");
        }
        return object;
    }
}
```



# Spring Bean

## @Component 和 @Bean 的区别

1. 作用对象不同: `@Component` 注解作用于**类**，而`@Bean`注解作用于**方法**。
2. `@Component`通常是通过类路径扫描来自动侦测以及自动装配到Spring容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。`@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 bean,`@Bean`告诉了Spring这是某个类的示例，当我需要用它的时候还给我。
3. `@Bean` 注解比 `Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现。

`@Bean`注解使用示例：

```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }

}
```

上面的代码相当于下面的 xml 配置

```xml
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

下面这个例子是通过 `@Component` 无法实现的。

```java
@Bean
public OneService getService(status) {
    case (status)  {
        when 1:
                return new serviceImpl1();
        when 2:
                return new serviceImpl2();
        when 3:
                return new serviceImpl3();
    }
}
```

## 如何将类声明为 Spring 的 Bean

我们一般使用 `@Autowired` 注解自动装配 bean，要想把类标识成可用于 `@Autowired` 注解自动装配的 bean 的类,采用以下注解可实现：

- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个Bean不知道属于哪个层，可以使用`@Component` 注解标注。
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao层。
- `@Controller` : 对应 Spring MVC 控制层，主要用于接受用户请求并调用 Service 层返回数据给前端页面。

## @AutoWired 和 @Resources 的区别

- `@Resource`和`@Autowired`都可以用来装配bean，都可以用于字段或setter方法。
- `@Autowired`默认**按类型**装配，默认情况下要求依赖对象必须存在，如果要允许null值，可以设置它的required属性为false。
- `@Resource`默认**按名称**装配，当找不到与名称匹配的bean时才按照类型进行装配。名称可以通过name属性指定，如果没有指定name属性，当注解写在字段上时，默认取字段名，当注解写在setter方法上时，默认取属性名进行装配。

> 注意：如果name属性一旦指定，就只会按照名称进行装配。

- `@Autowire`和`@Qualifier`配合使用效果和`@Resource`一样：

```java
@Autowired(required = false) @Qualifier("example")
private Example example;

@Resource(name = "example")
private Example example;
```

- `@Resource`装配顺序

1. 如果同时指定name和type，则从容器中查找唯一匹配的bean装配，找不到则抛出异常。
2. 如果指定name属性，则从容器中查找名称匹配的bean装配，找不到则抛出异常。
3. 如果指定type属性，则从容器中查找类型唯一匹配的bean装配，找不到或者找到多个抛出异常
4. 如果都不指定，则自动按照byName方式装配，如果没有匹配，则回退一个原始类型进行匹配，如果匹配则自动装配。

**两者之间的差异**

| 注解对比 | `@Resource` | `@Autowire` |
| -------- | ----------- | ----------- |
| 注解来源 | JDK         | Spring      |
| 装配方式 | 优先按名称  | 优先按类型  |
| 属性     | name、type  | required    |

[引自 zhangchong.blog.csdn.net](https://zhangchong.blog.csdn.net/article/details/79481007)

## Spring 中 Bean 的声明周期

这部分网上有很多文章都讲到了，下面的内容整理自：https://yemengying.com/2016/07/14/spring-bean-life-cycle/ ，除了这篇文章，再推荐一篇很不错的文章 ：https://www.cnblogs.com/zrtqsk/p/3735273.html 。

- Bean 容器找到配置文件中 Spring Bean 的定义。
- Bean 容器利用 Java Reflection API 创建一个Bean的实例。
- 如果涉及到一些属性值 利用 `set()`方法设置一些属性值。
- 如果 Bean 实现了 `BeanNameAware` 接口，调用 `setBeanName()`方法，传入Bean的名字。
- 如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoader`对象的实例。
- 与上面的类似，如果实现了其他 `*.Aware`接口，就调用相应的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessBeforeInitialization()` 方法
- 如果Bean实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。
- 如果 Bean 在配置文件中的定义包含 init-method 属性，执行指定的方法。
- 如果有和加载这个 Bean的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessAfterInitialization()` 方法
- 当要销毁 Bean 的时候，如果 Bean 实现了 `DisposableBean` 接口，执行 `destroy()` 方法。
- 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。

![image-20201217153658324](https://jason-01.oss-cn-hangzhou.aliyuncs.com/public/image/markdown/image-20201217153658324.png)

# SpringMVC工作原理



**流程说明（重要）：**

1. 客户端（浏览器）发送请求，直接请求到 `DispatcherServlet`。
2. `DispatcherServlet` 根据请求信息调用 `HandlerMapping`，解析请求对应的 `Handler`。
3. 解析到对应的 `Handler`（也就是我们平常说的 `Controller` 控制器）后，开始由 `HandlerAdapter` 适配器处理。
4. `HandlerAdapter` 会根据 `Handler `来调用真正的处理器来处理请求，并处理相应的业务逻辑。
5. 处理器处理完业务后，会返回一个 `ModelAndView` 对象，`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
6. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
7. `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
8. 把 `View` 返回给请求者（浏览器）

![image-20201217162727227](https://jason-01.oss-cn-hangzhou.aliyuncs.com/public/image/markdown/image-20201217162727227.png)

# Spring框架用到了哪些设计模式

关于下面一些设计模式的详细介绍，参考[《面试官:“谈谈Spring中都用到了那些设计模式?”。》](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485303&idx=1&sn=9e4626a1e3f001f9b0d84a6fa0cff04a&chksm=cea248bcf9d5c1aaf48b67cc52bac74eb29d6037848d6cf213b0e5466f2d1fda970db700ba41&token=255050878&lang=zh_CN#rd) 。

- **工厂设计模式** : Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理设计模式** : Spring AOP 功能的实现。
- **单例设计模式** : Spring 中的 Bean 默认都是单例的。
- **模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
- **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。
- ......

# Spring事务

## 事务的四大特性

1. **原子性**（Atomicity）：事务中所有操作是不可再分割的原子单位。事务中所有操作要么全部执行成功，要么全部执行失败。
2. **一致性**（Consistency）：事务执行后，数据库状态与其它业务规则保持一致。如转账业务，无论事务执行成功与否，参与转账的两个账号余额之和应该是不变的。
3.  **隔离性**（Isolation）：隔离性是指在并发操作中，不同事务之间应该隔离开来，使每个并发中的事务不会相互干扰。
4. **持久性**（Durability）：一旦事务提交成功，事务中所有的数据操作都必须被持久化到数据库中，即使提交事务后，数据库马上崩溃，在数据库重启时，也必须能保证通过某种机制恢复数据。

## 事务的隔离级别

数据库事务的隔离级别有4种，由低到高分别为**Read uncommitted** 、**Read committed** 、**Repeatable read** 、
**Serializable** 。而且，在事务的并发操作中可能会出现脏读，不可重复读，幻读，事务丢失。下方为事务中专业名词解释：

- **脏读：**（读取未提交的事务，然后该事物回滚了）事务A读取了事务B中尚未提交的数据。如果事务B回滚，则A读取使用了错误的数据。
- **不可重复读：**（读取已提交的新事物，针对更新操作）不可重复读是指在对于数据库中的某个数据，一个事务范围内多次査询却返回了不同的数据值，这是由于在查询间隔，该数据被另一个事务修改并提交了。
- **幻读：**（读取已提交的新事物，针对增删操作）它发生在一个A事务读取了几行数据，接着另一个并发B事务插入（删除）了一些数据时。在随后的查询中，A事务就会发现多了一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读。
- **第一类事务丢失：**（又称回滚丢失）对于第一类事物丢失，就是比如A和B同时在执行一个数据，然后B事物已经提交了，然后A事物回滚了，这样B事物的操作就因A事物回滚而丢失了。
- **第二类事务丢失：**（又称覆盖丢失）对于第二类事物丟失，就是A和B一起执行一个数据，两个同时取到一个数据，然后B事物首先提交，但是A事物加下来又提交，这样就覆盖了B事物。

**事务的隔离级别：**

- **Read uncommitted：**读未提交，顾名思义，就是一个事务可以读取另一个未提交事务的数据。 **可能会导致脏读、幻读或不可重复读**
- **Read committed：** 读提交，顾名思义，就是一个事务要等另一个事务提交后才能读取数据。**可以阻止脏读，但是幻读或不可重复读仍有可能发生**
- **Repeatable read：**重复读，就是在开始读取数据（事务开启）时，不再允许修改操作。 **可以阻止脏读和不可重复读，但幻读仍有可能发生。**
- **Serializable：**Serializable 是最高的事务隔离级别，在该级别下，事务串行化顺序执行，**可以避免脏读、不可重复读与幻读。**但是这种事务隔离级别效率低下，比较耗数据库性能，一般不使用。

在标准的事务隔离级别中，幻读是由更高的隔离级别 SERIALIZABLE 解决的，**但是MySQL InnoDB存储引擎提供Next-Key Lock 锁解决幻读。在Next Key lock算法下，对于索引的扫描，不仅是锁住扫描到的索引，而且还锁住这些索引覆盖的范围（gap）因此在这个范围内的插入都是不允许的。因此InnoDB存储引擎在Repeatable Read隔离级别下就阻止了幻读的出现。**

## Spring 事务中的隔离级别有哪几种?

**TransactionDefinition 接口中定义了五个表示隔离级别的常量：**

- **TransactionDefinition.ISOLATION_DEFAULT:** 使用后端数据库默认的隔离级别，MySQL默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别.
- **TransactionDefinition.ISOLATION_READ_UNCOMMITTED:** 最低的隔离级别，允许读取尚未提交事务的数据，**可能会导致脏读、幻读或不可重复读**
- **TransactionDefinition.ISOLATION_READ_COMMITTED:** 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**
- **TransactionDefinition.ISOLATION_REPEATABLE_READ:** 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。**
- **TransactionDefinition.ISOLATION_SERIALIZABLE:** 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。



## Spring 事务中哪几种事务传播行为?

**什么是事务的传播行为：**假如批量进行十次转账操作是（或不是）一个事务，而转账操作是（或不是）一个事务。当批量转账执行到第四次转账时，批量操作或转账操作出现异常，那么批量转账操作是否需要回滚的，转账操作是否需要回滚。传播行为主要就是解决这类事件。

**支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRED：** 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- **TransactionDefinition.PROPAGATION_SUPPORTS：** 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- **TransactionDefinition.PROPAGATION_MANDATORY：** 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）

**不支持当前事务的情况：**

- **TransactionDefinition.PROPAGATION_REQUIRES_NEW：** 创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NOT_SUPPORTED：** 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- **TransactionDefinition.PROPAGATION_NEVER：** 以非事务方式运行，如果当前存在事务，则抛出异常。

**其他情况：**

- **TransactionDefinition.PROPAGATION_NESTED：** 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。



## Spring中对事务两种支持方式

Spring支持编程式事务管理以及声明式事务管理两种方式。

1. **编程式事务管理**：编程式事务管理是侵入性事务管理，使用TransactionTemplate或者直接使用PlatformTransactionManager，对于编程式事务管理，Spring推荐使用TransactionTemplate。

2. **声明式事务管理**：声明式事务管理建立在AOP之上，其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，执行完目标方法之后根据执行的情况提交或者回滚。

**编程式事务**每次实现都要单独实现，但业务量大功能复杂时，使用编程式事务无疑是痛苦的，而**声明式事务**不同，声明式事务属于无侵入式，不会影响业务逻辑的实现，只需要在配置文件中做相关的事务规则声明或者通过注解的方式，便可以将事务规则应用到业务逻辑中。

**声明事务的方式有两种：**

1. 基于XML的声明式事务
2. 基于注解的声明式事务

Spring并不直接管理事务，而是提供了多种事务管理器，他们将事务管理的职责委托给 Hibernate或者JDBC等持久化机制所提供的相关平台框架的事务来实现。
Spring事务管理器的接口是org.springframework transaction.PlatformTransaction Manager，通过这个接口Spring为各个平台如JDBC、Hibernate等都提供了对应的事务管理器，但是具体的实现就是各个平台自己的事情。

此接口的内容如下：

```java
public interface PlatformTransactionManager extends TransactionManager {
    
    // 由TransactionDefinition 得到 TransactionStatus独享
    TransactionStatus getTransaction(@Nullable TransactionDefinition var1) throws TransactionException;

    // 事务的提交
    void commit(TransactionStatus var1) throws TransactionException;

   	// 事务的回滚
    void rollback(TransactionStatus var1) throws TransactionException;
}
```

## @Transactional(rollbackFor = Exception.class)注解了解吗？

我们知道：Exception分为运行时异常RuntimeException和非运行时异常。事务管理对于企业应用来说是至关重要的，即使出现异常情况，它也可以保证数据的一致性。

当`@Transactional`注解作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。如果类或者方法加了这个注解，那么这个类里面的方法抛出异常，就会回滚，数据库里面的数据也会回滚。

在`@Transactional`注解中如果不配置`rollbackFor`属性,那么事务只会在遇到`RuntimeException`的时候才会回滚,加上`rollbackFor=Exception.class`,可以让事务在遇到非运行时异常时也回滚。

**@Transactional 注解的属性信息**

| 属性名           | 说明                                                         |
| :--------------- | :----------------------------------------------------------- |
| name             | 当在配置文件中有多个 TransactionManager , 可以用该属性指定选择哪个事务管理器。 |
| propagation      | 事务的传播行为，默认值为 REQUIRED。                          |
| isolation        | 事务的隔离度，默认值采用 DEFAULT。                           |
| timeout          | 事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务。 |
| read-only        | 指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true。 |
| rollback-for     | 用于指定能够触发事务回滚的异常类型，如果有多个异常类型需要指定，各类型之间可以通过逗号分隔。 |
| no-rollback- for | 抛出 no-rollback-for 指定的异常类型，不回滚事务              |

关于 `@Transactional ` 注解推荐阅读的文章：[透彻的掌握 Spring 中@transactional 的使用](https://www.ibm.com/developerworks/cn/java/j-master-spring-transactional-use/index.html)