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
> **`抽象工厂模式`**有产品族的概念，如果各个产品是存在兼容性问题的，就要用抽象工厂模式
>
> **`单例模式`**为了保证全局使用的是同一对象，一方面是安全性考虑，一方面是为了节省资源
>
> **`建造者模式`**专门对付属性很多的那种类，为了让代码更优美
>
> **`原型模式`**了解 Object 类中的 clone() 方法相关的知识即可。