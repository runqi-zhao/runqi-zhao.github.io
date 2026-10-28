---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

<font color="red">将多个对象组成一条责任链，然后按照他们在职责链上的顺序一个个地找出到底应该谁来负责处理。</font>这种模式称为Chain of Responsibility模式（推卸责任模式）。

使用Chain of Responsibility模式可以弱化“请求方”和“处理方“之间的关联关系，让双方各自都成为可独立复用的组件。

## 例子

注意谁必须负责处理所发生地问题。

| 名字           | 说明                                               |
| -------------- | -------------------------------------------------- |
| Trouble        | 表示发生的问题的类                                 |
| Support        | 用来解决问题的抽象类                               |
| NoSupport      | 用来解决问题的具体类(永远”不处理问题“)             |
| LimitSupport   | 用来解决问题的具体类(进解决编号小于指定编号的问题) |
| OddSupport     | 用来解决问题的具体类(仅解决奇数编号的问题)         |
| SpecialSupport | 用来解决问题的具体类(永远”不处理问题“)             |
| Main           | 用来解决问题的具体类(永远”不处理问题“)             |

![image-20221018163848513](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221018163848513-1666082330192-1.png)

Trouble类时表示发生的问题类。number是问题编号。

```java
public class Trouble {
    private int number;
    public Trouble(int number) {
        this.number = number;
    }
    public int getNumber() {
        return number;
    }
    public String toString() {
        return "[Trouble " + number + "]";
    }
}
```

Support类是用来解决问题的抽象类，它是职责链上的对象。

```java
public abstract class Support {
    private String name;
    private Support next;
    public Support(String name) {
        this.name = name;
    }

    public Support setNext(Support next ) {
        this.next = next;
        return next;
    }

    public final void support(Trouble trouble) {
        if (resolve(trouble)) {
            done(trouble);
        } else if (next != null) {
            next.support(trouble);
        } else {
            fail(trouble);
        }
    }

    public String toString() {
        return "[" + name + "]";
    }

    //support方法调用了抽象方法resolve，因此它属于Template Method模式
    protected abstract boolean resolve(Trouble trouble);
    protected void done(Trouble trouble) {
        System.out.println(trouble + " is resolved by " + this + ".");
    }
    protected void fail(Trouble trouble) {
        System.out.println(trouble + " cannot be resolved.");
    }
}
```

接下来是NoSupport类，是Support子类，表示永远”不解决问题“的类

```java
public class NoSupport extends Support {
    public NoSupport(String name) {
        super(name);
    }

    @Override
    protected boolean resolve(Trouble trouble) {
        return false;
    }
}
```

LimitSupport类解决编号小于limit值得问题。

```java
public class LimitSupport extends Support{
    private int limit;
    public LimitSupport(String name, int limit) {
        super(name);
        this.limit = limit;
    }

    @Override
    protected boolean resolve(Trouble trouble) {
        if (trouble.getNumber() < limit) {
            return true;
        } else {
            return false;
        }
    }
}
```

OddSupport类解决奇数编号得问题

```java
public class OddSupport extends Support{
    public OddSupport(String name) {
        super(name);
    }

    @Override
    protected boolean resolve(Trouble trouble) {
        if (trouble.getNumber() % 2 == 1) {
            return true;
        } else {
            return false;
        }
    }
}
```

SpecialSupport类之解决指定编号的问题

```java
public class SpecialSupport extends Support {
    private int number;
    public SpecialSupport(String name, int number) {
        super(name);
        this.number = number;
    }

    @Override
    protected boolean resolve(Trouble trouble) {
        if (trouble.getNumber() == number) {
            return true;
        } else {
            return false;
        }
    }
}
```

Main用于测试

```java
public class Main {
    public static void main(String[] args) {
        NoSupport alice = new NoSupport("Alice");
        LimitSupport bob = new LimitSupport("Bob", 100);
        SpecialSupport charlie = new SpecialSupport("Charlie", 429);
        LimitSupport diana = new LimitSupport("Diana", 200);
        OddSupport elmo = new OddSupport("Elmo");
        LimitSupport fred = new LimitSupport("Fred", 300);
        alice.setNext(bob).setNext(charlie).setNext(diana).setNext(elmo).setNext(fred);
        for (int i = 0; i < 500; i += 33) {
            alice.support(new Trouble(i));
        }
    }
}
```

运行结果如下图：

![image-20221018165053161](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221018165053161-1666083054174-5.png)

让我们看看最终运行结果。最开始Bob非常努力地解决了几个问题，当他无法解决的时候会将问题交给Diana负责。在运行结果中，完全没有出现Alice的身影，这是因为Alice会把所有的问题推给别人。当问题编号超过300后，不论是哪个LimitSupport类的实例都无法解决了。不过，只要编号为奇数，OddSupport类的实例Elmo就可以帮我们解决问题。而SpecialSupport类的实例Charlie只负责解决编号为429的问题，因此在运行结果中它只出现了一次。
下图展示了解决编号为363号问题时的时序图。在该时序图中，我们重点关注了support方法的调用情况。实际上，每个Support在调用下一个Support的support方法之前，都会先调用自身的resolve方法。

![image-20221018165027509](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221018165027509-1666083028381-3.png)

## 登场角色

### Handler(处理者)

定义了处理请求的接口(API)。

### ConcreteHandler(具体的处理者)

处理请求的具体角色。

### Client(请求者)

Client角色是向第一个ConcreteHandler角色发送请求的角色。

![image-20221018165629389](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221018165629389-1666083390912-7.png)

## 总结

### 弱化了发出请求的人和处理请求的人之间的关系

最大的有点就在于它弱化了发出请求的人(Client角色)和处理请求的人(ConcreteHandler角色)之间的关系。Client角色向第一个ConcreteHandler角色发出请求，然后请求会在职责链中传播，知道某个ConcreteHandler角色处理该请求。

如果不使用该模式，就必须有某个伟大的角色知道“谁应该处理什么请求”，这有点类似中央集权制。而让“发出请求的人”知道“谁应该处理该请求”并不明智，因为如果发出请求的人不得不知道处理请求的人各自的责任分担情况，就会降低其作为可复用的组件的独立性。

> 为了简单起见，在示例程序中，我们扮演Client角色的Main类负责串联起ConcreteHandler的职责链

### 可以动态的改变职责链

在示例程序中，问题的解决是按照从Alice到Fred的固定顺序进行处理的。但是，我们还需要考虑负责处理的各个ConcreteHandler角色之间的关系可能会发生变化的情况。如果使用Chain ofResponsibility模式，通过委托推卸责任，就可以根据情况变化动态地重组职责链。

如果不使用Chain of Responsibility模式，而是在程序中固定写明“某个请求需要谁处理”这样的对应关系，那么很难在程序运行中去改变请求的处理者。

在视窗系统中，用户有时需要可以自由地在视窗中添加控件（按钮和文本输入框等）。这时，Chain of Responsibility模式就有了用武之地。

### 专注于自己的工作

“推卸”这个词虽然有贬义，但是反过来想，这样才可以使每个对象更专注于自己的工作，即每个ConcreteHandler角色都专注于自己所负责的处理。当自己无法处理时，ConcreteHandler角色就会干脆地对下一个处理者说一句“嘿，交给你了”，然后将请求转出去。这样，每个
ConcreteHandler角色就能只处理它应该负责的请求了。

如果我们不使用Chain of Responsibility模式又会怎样呢？这时，我们需要编写一个“决定谁应该负责什么样的处理”的方法。亦或是让每个ConcreteHandler角色自己负责“任务分配”工作，即“如果自己不能处理，就转交给那个人。如果他也不能处理，那就根据系统情况将请求再转交给另外一个人”。

### 推卸请求会导致处理延迟吗

使用Chain of Responsibility模式可以推卸请求，直至找到合适的处理请求的对象，这样确实提高了程序的灵活性，但是难道不会导致处理延迟吗？

确实如此，与“事先确定哪个对象负责什么样的处理，当接收到请求时，立即让相应的对象去处理请求”相比，使用Chain of Responsibility模式确实导致处理请求发生了延迟。不过，这是一个需要权衡的问题。如果请求和处理者之间的关系是确定的，而且需要非常快的处理速度时，不使用Chain of Responsibility模式会更好。

### 相关设计模式

- [[11_Composite模式]]
- [[Command模式]]
