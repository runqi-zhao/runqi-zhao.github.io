---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## Menento模式

我们在使用文本编辑器编写文件时，如果不小心删除了某句话，可以通过撤销(undo)功能将文件恢复至之前的状态。有些文本编辑器甚至支持多次撤销，能够恢复至很久之前的版本。

使用面向对象编程的方式实现撤销功能时，需要事先保存实例的相关状态信息。然后，在撤销时，还需要根据所保存的信息将实例恢复至原来的状态。

要想恢复实例，需要一个可以自由访问实例内部结构的权限。但是，如果稍不注意，又可能会将依赖于实例内部结构的代码分散地编写在程序中的各个地方，导致程序变得难以维护。这种情况就叫作“**破坏了封装性**”。

通过引入表示实例状态的角色，可以在保存和恢复实例时有效地防止对象的封装性遭到破坏。这就是我们在本章中要学习的Memento模式。使用Memento模式可以实现应用程序的以下功能。

- Undo（撤销）
- Redo（重做）
- History（历史记录）
- Snapshot（快照）

## 例子

这是一个收集水果和获取金钱树的投骰子游戏，游戏规则如下：

- 游戏是自动进行的
- 游戏的主人公通过掷骰子来决定下一步的状态
- 当骰子的点数是1的时候，主人公的金钱会增加
- 当骰子的点数是2的时候，主人公的金钱会减少
- 当投资的点数是6的时候，主人公会得到水果
- 主人公没有钱的时候，游戏就会结束

| 包   | 名字    | 说明                                                         |
| ---- | ------- | ------------------------------------------------------------ |
| game | Memento | 表示Gamer状态的类                                            |
| game | Gamer   | 表示主人公的类。它会生成Memento的实例                        |
| 无名 | Main    | 进行游戏的类。他会是先保存Memento的实例。之后根据需要恢复的Gamer的状态 |

![image-20221022104825876](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221022104825876-1666406907077-1.png)

Memento类时表示Gamer（主人公）状态的类。

Memento类和Gamer类都位于game包下。

Memento类中有两个字段，即money和fruits。money表示主人公现在所持有的金钱数目，fruits表示现在为止所获得的水果。之所以没有将money和fruits的可见性设为private,是因为我们希望同在game包下的Gamer类可以访问这两个字段。

getMoney方法的作用是获取主人公当前所持有的金钱数目。

Memento类的构造函数的可见性并非public,因此并不是任何其他类都可以生成Memento类的实例。只有在同一个包（本例中是game包）下的其他类才能调用Memento类的构造函数。具体来说，只有game包下的Gamer类才能生成Memento类的实例。

addFruit方法用于添加所获得的水果。该方法的可见性也不是public。这是因为只有同一个包下的其他类才能添加水果。因此，**无法从game包外部改变Memento内部的状态。**

此外，Memento类中有<font color="red">“narrow interface”和“wide interface'</font>”这样的注释。详情请看总结。

```java
public class Memento {
    int money;
    ArrayList<String> fruits;

    public int getMoney() {
        return money;
    }

    Memento(int money) {
        this.money = money;
        this.fruits = new ArrayList<>();
    }

    void addFruit(String fruit) {
        fruits.add(fruit);
    }

    List getFruits() {
        return (List) fruits.clone();
    }
}
```

Gamer类是表示游戏主人公的类。拥有3个字段，即为所持金钱（money）、获得的水果（fruit）以及一个随机数生成器（random）。而且还有一个名为fruitsname的静态字段。

进行游戏的主要方法是bet方法。在该方法中，只要主人公没有破产，就会一直掷骰子，并根据骰子结果改变所持有的金钱数目和水果个数。

createMemento方法的作用是保存当前的状态（拍摄快照）。在createMemento方法中，会根据当前在时间点所持有的金钱和水果生成一个Memento类的实例，该实例代表了“当前Gamer的状态”，它会被返回给调用者。就如同给对象照了张照片一样，我们将对象现在的状态封存在Memento类的实例中。请注意我们只保存了“好吃”的水果。

restoreMemento方法的功能与createMemento相反，它会根据接收到的Memento类的实例来将Gamer恢复为以前的状态，仿佛是在游戏中念了一通“复活咒语”一样。

```java
public class Gamer {
    private int money;
    private List<String> fruits = new ArrayList<>();
    private Random random = new Random();
    private static String[] fruitsname = {
            "苹果", "葡萄", "香蕉", "橘子"
    };

    public Gamer(int money) {
        this.money = money;
    }

    public int getMoney() {
        return money;
    }

    public void bet() {
        int dice = random.nextInt(6) + 1;
        if (dice == 1) {
            money += 100;
            System.out.println("所持金钱增加了");
        } else if (dice == 2) {
            money /= 2;
            System.out.println("所持金钱减半了");
        } else if (dice == 6) {
            String f = getFruit();
            System.out.println("获得了水果(" + f + ")。");
            fruits.add(f);
        } else {
            System.out.println("什么也没发生");
        }
    }

    public Memento createMemento() {
        Memento m = new Memento(money);
        Iterator<String> it = fruits.iterator();
        while (it.hasNext()) {
            String f = it.next();
            if (f.startsWith("好吃的")) {
                m.addFruit(f);
            }
        }
        return m;
    }

    public void restoreMemento(Memento memento) {
        this.money = memento.money;
        this.fruits = memento.fruits;
    }

    @Override
    public String toString() {
        return "[" +
                "money=" + money +
                ", fruits=" + fruits + "]";
    }

    private String getFruit() {
        String prefix = "";
        if (random.nextBoolean()) {
            prefix = "好吃的";
        }
        return prefix + fruitsname[random.nextInt(fruitsname.length)];
    }
}
```

Main方法进行测试，如果运气好，会调用保存当前状态的方法，如果运气不好，则会调用返回方法。

```java
public class Main {
    public static void main(String[] args) {
        Gamer gamer = new Gamer(100);
        Memento memento = gamer.createMemento();
        for (int i = 0; i < 100; i++) {
            System.out.println("====" + i);
            System.out.println("当前状态：" + gamer);
            gamer.bet();
            System.out.println("所持金钱为" + gamer.getMoney() + "元");
            if (gamer.getMoney() > memento.getMoney()) {
                System.out.println("（所持金钱增加了许多，因此保存游戏当前的状态）");
                memento = gamer.createMemento();
            } else if (gamer.getMoney() < memento.getMoney() / 2) {
                System.out.println("（所持金钱减少了许多，因此将游戏恢复至以前的状态）");
                gamer.restoreMemento(memento);
            }

            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            System.out.println("");
        }
    }
}
```

示例图如下：

![image-20221022105705071](./assets/image-20221022105705071-1666407425958-3.png)

## 登场角色

### Originator（生成者）

Originator角色会在保存自己的最新状态的时生成Memento角色。当把以前保存的Memento角色传递给Originator角色时，他会将自己恢复至生成该Memento角色的状态。

### Memento（纪念品）

Memento角色会将Originator角色的内部信息整合在一起。在Memento角色中虽然保存了Originator角色的信息名单时不会向外部公开这些信息。

Menento角色有以下两种接口

#### wide interface-宽接口（API）

Memento角色提供的宽接口（API）实质所有用于获取恢复对象信息的方法的集合。由于宽接口会暴漏所有Memento角色的内部信息，因此能够使用宽接口（API）的止呕Originator角色

### narrow interface-窄接口（API）

Memento角色为外部的Caretaker角色提供了“窄接口(API)”。可以通过窄接口(API)获取的Memento角色的内部信息非常有限，因此可以有效地防止信息泄露。

通过对外提供以上两种接口(API),可以有效地防止对象的封装性被破坏。

在示例程序中，由Memento类扮演此角色。

Originator角色和Memento角色之间有着非常紧密的联系。

### Caretaker（负责人）

当Caretaker角色要保存当前的Originator角色的状态时，会通知Originator角色。Originator角色中在接收到通知后会生成Memento角色的实例并将其返回给Caretaker角色。由于以后可能会用Memento实例来讲Originator恢复至原来的状态，因此Caretaker角色洄游至保存Memento实例。

不过，在Caretaker角色只是使用Memento角色两种接口（API）中窄接口，也就是说，它无法访问Memento内部的所有信息。**它只是将Originator角色生成Memento角色当作一个黑盒保存了起来。**

虽然Originator角色和Memento角色之间是强关联关系，但Caretaker角色和Memento角色之间是弱关联关系。Memento角色对Caretaker角色隐藏了自身的内部信息。

![image-20221022111704052](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221022111704052-1666408624943-5.png)

## 总结

### 两种API的可见性

为了能够实现Memento模式中的两套接口(API),我们利用了Java语言中的可见性。表展示了Java语言提供的4种可见性。

| 可见性    | 说明                               |
| --------- | ---------------------------------- |
| public    | 所有类都可以访问                   |
| protected | 同一包中的类或是该类的子类可以访问 |
| 无        | 同一包中的类可以访问               |
| private   | 只有该类自身可以访问               |

在Memento类的方法和字段中，有带public修饰符的，也有不带修饰符的。这表示设计者希望能够进行控制，从而使某些类可以访问这些方法和字段，而其他一些类则无法访问。

| 可见性 | 字段 方法 构造函数 | 哪个类可以访问             |
| ------ | ------------------ | -------------------------- |
| 无     | money              | Memento类、Gamer类         |
| 无     | fruits             | Memento类、Gamer类         |
| public | getMoney           | Memento类、Gamer类、Main类 |
| 无     | Memento            | Memento类、Gamer类         |
| 无     | addFruit           | Memento类、Gamer类         |

在Memento类中，只有getMoney方法是public的，它是一个窄接曰(API),因此该方法也可以被扮演Caretaker角色的Main类调用。

这里做一下补充说明。明明该方法带有修饰符pub1ic,但它却一个是窄接口(API),这不免让人感到有些奇怪。其实，这里所说的“窄”是指外部可以操作的类内部的内容很少。在Memento类的所有方法中，只有getMoney的可见性是public的。也就是说，扮演Caretaker角色的Main类可以获取的只有当前状态下的金钱数目而已。像这种“能够获取的信息非常少”的状态就是本章中“窄”的意思。

由于扮演Caretaker角色的Main类并不在game包下，所以它只能调用public的getMoney方法。因此，Main类无法随意改变Memento类的状态。

还有一点需要注意的是，在Main类中Memento类的构造函数是无法访问的，这就意味着无法像下面这样生成Memento类的实例。

```java
new Memento(100)
```

如果像这样编写了代码，在编译代码时编译器就会报错。如果Main类中需要用到Memento类的实例，可以通过调用Gamer类的createMemento方法告诉Gamer类“我需要保存现在的状态，请生成一个Memento类的实例给我”。

如果我们在编程时需要实现“允许有些类访问这个方法，其他类则不能访问这个方法”这种需求，可以像上面这样使用可见性来控制访问权限。

### 需要多少个Memento

在示例程序中，Main类只保存了一个Memento。如果在Main类中使用数组等集合，让它可以保存多个Memento类的实例，就可以实现保存各个时间，点的对象的状态。

### Memento的有效期限是多久

在示例程序中，我们是在内存中保存Memento的，这样并没有什么问题。但是如果要将Memento永远保存在文件中，就会出现有效期限的问题了。

这是因为，假设我们在某个时间点将Memento保存在文件中，之后又升级了应用程序版本那么可能会出现原来保存的Memento与当前的应用程序不匹配的情况。

### 划分Caretaker角色和Originator角色的意义

Caretaker角色的职责是决定合适拍摄快照，何时撤销以及保存Memento角色。

另一方面，Originator角色的职责是生成Memento角色和使用接收到的Memento角色来恢复自己的状态。

以上就是Caretaker角色与Originator角色的职责分担。有了这样的**职责分担**，我们需要对应以下的需求变更时，就可以完全不用修改Originator角色。

- **变更为可以多次撤销**
- **变更为不仅可以撤销，还可以将现在的状态保存在文件中**

Caretaker角色只能通过窄接口（API)来操作Memento角色。如果Caretaker角色可以随意地操作Memento角色，会导致Caretaker角色失去与Originator角色和Memento角色之间的独立性。如果Caretaker角色可以随意地操作Memento角色，当要修改Originator角色内部的代码时，就必须要同样地修改Caretaker角色。

而如果Caretaker角色只能使用窄接口（API),只要不修改那个接口(API),就可以自由地修改Originator角色和Memento角色。

