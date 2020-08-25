## Spring

### Spring的七大模块

<img src="https://gitee.com/hiloj/imgs/raw/master/spring/20200814223709.png" alt="体系结构" style="zoom:67%;" />

+ core：提供了框架的基本组成部分，包括：控制反转和依赖注入功能
+ beans：提供了`BeanFactory`，是工厂模式的实现，管理对象`Bean`
+ context：构建于core包基础上，提供了一种框架式的对象访问方法
+ jdbc：提供了一个JDBC的抽象层，用于简化JDBC
+ aop：提供了面向切面的编程实现，可以自定义拦截器、切点等
+ web：提供了针对Web开发的集成特性
+ test：为测试提供支持

### 用到了哪些设计模式

1. 工厂模式：`BeanFactory`是简单工厂模式的体现，用来创建对象的实例
2. 单例模式：`Bean`默认为单例模式
3. 代理模式：AOP功能用到了`JDK动态代理`和`cglib字节码生成技术`
4. 模板方法：解决代码重复问题，如：`RestTemplate`、`JdbcTemplate`
5. 观察者模式：当一个对象的状态发生变化时，有依赖它的对象都会得到通知被自动更新，如：`ApplicationListener`

### IOC的实现机制

> IOC的实现原理就是工厂模式 + 反射机制

~~~java
interface Fruit {
   public abstract void eat();
 }

class Apple implements Fruit {
    public void eat(){
        System.out.println("Apple");
    }
}

class Factory {
    public static Fruit getInstance(String ClassName) {
        Fruit f=null;
        try {
            f=(Fruit)Class.forName(ClassName).newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return f;
    }
}

class Client {
    public static void main(String[] a) {
        Fruit f=Factory.getInstance("io.github.dunwu.spring.Apple");
        if(f!=null){
            f.eat();
        }
    }
}
~~~

### BeanFactory和applicationContext的区别

![](https://gitee.com/hiloj/imgs/raw/master/spring/20200815104207.png)

### 依赖注入的实现方式

1. 接口注入（Spring4开始已被废弃）
2. Setter方法注入
3. 构造器注入

| 构造器注入         | setter注入         |
| ------------------ | ------------------ |
| 只能全部注入       | 可以部分注入       |
| 不会覆盖setter属性 | 覆盖setter属性     |
| 创建一个新的实例   | 不会创建新实例     |
| 适用于设置很多属性 | 适用于设置少量属性 |

### Bean的配置方式

1. XML配置文件

   + `<bean name="user" class="com.hiloj.bean.User"/>`

2. 基于注解的配置

   ~~~java
   @Component("user")
   public class User {...}
   ~~~

   + 需要在xml文件中开启包扫描`<context:component-scan base-package="com.hiloj.bean">`

3. 基于java的配置（javaConfig）

   ~~~java
   @Configuretion
   public class AppConfig {
       @Bean
       public User user (){
           return new User();
       }
   }
   ~~~

### Bean的作用域

* singleton：单例（默认），每个IOC容器仅有一个单实例 
* prototype：原型，每次请求都会产生一个新的实例
* request：每次请求都会产生一个新的实例，实例仅在当前请求内有效，需要web环境
* session：每次回话都会产生一个新的实例，实例仅在当前会话内有效，需要web环境
* global session：全局回话只产生一个实例，需要web环境

### 单例Bean线程安全么

不安全，Spring没有对单例bean进行多线程的封装处理

对于无状态的数据，如：controlelr、service等，可以认为是线程安全的

对于有状态的数据，因为有数据存储功能，就不是线程安全的

### Spring如何处理线程并发问题

使用`ThreadLocal`，为每一个线程提供一个独立的变量副本，隔离多个线程对数据的访问冲突

### Bean的生命周期

![初始化](https://gitee.com/hiloj/imgs/raw/master/spring/20200815192748.png)

![销毁](https://gitee.com/hiloj/imgs/raw/master/spring/20200815192842.png)

### Spring的内部Bean

当Bean B作为另一个Bean A的属性时，可以将B声明为一个内部Bean

~~~xml
public class A {
	private B b;
	...
}
public class B {
	...
}
bean.xml<bean id=“a" class="com.hiloj.A">
	<property name="b">
        <!--声明为内部bean -->
        <bean class="com.hiloj.B"></bean>
	</property>
</bean>
~~~

### 自动装配的5种方式

![自动装配](https://gitee.com/hiloj/imgs/raw/master/spring/20200815205958.png)

### @Autowired自动装配的过程

启动Spring IOC时，容器自动装载一个后置处理器`AutowiredAnnotationBeanPostProcessor`，当容器扫描到`@Autowired`、`@Resource`或`@Inject`时，就会在IOC容器自动查找需要的bean，并装配给该对象的属性。

使用`@Autowired`时，在容器中查询对应类型的bean：

+ 查询结果为一个，将该bean装配给指定数据
+ 查询结果为多个，根据名称进行查找
+ 查找结果为空，抛出异常，可以通过`required=false`解决

### 事务管理类型

**编程式事务管理**：通过编程的方式管理事务，比较灵活，但后期难维护

**声明式事务管理**：将业务代码和事务管理分离，通过注解和配置来管理事务

### 事务的传播行为

> 当前一个事务方法被另一个事务方法调用时,这个事务方法应该如何运行
>
> 如：A方法调用B方法，方法B是在方法A的事务中运行还是为自己开启一个新事务运行，是由方法B的事务传播行为决定的

Spring在`TransactionDefinition`接口中规定了7种类型的事务传播行为。

事务的传播行为是Spring框架独有的事务增强特性，7种事务传播行为如下：

+ `PROPAGATION_REQUIRED(默认)` 
  + 如果当前没有事务，就创建一个新事务；如果当前存在事务，就加入该事务
+ `PROPAGATION_SUPPORTS`
  + 支持当前事务。如果当前存在事务，就加入该事务；如果当前不存在事务，就以非事务执行

+ `PROPAGATION_MANDATORY`
  + 支持当前事务。如果当前存在事务，就加入该事务；如果当前不存在事务，就抛出异常

+ `PROPAGATION_REQUIRES_NEW`
  + 创建新事务，无论当前存不存在事务，都创建新事务

+ `PROPAGATION_NOT_SUPPORTED`
  + **以非事务方式执行**，如果当前存在事务，就把当前事务挂起

+ `PROPAGATION_NEVER`
  + **以非事务方式执行**，如果当前存在事务，则抛出异常

+ `PROPAGATION_NESTED`
  + 如果当前存在事务，则在嵌套事务内执行；如果当前没有事务，则按`REQUIRED`属性执行

### 事务的隔离级别

Spring有五大隔离级别：

1. `ISOLATION_DEFAULT(默认)`：使用底层数据库设置的隔离级别，数据库是什么，spring就是什么
2. `ISOLATION_READ_UNCOMMITTEN`:未提交读，事务未提交前，可被其他事务读取(会出现幻读、脏读、不可重复读)
3. `ISOLATION_READ_COMMITTED`：提交读，事务提交之后才能被其他事务读取(会出现幻读、不可重复读)，SqlServer默认隔离级别
4. `ISOLATION_REPEATABLE_READ`：可重复读，保证多次读取同一个数据时是一致的(会出现幻读)，MySql默认级别
5. `ISOLATION_SERIALIZABLE`：序列化，代价最高最可靠的隔离级别，防止脏读、不可重复读、幻读

**脏读**：一个事务能够读取另一个事务还未提交的数据

**不可重复读**：一个事务内，多次读同一数据

**幻读**：同一个事务内多次查询返回结果不一致

### Spring AOP和AspectJ AOP的区别

AOP实现的关键在于`代理模式`，AOP代理主要分为静态代理和动态代理。

静态代理的代表为AspectJ；动态代理的代表为Spring AOP

**静态代理**：在编译阶段生成代理类，即编译时增强，会在编译阶段将切面织入到Java字节码中，运行时就是增强之后的AOP对象

**动态代理**：在运行阶段临时为方法生成一个AOP对象，次对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法

### JDK动态代理和CGLIB动态代理的区别

+ JDK动态代理只提供接口的代理，不支持类的代理。核心`InvocationHandler`接口和`Proxy`类。`InvocationHandler`通过`invoke()`方法反射来调用目标类中的代码，动态地将横切逻辑和业务编织在一起；接着，`Proxy`利用`InvocationHandler`动态创建一个符合某一个接口的实例，生成目标类的代理对象
+ CGLIB动态代理在运行时动态地生成指定类的一个子类对象，覆盖其中特定方法并添加增强代码，从而实现AOP。CGLIB通过继承的方式做动态代理，因此，被`final`标记的类无法使用CGLIB动态代理

### AOP的几个名词

* 切面(`@Aspect`)：泛指交叉业务逻辑，如：事务处理、日志处理，是对主业务逻辑的一种增强
* 织入：将切面代码插入到目标对象的**过程**，有三个时间点可以进行织入
  + 编译期：在目标类编译时被织入，如AsceptJ
  + 类加载期：在目标类加载到JVM时被织入。需要特殊的类加载器，可以在目标类被引入应用之前增强目标类的字节码
  + 运行期：切面在应用运行的某个时刻被织入，如：SpringAOP
* 连接点(`JoinPoint`)：**切面可以织入的位置**
* 切入点(`@Pointcut`)：**切面具体织入的位置**
  * 切点函数(`execution(方法修饰符(可选) 返回类型 方法名 参数 异常模式(可选))`)
    + 参数允许的通配符：`*`(任意字符，只能匹配一个元素)、`..`(任意字符，匹配多个元素，表示类时，必须和`*`联合使用)、`+`(必须在类名后面，表示类本身和继承或扩展指定类 的所有类)
  * 其他函数
    + `@annotation`：表示标注了指定注解的目标类方法
    + `@args/args`：通过目标类方法的参数类型指定切点
    + `@within/within`：通过类名指定切点
    + `@target/target`：通过类名指定，同事包含包含所有子类
* 通知：切面的一种实现。定义了增强代码切入到目标代码的**时间点**
* 顾问：切面的另一种实现，将通知包装为更复杂切面的装配器，可以指定切入时间点和具体的切入点

~~~java
@Aspect // 切面
@Component
public class LogAspect {
    // 切入点
    @Pointcut("execution(* com.hiloj.springboot.controller.*.*(..))")
    public void log(){}

    // 5种通知方式
    @Before("log()")
    public void before(){
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();
        System.out.println("@Before获取URL:" + request.getRequestURL());
        System.out.println("@Before获取请求方法：" + request.getMethod());
    }

    @After(value = "log()")
    public void after(JoinPoint jp){
        System.out.println("@After方法入参" + jp.getThis());
        System.out.println("@After方法最后执行...");
    }

    @AfterReturning(returning = "result",pointcut = "log()")
    public void afterReturn(Object result){
        System.out.println("@AfterReturning方法返回值：" + result);
    }

    @AfterThrowing("log()")
    public void afterThrowing(){
        System.out.println("@AfterThrowing方法执行异常...");
    }

    @Around("log()")
    public void around(ProceedingJoinPoint jp){
        System.out.println("@Around方法环绕start...");
        Object proceed = null;
        try {
            proceed = jp.proceed();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        System.out.println("@Around方法环绕的结果为" + proceed);
        System.out.println("@Around方法环绕end...");
    }

    // annotation注解，括号中参数为接口
    @Pointcut(value = "@annotation(com.hiloj.springboot.config.MyLog)")
    public void logTest(){}

    @Before("logTest()")
    public void test(){
        System.out.println("@annotation before");
    }

    // 注解中的log对应方法入参log
    @Around("@annotation(log)")
    public Object test2(ProceedingJoinPoint pjp, MyLog log){
        System.out.println("注解的值为：" + log.info());
        try {
            Object proceed = pjp.proceed();
            return  proceed;
        } catch (Throwable throwable) {
            throwable.printStackTrace();
            return null;
        }
    }
}
~~~

### 通知的五种类型

* 前置通知(`@Before`)：在目标方法被调用前调用通知
* 后置通知(`@After`)：在目标方法完成之后调用通知，不关心方法的输出
* 返回通知(`@AfterReturning`)：在目标方法成功执行之后调用通知
* 异常通知(`@AfterThrowing`)：在目标方法抛出异常之后调用通知
* 环绕通知(`@Around`)：在目标方法执行前、后都调用通知

### IOC初始化流程

IOC容器初始化分为三个过程实现：

1. Resource(即BeanDefinition)资源定位，是容器找数据的过程
2. BeanDefinition的载入，是把用户定义好的Bean表示成IOC容器内部的数据结构(BeanDefinition)
3. 向IOC容器中注册这些BeanDefinition的过程，是将第二步的BeanDefinition保存到HashMap中

### 循环依赖

#### 什么是循环依赖

