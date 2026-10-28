---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
## Strategy模式

无论什么程序，其目的都是为了解决问题，我们有需要编写特定的算法。使用Strategy模式可以整体地替换算法的实现部分。能够整体地替换算法，能让我们轻松地以不同算法去解决一个问题，这就是策略模式。

## 例子

功能是让电脑“猜拳”的游戏。

考虑两种策略，一种是“如果这局猜拳获胜，那么下一局也出一样的手势”，另一种策略是“根据上一局的手势从概率上计算出下一局的手势”。

| 名字            | 说明                                                   |
| --------------- | ------------------------------------------------------ |
| Hand            | 表示猜拳游戏中的“手势”的类                             |
| Strategy        | 表示猜拳游戏中的策略的类                               |
| WinningStrategy | 表示“如果这局猜拳获胜，那么下一局也出一样的手势”的策略 |
| ProbStrategy    | 表示”根据上一局的手势从概率上计算出下一局的手势“的策略 |
| Player          | 表示进行猜拳游戏的选手的类                             |
| Main            | 测试程序                                               |

![image-20221014100130489](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221014100130489-1665712891396-1.png)

Hand类时表示猜拳游戏中”手势“的类。在这个类内部，用int表示所出手势，其中0表示石头，1表示剪刀，2表示布，并将值保存在handValue字段中。

Hand类的实例可以通过实用类方法getHand来获取。只要将手势的值作为参数传递给getHand方法，就会将手势的值所对应的Hand类的实例返回给我们。只是一种Singleton模式。

isStrongerThan方法和isWeakerThan方法用于判断猜拳结果。

虽然Hand类会被其他类使用，但它并非Strategy模式中的角色。

```java
public class Hand {
    public static final int HANDVALUE_GUU = 0;
    public static final int HANDVALUE_CHO = 1;
    public static final int HANDVALUE_PAA = 2;
    public static final Hand[] hand = {
            new Hand(HANDVALUE_GUU),
            new Hand(HANDVALUE_CHO),
            new Hand(HANDVALUE_PAA),
    };

    private static final String[] name = {
            "石头", "剪刀", "布"
    };
    private int handValue;
    public Hand(int handValue) {
        this.handValue = handValue;
    }

    public static Hand getHand(int handValue) {
        return hand[handValue];
    }

    public boolean isStrongerThan(Hand h) {
        return fight(h) == 1;
    }

    public boolean isWeakerThan(Hand h) {
        return fight(h) == -1;
    }

    private int fight(Hand h) {
        if (this == h) {
            return 0;
        } else if ((this.handValue + 1) % 3 == h.handValue) {
            return 1;
        } else {
            return -1;
        }
    }

    @Override
    public String toString() {
        return name[handValue];
    }
}
```

Strategy接口定义了猜拳策略的抽象方法的接口。

nextHand方法的作用是“获取下一局要出的手势”。

study方法的作用是“上一局手势是否获胜了”。

```java
public interface Strategy {
    public abstract Hand nextHand();
    public abstract void study(boolean win);
}
```

接下来是策略的实现。

WinningStrategy类是Strategy接口的实现类之一。使用的策略是“如果这局猜拳获胜，那么下一局也出一样的手势”。

```java
public class WinningStrategy implements Strategy{
    private Random random;
    private boolean won = false;
    private Hand prevHand;

    public WinningStrategy(int seed) {
        random = new Random(seed);
    }
    @Override
    public Hand nextHand() {
        if (!won) {
            prevHand = Hand.getHand(random.nextInt(3));
        }
        return prevHand;
    }

    @Override
    public void study(boolean win) {
        won = win;
    }
}
```

ProbStrategy类是另一种具体策略，实现根据上一局的手势从概率上计算出下一局的手势的策略。

istory字段是一个表，被用于根据过去的胜负来进行概率计算。它是一个二维数组，每个

数组下标的意思如下。

history`[上一局出的手势][这一局所出的手势]`

这个表达式的值越大，表示过去的胜率越高。下面稍微详细讲解下。

假设我们上一局出的是石头。

history`[0][0]`	两局分别出石头、石头时胜了的次数
history`[0][1]`	两局分别出石头、剪刀时胜了的次数
history`[0][2]`	两局分别出石头、布时胜了的次数

那么，我们就可以根据history`[0][0]`、history`[0][1]`、history`[0][2]`这3个表达式的值从概率上计算出下一局出什么。简而言之，就是先计算3个表达式的值的和(getSum方法)，然后再从0与这个和之间取一个随机数，并据此决定下一局应该出什么（nextHand方法)。

例如，如果

history`[0][0]`是3

history`[0][1]`是5

history`[0][2]`是7

那么，下一局出什么就会以石头、剪刀和布的比率为3：5：7来决定。然后在0至15（不含15,15是3+5+7的和)之间取一个随机数。

- 如果该随机数在0至3（不含3）之间，那么出石头
- 如果该随机数在3至8（不含8）之间，那么出剪刀
- 如果该随机数在8至15（不含15）之间，那么出布

study方法会根据nextHand方法返回的手势的胜负结果来更新history字段中的值。

> 注意：此策略的大前提是对方只有一种猜拳模式

```java
public class ProbStrategy implements Strategy{
    private Random random;
    private int preHandValue = 0;
    private int currentHandValue = 0;
    private int[][] history = {
            {1, 1, 1,},
            {1, 1, 1,},
            {1, 1, 1,}
    };

    public ProbStrategy(int seed) {
        random = new Random(seed);
    }

    @Override
    public Hand nextHand() {
        int bet = random.nextInt(getSum(currentHandValue));
        int handValue = 0;
        if (bet < history[currentHandValue][0]) {
            handValue = 0;
        } else if (bet < history[currentHandValue][0] + history[currentHandValue][1]) {
            handValue = 1;
        } else {
            handValue = 2;
        }
        preHandValue = currentHandValue;
        currentHandValue = handValue;
        return Hand.getHand(handValue);
    }

    @Override
    public void study(boolean win) {
        if (win) {
            history[preHandValue][currentHandValue]++;
        } else {
            history[preHandValue][(currentHandValue + 1) % 3]++;
            history[preHandValue][(currentHandValue + 2) % 3]++;
        }
    }

    private int getSum(int preHandValue) {
        int sum = 0;
        for (int i = 0; i < 3; i++) {
            sum += history[preHandValue][i];
        }
        return sum;
    }
}
```

Player是表示进行猜拳游戏的选手的类。

```java
public class Player {
    private String name;
    private Strategy strategy;
    private int wincount;
    private int losecount;
    private int gamecount;
    public Player(String name, Strategy strategy) {
        this.name = name;
        this.strategy = strategy;
    }

    public Hand nextHand() {
        return strategy.nextHand();
    }

    public void win() {
        strategy.study(true);
        wincount++;
        gamecount++;
    }

    public void even() {
        gamecount++;
    }

    public void lose() {
        strategy.study(false);
        losecount++;
        gamecount++;
    }

    @Override
    public String toString() {
        return "Player{" +
                "name='" + name + '\'' +
                ", strategy=" + strategy +
                ", wincount=" + wincount +
                ", losecount=" + losecount +
                ", gamecount=" + gamecount +
                '}';
    }
}
```

Main类进行测试

```java
public class Main {
    public static void main(String[] args) {
        if (args.length != 2) {
            System.out.println("Usage: java Main randomseed1 radomseed2");
            System.out.println("Example: java Main 314 315");
            System.exit(0);
        }
        int seed1 = Integer.parseInt(args[0]);
        int seed2 = Integer.parseInt(args[1]);
        Player player1 = new Player("Taro", new WinningStrategy(seed1));
        Player player2 = new Player("Hana", new ProbStrategy(seed2));
        for (int i = 0; i < 1000; i++) {
            Hand nextHand1 = player1.nextHand();
            Hand nextHand2 = player2.nextHand();
            if (nextHand1.isStrongerThan(nextHand2)) {
                System.out.println("Winner: " + player1);
                player1.win();
                player2.lose();
            } else if (nextHand2.isStrongerThan(nextHand1)) {
                System.out.println("Winner: " + player2);
                player2.win();
                player1.lose();
            } else {
                System.out.println("Even...");
                player1.even();
                player2.even();
            }
        }
        System.out.println("Total result: ");
        System.out.println(player1.toString());
        System.out.println(player2.toString());
    }
}
```

## 登场角色

- Strategy（策略）

  负责决定实现策略所必需的接口。

- ConcreteStrategy（具体的策略）

  负责实现Strategy角色的接口，和负责具体的策略。

- Context（上下文）

  负责使用Strategy角色。Context角色保存了ConcreteStrategy角色的实例，并使用ConcreteStrategy角色实现需求。

![image-20221014102232318](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221014102232318-1665714153422-3.png)

## 总结

### 为什么需要特意编写Strategy角色

通常在编程时算法会被写在具体方法中。Strategy模式却特意将算法与其他部分分离开来，只是定义了与算法相关的接口(API),然后在程序中以委托的方式来使用算法。

这样看起来程序好像变复杂了，其实不然。例如，当我们想要通过改善算法来提高算法的处理速度时，如果使用了Strategy模式，就不必修改Strategy角色的接口(API)了，仅仅修改ConcreteStrategy角色即可。而且，**使用委托这种弱关联关系可以很方便地整体替换算法**，例如，如果想比较原来的算法与改进后的算法的处理速度有多大区别，简单地替换下算法即可进行测试。

使用Strategy模式编写象棋程序时，可以方便地根据棋手的选择切换AI例程的水平。

### 程序运行中也可以切换策略

如果使用Strategy模式，在程序运行中也可以切换ConcreteStrategy角色。例如，在内存容量少的运行环境中可以使用SlowButLessMemoryStrategy（速度慢但省内存的策略)，而在内存容量多的运行环境中则可以使用FastButMoreMemoryStrategy(速度快但耗内存的策略)。

此外，还可以用某种算法去“验算”另外一种算法。例如，假设要在某个表格计算软件的开发版本中进行复杂的计算。这时，我们可以准备两种算法，即“高速但计算上可能有错误的算法”和“低速但计算准确的算法”，然后让后者去验算前者的计算结果。

## 相关设计模式

- [[flyweight模式]]
- [[08_Abstract_Fatctory模式]]
- [[19_State模式]]
