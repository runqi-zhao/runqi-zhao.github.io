---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## Observer模式

在本章中，我们将要学习**Observer模式**。

Observer的意思是“进行观察的人”，也就是“观察者”的意思。

在Observer模式中，当观察对象的状态发生变化时，会通知给观察者。Observer模式适用于根据对象状态进行相应处理的场景。

## 例子

观察者将会观察一个会生成数值的对象，并将它生成的数值结果显示出来。不过，不同的观察者的显示方式不一样，DigitObserver会以数字形式显示数值，而GraphObserver则会以数字形式显示数值，而GraphObserver则会以简单的图示形式显示数值

| 名字                  | 说明                             |
| --------------------- | -------------------------------- |
| Observer              | 表示观察者的接口                 |
| NumberGenerator       | 表示生成数值的对象的抽象类       |
| RandomNumberGenerator | 生成随机数的类                   |
| DigitObserver         | 表示以数字形式显示数值的类       |
| GraphObserver         | 表示以简单的图示形式显示数值的类 |
| Main                  | 测试程序的类                     |

![image-20221021095228609](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221021095228609-1666317150119-1.png)

Observer接口时表示“观察值”的接口。具体的观察者会实现这个接口。

```java
public interface Observer {
    public abstract void update(NumberGenerator generator);
}
```

NumberGenerator类是用于生成数值的抽象类。

```java
public abstract class NumberGenerator {
    private ArrayList<Observer> observers = new ArrayList<>();

    public void addObserver(Observer observer) {
        observers.add(observer);
    }

    public void deleteObserver(Observer observer) {
        observers.remove(observer);
    }

    public void notifyObservers() {
        Iterator<Observer> iterator = observers.iterator();
        while (iterator.hasNext()) {
            Observer observer = (Observer) iterator.next();
            observer.update(this);
        }
    }

    public abstract int getNumber();
    public abstract void execute();
}
```

RandomNumberGenerator类是NumberGenerator的子类，他会生成随机数。

```java
public class RandomNumberGenerator extends NumberGenerator {
    private Random random = new Random();
    private int number;
    @Override
    public int getNumber() {
        return number;
    }

    @Override
    public void execute() {
        for (int i = 0; i < 20; i++) {
            number = random.nextInt(50);
            notifyObservers();
        }
    }
}
```

DigitObserver类实现了Observer接口，它的功能是以胡子形式显示观察到的数值。

```java
public class DigitObserver implements Observer {
    @Override
    public void update(NumberGenerator generator) {
        System.out.println("DigitObserver: " + generator.getNumber());
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

GraphObserver类也是先Observer接口。可以观察到的数值以```****```这样的简单图示的形式显示。

```java
public class GraphOberver implements Observer {
    @Override
    public void update(NumberGenerator generator) {
        System.out.println("GraphObserver");
        int count = generator.getNumber();
        for (int i = 0; i < count; i++) {
            System.out.print("*");
        }
        System.out.println("");
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

Main类进行测试，生成一个RandomNumberGenerator类的实例和两个观察者，其中observer1是DigitObserve类的实例，observer2是GraphObserver类的实例。

```java
public class Main {
    public static void main(String[] args) {
        RandomNumberGenerator generator = new RandomNumberGenerator();
        DigitObserver observer1 = new DigitObserver();
        GraphOberver oberver2 = new GraphOberver();
        generator.addObserver(observer1);
        generator.addObserver(oberver2);
        generator.execute();
    }
}
```

## 登场角色

### Subject（观察对象）

表示具体的被观察者对象

### ConcreteSubject（具体的观察对象）

负责接收来自Subject角色的状态变化的通知。

### Observer（观察者）

表示具体的Observer。当它的update方法被调用后，会去获取想要观察的对象的最新状态。

### ConcreteObserver（具体的观察者）

表示具体的Observer。当他的update方法被调用后，会去获取想要观察的对象的最新状态。

![image-20221021100611315](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221021100611315.png)

## 总结

### 这里也出现了可替换性

使用设计模式的目的之一就是为了使类成为可复用的组件。

在Observer模式中，有带状态的ConcreteSubject角色和接受状态变化通知的ConcreteObserver角色。连接这两个角色就是他们的接口（API）Subject角色和Observer角色。

一方面RandomNumberGenerator类并不知道，也无需在意正在观察自己的（自己需要通知的对象)到底是Digitobserver类的实例还是GraphObserver类的实例。不过它知道在它的observers字段中所保存的观察者们都实现了Observer接口。因为这些实例都是通过
addObserver方法注册的，这就确保了它们一定都实现了Observer接口，一定可以调用它们的update方法。

另一方面，Digitobserver类也无需在意自己正在观察的究竟是RandomNumberGenerator类的实例还是其他XXXXNumberGenerator类的实例。不过，Digitobserver类知道它们是NumberGenerator类的子类的实例，并持有getNumber方法。按照章节顺序阅读本书的读者一定注意到了在本书中已经多次出现了这种可替换性的设计思想。

- **利用抽象类和接口从具体类中抽出抽象对象**
- **再将实例作为参数传递值类中，或者在类的字段中保存实例时，不使用具体类型，而是使用抽象类型和接口**

这样的实现方式可以帮助我们轻松替换具体类。

### Observer的顺序

Subject角色中注册有多个Observer角色。在示例程序的notifyobservers方法中，先注册的Observer的update方法会先被调用。

通常，在设计ConcreteObserver角色的类时，需要注意这些Observer的update方法的调用顺序，不能因为update方法的调用顺序发生改变而产生问题。例如，在示例程序中，绝不能因为先调用Digitobserver的update方法后调用GraphObserver的update方法而导致应用程序不能正常工作。当然，通常，只要保持各个类的独立性，就不会发生上面这种类的依赖关系混乱的问题。

不过，我们还需要注意下面将要提到的情况。

### 当Observer的行为会对Subject产生影响时

例子中，RandomNumberGenerator类会在自身内部生成数值，调用update方法。不过，通常的Observer模式中，也可能是其他类出发Subject角色调用update方法。例如，在GUI应用程序中，多数情况下时用户按下announced后会触发updat方法被调用。

subject状态发生变化->通知Observer->Observer调用Subjecr的方法->导致Subject状态发生变化->通知Observer

### 传递更新信息的方式

NumberGenerator利用update方法告诉Observer自己的状态发生了更新。传递给update方法的参数只有一个，就是调用update方法的NumberGenerator的实例自身，Observer会在update方法中调用该实例的getNumeber来获取数据。

不过在示例程序中，update方法接收到的参数中并没有被更新的数值。也就是说，update方法的定义可能不是如下(1)中这样，而是如下(2)中这样，或者更简单的(3)这样的。

```
void update(NumberGenerator generator)						..(1)
void update(NumberGenerator generator,int number);			..(2)
void update(int number);									..(3)
```

(1)只传递了Subject角色作为参数。Observer角色可以从Subject角色中获取数据。
(2)除了传递Subject角色以外，还传递了Observer所需的数据（这里指的是所有的更新信息）。这样就省去了Observer自己获取数据的麻烦。不过，这样做的话，Subject角色就知道了Observer所要进行处理的内容了。

在很复杂的程序中，让Subject角色知道Observer角色所要进行的处理会让程序变得缺少灵活性。例如，假设现在我们需要传递上次传递的数值和当前的数值之间的差值，那么我们就必须在Subject角色中先计算出这个差值。因此，我们需要综合考虑程序的复杂度来设计update方法的参数的最优方案。

(3)比(2)简单，省略了Subject角色。示例程序同样也适用这种实现方式。不过，如果一个Observer角色需要观察多个Subject角色的时候，此方式就不适用了，这是因为Observer角色不知道传递给update方法的参数究竟是其中哪个Subject角色的数值。

### 从“观察”变为“通知”

Observer本意时“观察者”，但实际上Observer角色并非主动地观察，而是被动的接受来自Subject角色的通知，因此，Observer模式也成为Pubish-Subscribe(**<font color="red">发布-订阅</font>**)模式。

### Model/View/Controller(MVC)

MVC中的Model和View的关系与Subject角色和Observer角色的关系相对应，Model是指操作“不依赖于显示形式的内部模型，的部分，View
则是管理Model“怎样显示”的部分，通常情况下，一个Model对应多个View。

## 拓展-java.util.Observer接口

Java类库中的java.util.Observer接口和java,util.Observable类就是一种Observer模式。

java.util.Observer接口中定义了以下方法。

```java
public void update(Observable obj,Object arg)
```

而update方法的参数则接收到了如下内容。

- **Observable类的实例是被观察的Subject角色**
- **Object类的实例是附加信息**

这与上文中提到的类型(2)相似。

看到这里，大家可能会有这样的想法：原来Java已经为我们提供了Observer模式了啊，那我们直接用就可以了吧。

话虽如此，但是java.util.Observer接口和java,util.Observable类并不好用。理由很简单，传递给java.util.Observer接口的Subject角色必须是java.util.Observable类型（或者它的子类型）的。但Java只能单一继承，也就说如果Subject角色已经是某个类的子类了，那么它将无法继承java.util.Observable类。

## 相关设计模式

- [[16_Mediator模式]]
