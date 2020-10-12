## 知识点一览

+ 设计原则

  > 7种：`开闭`、`依赖倒置`、`单一职责`、`接口隔离`、`最少知道(迪米特法则)`、`里氏替换`、`合成复用`

+ 创建型模式

  > 6种：`简单工厂`、`工厂方法`、`抽象工厂`、`单例`、`原型`、`建造者`

+ 结构型模式

  > 7种：`适配器`、`装饰`、`代理`、`外观`、`组合`、`桥接`、`享元`

+ 行为型模式

  > 11种：`模板方法`、`观察者`、`访问者`、`观察者`、`中介者`、`责任链`、`迭代器`、`备忘录`、`命令`、`策略`、`状态`

## 7大设计原则

 + 开闭原则：对扩展开放，对修改关闭
 + 依赖倒置：通过抽象使各个类或模块不相互影响，实现松耦合
 + 单一职责：一个类、接口、方法只做一件事
 + 接口隔离：尽量保持接口的纯洁性，客户端不应该依赖不需要的接口
 + 最少知道：迪米特法则，一个类对其所依赖的类知道的越少越好
 + 里氏替换：子类可以扩展父类的功能但不能改变父类原有的功能
 + 合成复用：尽量使用对象组合、聚合，而不是用继承关系达到代码复用的目的

## 6种创建型模式

### 6.1 简单工厂模式

> 通过工厂类的静态方法获取对象

![KKkotI.png](https://s2.ax1x.com/2019/10/20/KKkotI.png)

~~~java
// 定义接口
public interface Computer {
    void show();
}

// 接口实现
public class DellComputer implements Computer {
    public void show() {
        System.out.println("戴尔笔记本");
    }
}

public class HuaWeiComputer implements Computer {
    public void show() {
        System.out.println("华为笔记本");
    }
}

public class XiaoMiComputer implements Computer {
    public void show() {
        System.out.println("小米笔记本");
    }
}

// 工厂类
public class ComputerSimpleFactory {
    public static Computer getComputer(String name) {
        if (name.equalsIgnoreCase("dell")) {
            return new DellComputer();
        } else if (name.equalsIgnoreCase("huawei")){
            return new HuaWeiComputer();
        } else if (name.equalsIgnoreCase("xiaomi")) {
            return new XiaoMiComputer();
        } else {
            return null;
        }
    }
}
~~~

### 6.2 工厂方法模式

> 通过各自的工厂获取对象

![KK3kJU.png](https://s2.ax1x.com/2019/10/20/KK3kJU.png)

~~~java
// 定义工厂类接口
public interface Factory {
    Computer product();
}

// 接口实现,将具体的产品与工厂类的实现绑定
public class DellFactory implements Factory {
    public Computer product() {
        return new DellComputer();
    }
}

public class HuaWeiFactory implements Factory {
    public Computer product() {
        return new HuaWeiComputer();
    }
}

public class XiaoMiFactory implements Factory {
    public Computer product() {
        return new XiaoMiComputer();
    }
}
~~~

### 6.3 抽象工厂模式

> 针对一系列产品(产品簇)

![KMRzWT.png](https://s2.ax1x.com/2019/10/20/KMRzWT.png)

~~~java
// 定义工厂接口,包含所有的产品
public interface AbstractFactory {
    Computer productComputer();

    Television productTelevision();
}

// 具体工厂实现
public class DellAbstractFactory implements AbstractFactory {
    public Computer productComputer() {
        return new DellComputer();
    }

    public Television productTelevision() {
        return new DellTelevision();
    }
}

public class HuaWeiAbstractFactory implements AbstractFactory {
    public Computer productComputer() {
        return new HuaWeiComputer();
    }

    public Television productTelevision() {
        return new HuaWeiTelevision();
    }
}

public class XiaoMiAbstractFactory implements AbstractFactory {
    public Computer productComputer() {
        return new XiaoMiComputer();
    }

    public Television productTelevision() {
        return new XiaoMiTelevision();
    }
}
~~~

### 6.4 单例模式

> 保证对象仅存一份

#### 6.4.1 懒汉式

> **方法调用时**创建对象

~~~java
// 线程不安全
public class LazySingleTon {
    private static LazySingleTon lazySingleTon;

    private LazySingleTon() {}

    public static LazySingleTon getInstance() {
        if (lazySingleTon == null) {
            lazySingleTon = new LazySingleTon();
        }
        return lazySingleTon;
    }
}

// 线程安全的,效率不高
public class SynchronizedLazySingleTon {
    private static SynchronizedLazySingleTon synchronizedLazySingleTon;

    private SynchronizedLazySingleTon() {}

    public static synchronized SynchronizedLazySingleTon getInstance() {
        if (synchronizedLazySingleTon == null) {
            synchronizedLazySingleTon = new SynchronizedLazySingleTon();
        }
        return synchronizedLazySingleTon;
    }
}
~~~

#### 6.4.2 饿汉式

> 初始化时创建对象，线程安全，但是可能造成空间浪费

~~~java
public class HungrySingleTon {
    private static HungrySingleTon hungrySingleTon = new HungrySingleTon();

    private HungrySingleTon() {}

    public static HungrySingleTon getInstance() {
        return hungrySingleTon;
    }
}
~~~

#### 6.4.3 双检锁

> 两次检查对象是否被初始化

~~~java
public class DoubleCheckSingleTon {
    private static volatile DoubleCheckSingleTon doubleCheckSingleTon;

    private DoubleCheckSingleTon() {}

    public static DoubleCheckSingleTon getInstance() {
        if (doubleCheckSingleTon == null) {
            synchronized (DoubleCheckSingleTon.class) {
                if (doubleCheckSingleTon == null) {
                    doubleCheckSingleTon = new DoubleCheckSingleTon();
                }
            }
        }
        return doubleCheckSingleTon;
    }
}
~~~

#### 6.4.4 静态内部类

> 懒加载且线程安全

~~~java
public class StaticInnerSingleTon {
    private StaticInnerSingleTon() {}

    private static class Inner {
        private static final StaticInnerSingleTon staticInnerSingleTon = new StaticInnerSingleTon();
    }

    public static StaticInnerSingleTon getInstance() {
        return Inner.staticInnerSingleTon;
    }
}
~~~

#### 6.4.5 枚举

> 更加简洁，自动支持序列化机制

~~~java
public enum EnumSingleTon {
    INSTANCE;
    public void test(){
        System.out.println("枚举式单例调用实例方法");
    }
}

// 枚举式单例的使用
public void EnumSingleton(){
    EnumSingleTon instance = EnumSingleTon.INSTANCE;
    instance.test();
}
~~~

### 6.5 建造者模式

> 利用既有的对象组装出新的对象

![K0bTqP.png](https://s2.ax1x.com/2019/10/26/K0bTqP.png)

~~~java
// 美食对象,包含食物和饮品两种属性
public class Meal {
    private String food;
    private String drink;

    public void setFood(String food){
        this.food = food;
    }
    public String getFood(){
        return food;
    }
    public void setDrink(String drink){
        this.drink = drink;
    }
    public String getDrink(){
        return drink;
    }

    public String toString(){
        return "food: " + this.food + ", drink: " + this.drink;
    }
}

// 抽象建造者
public abstract class AbstractBuilder {
    Meal meal = new Meal();

    public abstract void buildFood();

    public abstract void buildDrink();

    public Meal getMeal(){
        return meal;
    }
}

// 抽象建造者的具体实现
public class BuilderA extends AbstractBuilder {
    public void buildFood() {
        meal.setFood("油条");
    }

    public void buildDrink() {
        meal.setDrink("豆浆");
    }
}

public class BuilderB extends AbstractBuilder {
    public void buildFood() {
        meal.setFood("面包");
    }

    public void buildDrink() {
        meal.setDrink("牛奶");
    }
}

// 对外暴露提供服务的对象,封装了建造的过程,调用proviceMeal方法就能获取对应的对象
public class Director {
    private AbstractBuilder builder;

    public Director(AbstractBuilder builder) {
        this.builder = builder;
    }

    public Meal provideMeal() {
        builder.buildFood();
        builder.buildDrink();
        return builder.getMeal();
    }
}
~~~

### 6.6 原型模式

> 核心是**克隆**，即实现了`java.lang.Cloneable`接口，区分深克隆与浅克隆
>
> 基本数据类型可以通过浅克隆，深克隆主要针对复杂数据类型(`对象、数组`)等 
>
> 深克隆步骤：`序列化`:arrow_right:`转换二进制码`:arrow_right:`反序列化`:arrow_right:`赋值`

![K0bvxs.png](https://s2.ax1x.com/2019/10/26/K0bvxs.png)

~~~java
// 原型抽象
public abstract class Product implements Cloneable {
    public abstract void use(String s);
    public abstract Product cloneObj();
}

// 原型实现(克隆的对象)
public class ProductA extends Product {
    // 装饰字符(浅克隆)
    private char decochar;

    public ProductA(char decochar) {
        this.decochar = decochar;
    }

    public void use(String s) {
        int length = s.length();
        for (int i = 0; i < length + 4; i++) {
            System.out.print(decochar);
        }
        System.out.println("");
        System.out.println(decochar + " " + s + decochar);
        for (int i = 0; i < length + 4; i++) {
            System.out.print(decochar);
        }
    }

    public Product cloneObj() {
        Product p = null;
        try {
            p = (Product) clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return p;
    }
}

// 对象管理
public class Manager {
    private HashMap<String,Product> showcase = new HashMap<String, Product>();
	
    // 对象注册进容器
    public void register(String name, Product product){
        showcase.put(name,product);
    }

    // 从容器中取出对象并进行克隆,返回克隆的对象
    public Product create(String productName){
        Product product = showcase.get(productName);
        return product.cloneObj();
    }
}

// 对象深克隆 序列化 → 转换二进制码 → 反序列化 → 赋值
public class Student implements Serializable {
    // 自动生成序列化ID
    private static final long serialVersionUID = 296199231040740826L;
    private String name;

    public Student clone() {
        Student s = null;
        try {
            // 对象序列化成流
            ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
            ObjectOutputStream objectOutputStream = new ObjectOutputStream(byteArrayOutputStream);
            objectOutputStream.writeObject(this);

            // 将序列化的流转换成对象
            ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(byteArrayOutputStream.toByteArray());
            ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream);
            s = (Student) objectInputStream.readObject();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        return s;
    }
}
~~~

### 6.7 总结

> **`简单工厂`**模式最简单；
>
> **`工厂模式`**在简单工厂模式的基础上**增加了选择工厂的维度**，需要第一步选择合适的工厂
>
> **`抽象工厂模式`**有**产品族**的概念，如果各个产品是存在兼容性问题的，就要用抽象工厂模式
>
> **`单例模式`**为了保证全局使用的是同一对象，一方面是安全性考虑，一方面是为了节省资源
>
> **`建造者模式`**专门对付**属性很多的那种类**，为了让代码更优美
>
> **`原型模式`**了解 Object 类中的 clone() 方法相关的知识即可。

## 7种结构型模式

### 7.1 适配器模式

> 将不相关的类转换成需要的类

#### 7.1.1 默认适配器模式

> 避免无关接口方法的实现

![K0vPC8.png](https://s2.ax1x.com/2019/10/26/K0vPC8.png)

~~~java
// 定义接口
public interface DefaultAdapter {
    public void testOne();
    public void testTwo();
    public void testThree();
}

// 默认接口实现类
public class DefaultAdapterImpl implements DefaultAdapter {
    public void testOne() {}

    public void testTwo() {}

    public void testThree() {}
}

// 需要使用的类,仅重写部分方法
public class Target extends DefaultAdapterImpl {
    @Override
    public void testTwo() {
        System.out.println("我只需要重写testTwo方法");
    }
}
~~~

#### 7.1.2 对象适配器模式

> 旧的对象为内核，新的对象为外壳

![K0xERO.png](https://s2.ax1x.com/2019/10/26/K0xERO.png)

~~~java
// 客户端请求对象
public interface Target {
    public void say();
}

//  旧的对象
public class Dog {
    public void say(){
        System.out.println("汪汪汪");
    }
}

//  新的的对象
public class Cat implements Target {
    private Dog dog;
    public Cat(Dog dog){this.dog = dog;}
    public void say() {
        dog.say();
    }
}

// 客户端调用
public class Client {
    public static void main(String[] args) {
        Dog dog = new Dog();
        Target target = new Cat(dog);
        target.say();
    }
}
~~~

#### 7.1.3 类适配器模式

> 在旧类的基础上扩展出新的需要的类

![K0zlnJ.png](https://s2.ax1x.com/2019/10/26/K0zlnJ.png)

~~~java
//  定义接口
public interface AdapteeInter {
    public void eat();
}

//  现有的类
public class Adapteeobj {
    public void run(){
        System.out.println("我会跑");
    }

    public void sleep(){
        System.out.println("我会睡");
    }
}

//  需要的新的类
public class Adapter extends Adapteeobj implements AdapteeInter {
    public void eat() {
        System.out.println("我会吃");
    }
}

// 客户端调用
public class Client {
    public static void main(String[] args) {
        Adapter adapter = new Adapter();
        adapter.eat();
        adapter.run();
        adapter.sleep();
    }
}
~~~

### 7.2 桥接模式

> 将类与类之间继承的关系，变成`抽象类`或者`接口与接口`之间的关联关系，实现抽象化与实现化的解耦

![KXcDoV.md.png](https://s2.ax1x.com/2019/11/03/KXcDoV.md.png)

~~~java
// 定义颜色接口,充当桥梁
public interface Color {
    void paint(String penType, String name);
}

// 接口的子类实现
public class Blue implements Color {
    public void paint(String penType, String name) {
        System.out.println(penType + "画蓝色的" + name);
    }
}

public class Red implements Color {
    public void paint(String penType, String name) {
        System.out.println(penType + "画红色的" + name);
    }
}

// 定义抽象类,持有接口的引用
public abstract class AbstracePen {
    // 持有接口的引用
    protected Color color;
    protected void setColor(Color color){this.color = color;}
    public abstract void draw(String name);
}

// 抽象类实现,重写的方法底层是借助于桥梁接口的子类
public class BigPen extends AbstracePen {
    public void draw(String name) {
        String penType = "大号笔";
        this.color.paint(penType,name);
    }
}

public class SmallPen extends AbstracePen {
    public void draw(String name) {
        String penType = "小号笔";
        this.color.paint(penType,name);
    }
}

// 客户端调用
public class Client {
    public static void main(String[] args) {
        Red red = new Red();
        BigPen bigPen = new BigPen();

        bigPen.setColor(red);
        bigPen.draw("鲜花");
    }
}
~~~

### 7.3 组合模式

> 将一组相似的对象(具有层次结构的数据)当作一个单一的对象

~~~java
public class Emplayee {
    private String name;
    private String phoneNum;
    // 下属集合
    private List<Emplayee> emplayees;

    public Emplayee(String name, String phoneNum) {
        this.name = name;
        this.phoneNum = phoneNum;
        emplayees = new ArrayList<Emplayee>();
    }

    public void add(Emplayee emplayee){
        this.emplayees.add(emplayee);
    }

    public void remove(Emplayee emplayee){
        this.emplayees.remove(emplayee);
    }

    public List<Emplayee> getChildren(){
        return this.emplayees;
    }

    public String toString(){
        return "Emplayee:[name:"+this.name+", phoneNum:"+this.phoneNum+"]";
    }
}
~~~

### 7.4 装饰模式

> 动态地给一个对象添加或删除一个装饰的功能，用以**扩展功能**，相比继承更加的灵活，并且不改变现有代码
>
> 典型案例：java IO模块

![MGfFt1.png](https://s2.ax1x.com/2019/11/13/MGfFt1.png)

~~~java
// 定义接口
public interface Person {
    void eat();
}

// 接口实现
public class Man implements Person {
    public void eat() {
        System.out.println("男人在吃");
    }
}

// 抽象的装饰类,子类在此基础上进行扩展
public abstract class Decorator implements Person {
    protected Person person;
    public void setPerson(Person person) {
        this.person = person;
    }
    public void eat(){
        person.eat();
    }
}

// 子类添加喝汤功能
public class ManDecoratorA extends Decorator {
    public void eat(){
        super.eat();
        drink();
        System.out.println("ManDecoratroA类");
    }

    public void drink(){
        System.out.println("男人喝蛋花汤");
    }
}

// 子类去除喝汤功能
public class ManDecoratorB extends Decorator {
    public void eat(){
        super.eat();
        System.out.println("==========");
        System.out.println("ManDecoratorB类");
    }
}
~~~

### 7.5 外观模式

> 隐藏系统的复杂性，向客户端提供一个简单易用的访问接口

![MGTbKP.png](https://s2.ax1x.com/2019/11/13/MGTbKP.png)

~~~java
// 定义CPU类
public class CPU {
    public void start(){
        System.out.println("cpu启动");
    }

    public void shutDown(){
        System.out.println("cpu关闭");
    }
}

// 定义硬盘类
public class Disk {
    public void start(){
        System.out.println("disk启动");
    }

    public void shutDown(){
        System.out.println("disk关闭");
    }
}

// 定义内存类
public class Memory {
    public void start(){
        System.out.println("memory启动");
    }

    public void shutDown(){
        System.out.println("memory关闭");
    }
}

// 定义电脑类
public class Computer {
    private CPU cpu;
    private Disk disk;
    private Memory memory;
    public Computer(){
        cpu = new CPU();
        disk = new Disk();
        memory = new Memory();
    }

    public void start(){
        System.out.println("电脑开始启动");
        cpu.start();
        disk.start();
        memory.start();
        System.out.println("电脑启动完成");
    }

    public void shutDown(){
        System.out.println("电脑开始关机");
        cpu.shutDown();
        disk.shutDown();
        memory.shutDown();
        System.out.println("电脑关机完成");
    }
}

//  客户端调用
public class Client {
    public static void main(String[] args) {
        Computer computer = new Computer();
        computer.start();
        System.out.println("===电脑使用完毕===");
        computer.shutDown();
    }
}
~~~

### 7.6 享元模式

> 同一对象实例的重复使用

![MGOv5T.png](https://s2.ax1x.com/2019/11/13/MGOv5T.png)

~~~java
// 定义接口
public interface FlyWeight {
    void action();
}

// 接口子类实现
public class ChildFlyWeight implements FlyWeight {
    public void action() {
        System.out.println("享元模式");
    }
}

//  享元工厂，享元模式的核心
public class FlyWeightFactory {
    // 对象存放容器
    private static ConcurrentHashMap<String,FlyWeight> map = new ConcurrentHashMap<String, FlyWeight>();

    public static FlyWeight getFlyWeight(String name){
        if (map.get(name) == null) {
            synchronized (map) {
                if (map.get(name) == null) {
                    ChildFlyWeight childFlyWeight = new ChildFlyWeight();
                    map.put(name,childFlyWeight);
                }
            }
        }
        return map.get(name);
    }
}
~~~

### 7.7 代理模式

> 代理对象替代真实对象完成用户的请求

#### 7.7.1 静态代理

> 代理对象持有真实对象的引用，**代理类和真实类实现共同的接口**

![MJFk59.png](https://s2.ax1x.com/2019/11/13/MJFk59.png)

~~~java
// 定义接口
public interface Movie {
    void play();
}

// 接口实现
public class RealMovie implements Movie {
    public void play() {
        System.out.println("观看电影《当幸福来敲门》");
    }
}

// 代理类
public class ProxyMovie implements Movie {
    // 持有真实对象的引用
    RealMovie realMovie;
    public ProxyMovie(RealMovie realMovie) {
        this.realMovie = realMovie;
    }
    public void play() {
        slogan(true);
        realMovie.play();
        slogan(false);
    }
    private void slogan(boolean isStart){
        if (isStart) {
            System.out.println("电影即将开始,请关闭手机");
        } else {
            System.out.println("电影即将结束,请有序离场");
        }
    }
}

// 调用
public class Client {
    public static void main(String[] args) {
        RealMovie realMovie = new RealMovie();
        ProxyMovie proxyMovie = new ProxyMovie(realMovie);
        proxyMovie.play();
    }
}
~~~

#### 7.7.2动态代理

##### 7.7.2.1 JDK动态代理

> JDK动态代理的代理对象在创建时，需要使用业务实现类所实现的接口（即Movie）作为参数，没有接口则无法使用

![MJe1IS.png](https://s2.ax1x.com/2019/11/13/MJe1IS.png)

~~~java
// 定义接口
public interface Movie {
    void play();
}

// 接口实现
public class RealMovie implements Movie {
    public void play() {
        System.out.println("观看电影《当幸福来敲门》");
    }
}

// 代理类,要实现invocationHandler
public class ProxyMovie implements InvocationHandler {
    Movie movie;
    public ProxyMovie(Movie movie){
        this.movie = movie;
    }
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("电影即将开始,请坐好");
        method.invoke(movie,args);
        System.out.println("电影即将结束,请起立");
        return null;
    }
}

// 使用
public class Client {
    public static void main(String[] args) {
        RealMovie realMovie = new RealMovie();
        ProxyMovie proxyMovie = new ProxyMovie(realMovie);
        // 通过Proxy获取代理实例
        Movie proxyInstance = (Movie) Proxy.newProxyInstance(RealMovie.class.getClassLoader(), RealMovie.class.getInterfaces(), proxyMovie);
        proxyInstance.play();

    }
}
~~~

##### 7.7.2.2 CGLIB动态代理

> 针对类来实现代理，原理是**对指定的业务类生成一个子类，并覆盖其中业务方法**。因采取的是继承，所以不能对final修饰的类进行代理

~~~xml
<!-- 导入依赖 -->
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.3.0</version>
</dependency>
~~~

~~~java
// 定义类
public class RealMovie {
    public void play() {
        System.out.println("观看电影《当幸福来敲门》");
    }
}

// 定义代理类
public class ProxyMovie implements MethodInterceptor {
    Object object;

    public Object getInstance(Object object) {
        this.object = object;
        // 用来创建动态代理类
        Enhancer enhancer = new Enhancer();
        // 设置超类
        enhancer.setSuperclass(this.object.getClass());
        // 设置回调
        enhancer.setCallback(this);
        // 返回创建的代理类
        return enhancer.create();
    }

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("电影即将开始，请坐好");
        // 调用父类方法
        methodProxy.invokeSuper(o, objects);
        System.out.println("电影即将结束，请起立");
        return null;

    }
}
~~~

### 7.8 总结

> `代理模式`：方法的增强
>
> `装饰者模式`：类的增强
>
> `适配器模式`：基于旧类获取需要的新类
>
> `桥接模式`：抽象化与实现化的解耦
>
> `门面模式`：化繁为简，供客户使用
>
> `组合模式`：针对具有层次结构的数据
>
> `享元模式`：对象缓存，避免重复创建