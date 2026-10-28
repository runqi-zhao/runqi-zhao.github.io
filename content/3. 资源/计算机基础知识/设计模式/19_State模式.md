---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## State模式

在面向对象编程中，是用类表示对象的。也就是说，程序的设计者需要考虑用类来表示什么东西。类对应的东西可能存在于真实世界中，也可能不存在于真实世界中。对于后者，可能有人看到代码后会感到吃惊：这些东西居然也可以是类啊。

在本章中，我们将要学习了State模式。

在State模式中，我们用类来表示状态。State的意思就是“状态”。在现实世界中，我们会考虑各种东西的“状态”，但是几乎不会将状态当作“东西”看待。因此，可能大家很难理解“用类来表示状态”的意思。

在本章中，我们将要学习用类来表示状态的方法。以类来表示状态后，我们就能通过切换类来方便地改变对象的状态。当需要增加新的状态时，如何修改代码这个问题也会很明确。

## 例子

金库报警系统。

戒备状态每个小时改变一次报警系统。

- 有一个金库
- 金库与警报中心相连
- 金库里有警铃和正常通话用的电话
- 金库里有时钟，监视着现在的时间

----

- 白天的时间范围是`9：00~16：59`，晚上的时间范围是`17：00~23：59`和`0：00~8：59`
- 金库只能在白天使用
- 白天使用金库的话，会在警报中心留下记录
- 晚上使用金库的话，会向警报中心发送紧急事态通知

----

- 任何时候都可以使用警铃
- 使用警铃的话，会向警报中心发送紧急事态通知

----

- 任何时候都可以使用电话（但晚上只有留言电话）
- 白天使用电话的话，会呼叫警报中心
- 晚上用电话的话，会呼叫警报中心的留言电话

![image-20221023110024363](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221023110024363-1666494025873-1.png)

### 不使用state模式的代码

在学习使用了State模式的示例程序前，请大家先思考一下自己会以什么样的编程方式来实现这个系统。

如果是我的话，在读完了前面的内容后，会像下面这样考虑。

“总的说来，就是每小时改变系统的行为。当发生使用金库、按下警铃和正常通话这3个事件的时候，会有某种通知到达警报中心。然后，通知的内容会根据时间发生变化”...

### 使用State模式的伪代码

并不能说上面的想法是错的。不过，在本章中我们要学习的State模式则是从完全不同的角度出发，实用类进行表示白天与晚上更加合适。而不是使用if来进行判断。

因此，我们再进行设计时，分成以下几个类：

| 名字       | 说明                                                  |
| ---------- | ----------------------------------------------------- |
| State      | 表示金库状态的接口                                    |
| DayState   | 表示“白天”状态的类。它实现了State接口                 |
| NightState | 表示“晚上”状态的类。它实现了State接口                 |
| Context    | 表示管理金库状态，并于警报中心联系的接口              |
| SafeFrame  | 实现了Context接口。在塔内布持有按钮和画面显示等UI信息 |
| Main       | 进行测试                                              |

![image-20221023110708845](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221023110708845-1666494429840-3.png)

State接口是表示金库状态的接口。在State接口定义了以下实践对应的接口(API)。

- 设置时间
- 使用金库
- 按下警铃
- 正常通话

```java
public interface State {
    public abstract void doClock(Context context, int hour);
    public abstract void doUse(Context context);
    public abstract void doAlarm(Context context);
    public abstract void doPhone(Context context);
}
```

DayState表示白天的状态。

doClock是用于设置时间的方法。如果接收到的参数表示晚上的时间，就会切换到夜间状态，即发生状态变化（状态迁移）。在该类中，我们调用Context接口的changeState方法改变状态。

```java
public class DayState implements State {
    private static DayState singleton = new DayState();
    private DayState() {

    }

    public static State getInstance() {
        return singleton;
    }

    @Override
    public void doClock(Context context, int hour) {
        if (hour < 9 || 17 <=hour) {
            context.changeState(NightState.getInstance());
        }
    }

    @Override
    public void doUse(Context context) {
        context.recordLog("使用金库（白天）");
    }

    @Override
    public void doAlarm(Context context) {
        context.callSecurityCenter("按下警铃（白天）");
    }

    @Override
    public void doPhone(Context context) {
        context.callSecurityCenter("正常通话（白天）");
    }

    @Override
    public String toString() {
        return "[白天]";
    }
}
```

NightState表示晚上的状态

```java
public class NightState implements State {
    private static NightState singleton = new NightState();
    private NightState() {

    }

    public static State getInstance() {
        return singleton;
    }

    @Override
    public void doClock(Context context, int hour) {
        if (9 <= hour && hour < 17) {
            context.changeState(DayState.getInstance());
        }
    }

    @Override
    public void doUse(Context context) {
        context.recordLog("紧急：晚上使用金库！");
    }

    @Override
    public void doAlarm(Context context) {
        context.callSecurityCenter("按下警铃（晚上）");
    }

    @Override
    public void doPhone(Context context) {
        context.recordLog("晚上打的电话录音");
    }

    @Override
    public String toString() {
        return "[晚上]";
    }
}
```

Context接口负责管理状态和联系警报中心的接口。

```java
public interface Context {
    public abstract void setClock(int hour);
    public abstract void changeState(State state);
    public abstract void callSecurityCenter(String msg);
    public abstract void recordLog(String msg);
}
```

SafeFrame类时使用GUI实现警报系统界面的类

给代表状态的字段赋予表示当前状态的类的实例，就相当于进行了状态迁移。

```java
public class SafeFrame extends Frame implements ActionListener, Context {
    private TextField textClock =  new TextField(60);
    private TextArea textScreen =  new TextArea(10, 60);
    private Button buttonUse = new Button("使用金库");
    private Button buttonAlarm = new Button("按下警铃");
    private Button buttonPhone = new Button("正常通话");
    private Button buttonExit = new Button("结束");

    private State state = DayState.getInstance();

    public SafeFrame(String title) {
        super(title);
        setBackground(Color.lightGray);
        setLayout(new BorderLayout());
        add(textClock, BorderLayout.NORTH);
        textClock.setEditable(false);
        add(textScreen, BorderLayout.CENTER);
        textScreen.setEditable(false);
        Panel panel = new Panel();
        panel.add(buttonUse);
        panel.add(buttonAlarm);
        panel.add(buttonPhone);
        panel.add(buttonExit);
        add(panel, BorderLayout.SOUTH);
        pack();
        show();
        buttonUse.addActionListener(this);
        buttonAlarm.addActionListener(this);
        buttonPhone.addActionListener(this);
        buttonExit.addActionListener(this);
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println(e.toString());
        if (e.getSource() == buttonUse) {
            state.doUse(this);
        } else if (e.getSource() == buttonAlarm) {
            state.doAlarm(this);
        } else if (e.getSource() == buttonPhone) {
            state.doPhone(this);
        } else if (e.getSource() == buttonExit) {
            System.exit(0);
        } else {
            System.out.println("?");
        }
    }

    @Override
    public void setClock(int hour) {
        String clockString = "现在时间是";
        if (hour < 10) {
            clockString += "0" + hour + ":00";
        } else {
            clockString += hour + ":00";
        }
        System.out.println(clockString);
        textClock.setText(clockString);
        state.doClock(this, hour);
    }

    @Override
    public void changeState(State state) {
        System.out.println(this.state + " -> " + state);
        this.state = state;
    }

    @Override
    public void callSecurityCenter(String msg) {
        textScreen.append("call! " + msg + "\n");
    }

    @Override
    public void recordLog(String msg) {
        textScreen.append("record..." + msg + "\n");
    }
}
```

![image-20221023111456929](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221023111456929-1666494897929-5.png)

Main类生成一个进行测试

```java
public class Main {
    public static void main(String[] args) {
        SafeFrame frame = new SafeFrame("State Sample");
        while (true) {
            for (int hour = 0; hour < 24; hour++) {
                frame.setClock(hour);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                }
            }
        }
    }
}
```

## 登场角色

### State（状态）

表示状态，定义了根据不同状态进行不同处理的接口。该接口是那些处理内容依赖于装他的方法的集合。

### ConcreteState（具体状态）

表示各个具体的状态，实现了State接口。

### Context（状态、前后关系、上下文）

Context角色持有表示当前状态的ConcreteState角色。此外，它还定义了供外部调用者使用State模式的接口(API)。在示例程序中，由Context接口和SafeFrame类扮演此角色。

这里稍微做一下补充说明。在示例程序中，Context角色的作用被Context接口和SafeFrame类分担了。具体而言，Context接口定义了供外部调用者使用State模式的接口(API),而SafeFrame类则持有表示当前状态的ConcreteState角色。

为什么没有将Context实现为类？

答：因为在Java中只能单一继承，所以如果将Context角色定义为类，那么由于SafeFrame类已经是Frame类的子类了，它将无法再继承Context类。

不过，如果另外编写一个Context类的子类，并将它的实例保存在SafeFrame类的字段中，那么通过将处理委托给这个实例是可以实现习题中的需求的。

![image-20221023111821311](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221023111821311-1666495102419-7.png)

## 总结

### 分而治之

在编程时，我们经常会使用分而治之的方针。它非常适用于大规模的复杂处理。当遇到庞大且复杂的问题，不能用一般的方法解决时，我们会先将该问题分解为多个小问题。如果还是不能解决这些小问题，我们会将它们继续划分为更小的问题，直至可以解决它们为止。分而治之，简单而言就是将一个复杂的大问题分解为多个小问题然后逐个解决。

在State模式中，我们用类来表示状态，并为每一种具体的状态都定义一个相应的类。这样，问题就被分解了。开发人员可以在编写一个ConcreteState角色的代码的同时，在头脑中（一定程度上)考虑其他的类。在本章的金库警报系统的示例程序中，只有“白天”和“晚上”两个状态，可能大家对此感受不深，但是当状态非常多的时候，State模式的优势就会非常明显了。

在不使用State模式时，我们需要使用条件分支语句判断当前的状态，然后进行相应的处理。状态越多，条件分支就会越多。而且，我们必须在所有的事件处理方法中都编写这些条件分支语句。State模式用类表示系统的“状态”，并以此将复杂的程序分解开来。

### 依赖于状态的处理

我们来思考一下SafeFrame类的setclock方法（代码清单19-7）和State接口的doclock方法（代码清单19-3）之间的关系。

Main类会调用SafeFrame类的setClock方法，告诉setClock方法“请设置时间”。在setClock方法中，会像下面这样将处理委托给state类。

```java
state.doclock(this,hour);
```

也就是说，我将设置时间的处理看作是“依赖于状态的处理”当然，不只是doc1ock方法。在State接口中声明的所有方法都是“依赖于状态的处理”，都是“状态不同处理也不同”。这虽然看似理所当然，不过却需要我们特别注意。

在State模式中，我们应该如何编程，以实现“依赖于状态的处理”呢？总结起来有如下两点。

- **定义接口，声明抽象方法**
- **定义多个类，实现具体方法**

### 应当是谁来管理状态迁移

用类来表示状态，将依赖于状态的处理分散在每个ConcreteState角色中，这是一种非常好的解决办法。

不过，在使用State模式时需要注意应当是谁来管理状态迁移

在示例程序中，扮演Context角色的SafeFrame类实现了实际进行状态迁移的changeState方法。但是，实际调用该方法的却是扮演ConcreteState角色的DayState类和NightState类。也就是说，在示例程序中，我们将“状态迁移”看作是“依赖于状态的处理”。这种处理方式既有优点也有缺点。

优点是这种处理方式将“什么时候从一个状态迁移到其他状态“的信息集中在了一个类中。也就是说，当我们想知道“什么时候会从DayState类变化为其他状态”时，只需要阅读DayState类的代码就可以了。

缺点是“每个ConcreteState角色都需要知道其他ConcreteState角色”。例如，DayState类的doClock方法就使用了NightState类。这样，如果以后发生需求变更，需要删除NightState类时，就必须要相应地修改DayState类的代码。将状态迁移交给ConcreteState角色后，每个ConcreteState角色都需要或多或少地知道其他ConcreteState角色。也就是说，将状态迁移交给ConcreteState角色后，各个类之间的依赖关系就会加强。

我们也可以不使用示例程序中的做法，而是将所有的状态迁移交给扮演Context角色的SafeFrame类来负责。有时，使用这种解决方法可以提高ConcreteState角色的独立性，程序的整体结构也会更加清晰。不过这样做的话，Context角色就必须要知道“所有的ConcreteState角色”。在这种情况下，我们可以使用Mediator模式。

当然，还可以不用State模式，而是用状态迁移表来设计程序。所谓状态迁移表是可以根据“输入和内部状态”得到“输出和下一个状态”的一览表（这超出了本书的范围，我们暂且不深人学习该方法)。当状态迁移遵循一定的规则时，使用状态迁移表非常有效。此外，当状态数过多时，可以用程序来生成代码而不是手写代码。

### 不自相矛盾

如果不使用State模式，我们需要使用多个变量的值的集合来表示系统的状态。这时，必须十分小心，注意不要让变量的值之间互相矛盾。

而在State模式中，是用类来表示状态的。这样，我们就只需要一个表示系统状态的变量即可。在示例程序中，SafeFrame类的state字段就是这个变量，它决定了系统的状态。因此，不会存在自相矛盾的状态。

### 易于增加新的状态

能够增加新的类，但是，在State模式中，增加其他”以来状态的处理“是很困难的。

### 实例的多面性

请注意SafeFrame类中的以下两条语句。

- SafeFrame类的构造函数中的

  ```java
  buttonUse.addActionListener(this);
  ```

- actionPerformed方法中的

  ```java
  state.doUse(this);
  ```

这两条语句中都有this。那么这个this到底是什么呢？当然，它们都是SafeFrame类的实例。由于在示例程序中只生成了一个Saferrame的实例，因此这两个this其实是同一个对象。

不过，在addActionListener方法中和doUse方法中，对this的使用方式是不一样的。

向addActionListener方法传递this时 **，该实例会被当作“实现了ActionListener接口的类的实例”来使用**。这是因为addActionListener方法的参数类型是ActionListener类型。在addActionListener方法中会用到的方法也都是在ActionListener接口中定义了的方法。至于这个参数是否是SafeFrame类的实例并不重要。

向doUse方法传递this时，**该实例会被当作“实现了Context接口的类的实例”来使用**。这是因为doUse方法的参数类型是Context类型。在doUse方法中会用到的方法也都是在Context接口中定义了的方法（大家只要再回顾一下DayState类和NightState类的doUse
方法就会明白了)。

请大家一定要透彻理解此处的实例的多面性。

## 相关设计模式

- [[05_Singleton模式]]
- [[flyweight模式]]

