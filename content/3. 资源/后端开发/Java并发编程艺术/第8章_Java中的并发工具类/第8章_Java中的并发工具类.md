---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
在JDK的并发包里提供了几个非常有用的并发工具类。CountDownLatch、CyclicBarrier和Semaphore工具类提供了一种并发流程控制的手段，Exchanger工具类则提供了在线程间交换数据的一种手段。

## 等待多线程完成的CountDownLatch

CountDownLatch允许一个或多个线程等待其他线程完成操作。

假如有这样一个需求：我们需要解析一个Excel里多个sheet的数据，此时可以考虑使用多线程，每个线程解析一个sheet里的数据，等到所有的sheet都解析完之后，程序需要提示解析完成。在这个需求中，要实现主线程等待所有线程完成sheet的解析操作，最简单的做法是使用join()方法：

```java
package com.example.adtper02;

public class JoinCountDownLatchTest {
    public static void main(String[] args) throws InterruptedException {
        Thread parser1 = new Thread(new Runnable() {
            @Override
            public void run() {

            }
        });
        Thread parser2 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("parser2 finish.");
            }
        });
        parser1.start();
        parser2.start();
        parser1.join();
        parser2.join();
        System.out.println("all parser finish");
    }
}

```

join用于让当前执行线程等待join线程执行结束。执行原理是不停检查join线程是否存货，如果join线程存在则让当前线程永远等待。其中，wait(0)表示永远等待下去：

```java
while (isAlive()) {
	wait(0);
}
```

知道join终止后，线程的this.notifyAll()方法会调用，调用notifyAll()是在JVM里实现的。

在JDK 1.5之后的并发包中提供的CountDownLatch也可以实现join的功能，并且比join的功能更多：

```java
public class CountDownLatchTest {
    static CountDownLatch c = new CountDownLatch(2);

    public static void main(String[] args) throws InterruptedException {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(1);
                c.countDown();
                System.out.println(2);
                c.countDown();
            }
        }).start();
        c.await();
        System.out.println("3");
    }
}
```

CountDownLatch的构造函数结果后一个int类型的参数作为计数器，如果你想等待N个点完成，这里就传入N。

当我们调用CountDownLatch的countDown方法时，N就会减1，CountDownLatch的await方法会阻塞当前线程，直到N变成零。由于countDown方法可以用在任何地方，所以这里说的N个点，可以是N个线程，也可以是1个线程里的N个执行步骤。用在多个线程时，只需要把这个CountDownLatch的引用传递到线程里即可。

如果有某个解析sheet的线程处理得比较慢，我们不可能让主线程一直等待，所以可以使用另外一个带指定时间的await方法——await（long time，TimeUnit unit），这个方法等待特定时间后，就会不再阻塞当前线程。join也有类似的方法。

注意：**计数器必须大于等于0**。计数器等于0的时候，调用await发给发是不会阻塞线程。<font color="red">CountDownLatch不可能重新初始化或者修改CountDownLatch对象的内部计数器的值</font>。一个线程调用countDown方法happen-before，另外一个线程调用await方法。

## 同步屏障CyclicBarrier

目标：让一组线程到达一个屏障（同步点）时阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。

### 简介

CyclicBarrier默认的构造方法是CyclicBarrier（int parties），其参数表示屏障拦截的线程数量，每个线程调用await方法告诉CyclicBarrier我已经到达了屏障，然后当前线程被阻塞。

```java
package com.example.adtper02;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierTest {
    static CyclicBarrier c = new CyclicBarrier(2);

    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    c.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    throw new RuntimeException(e);
                }
                System.out.println(1);
            }
        }).start();
        try {
            c.await();
        } catch (InterruptedException | BrokenBarrierException e) {
            throw new RuntimeException(e);
        }
        System.out.println(2);
    }
}

```

由于主线程和子线程调度是由CPU决定的，两个线程都有可能先执行。

如果把new CyclicBarrier(2)修改改成new CyclicBarrier(3)，则主线程和子线程永远等待。

CyclicBarrier还提供一个更高级的构造函数CyclicBarrier（int parties，Runnable barrier-Action），用于在线程到达屏障时，优先执行barrierAction，方便处理更复杂的业务场景。

```java
import java.util.concurrent.CyclicBarrier;
public class CyclicBarrierTest2 {
        static CyclicBarrier c = new CyclicBarrier(2, new A());
        public static void main(String[] args) {
                new Thread(new Runnable() {
                        @Override
                        public void run() {
                                try {
                                        c.await();
                                } catch (Exception e) {
                                       }
                                System.out.println(1);
                        }
                }).start();
                try {
                        c.await();
                } catch (Exception e) {
                }
                System.out.println(2);
        }
        static class A implements Runnable {
                @Override
                public void run() {
                        System.out.println(3);
                }
        }
}
```

输出结果如下

```
3
1
2
```

CyclicBarrier设置了拦截线程的数量是2，所以必须等代码中的第一个线程和线程A都执行完之后，才会继续执行主线程，然后输出2。

## 应用场景

CyclicBarrier可以用于多线程计算数据，最后合并计算结果的场景。

例如，用一个Excel保存了用户所有银行流水，每个Sheet保存一个账户近一年的每笔银行流水，现在需要统计用户的日均银行流水，先用多线程处理每个sheet里的银行流水，都执行完之后，得到每个sheet的日均银行流水，最后，再用barrierAction用这些线程的计算结果，计算出整个Excel的日均银行流水。

```java
package com.example.adtper02;

import java.util.Map;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.Executor;
import java.util.concurrent.Executors;

public class BankWaterService implements Runnable {
    private CyclicBarrier c = new CyclicBarrier(4, this);
    private Executor executor = Executors.newFixedThreadPool(4);
    private ConcurrentHashMap<String, Integer> sheetBankWaterCount = new ConcurrentHashMap<>();

    private void count() {
        for (int i = 0; i < 4; i++) {
            executor.execute(new Runnable() {
                @Override
                public void run() {
                    sheetBankWaterCount.put(Thread.currentThread().getName(), 1);
                    try {
                        c.await();
                    } catch (InterruptedException | BrokenBarrierException e) {
                        throw new RuntimeException(e);
                    }
                }
            });
        }
    }
    @Override
    public void run() {
        int result = 0;
        for (Map.Entry<String, Integer> sheet : sheetBankWaterCount.entrySet()) {
            result += sheet.getValue();
        }
        sheetBankWaterCount.put("result", result);
        System.out.println(result);
    }

    public static void main(String[] args) {
        BankWaterService bankWaterCount = new BankWaterService();
        bankWaterCount.count();
    }
}

```

使用线程池创建4个线程，分别计算每个sheet里的数据，每个sheet计算结果是1，再由BankWaterService线程汇总4个sheet计算出的结果，输出结果如下。

```
4
```

### CyclicBarrier和CountDownLatch的区别

1. CountDoenLatch的计数器只能使用一次，而CyclicBarrier的计数器可以使用reset()方法重置。
2. CyclicBarrier还提供其他有用的方法，比如getNumberWaiting方法可以获得Cyclic-Barrier阻塞的线程数量。isBroken()方法用来了解阻塞的线程是否被中断。

```java
public class CyclicBarrierTest3 {
    static CyclicBarrier c = new CyclicBarrier(2);

    public static void main(String[] args) throws InterruptedException, BrokenBarrierException {
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    c.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                }
            }
        });
        thread.start();
        thread.interrupt();
        try {
            c.await();
        } catch (InterruptedException | BrokenBarrierException e) {
            System.out.println(c.isBroken());
        }
    }
}
```

输出true。

## 控制并发线程数的Semaphore

semaphore（信号量）是用来控制同时访问特定资源的线程数量，它通过协调各个线程，以保证合理的使用公共资源。

多年以来，我都觉得从字面上很难理解Semaphore所表达的含义，只能把它比作是控制流量的红绿灯。比如××马路要限制流量，只允许同时有一百辆车在这条路上行使，其他的都必须在路口等待，所以前一百辆车会看到绿灯，可以开进这条马路，后面的车会看到红灯，不能驶入××马路，但是如果前一百辆中有5辆车已经离开了××马路，那么后面就允许有5辆车驶入马路，这个例子里说的车就是线程，驶入马路就表示线程在执行，离开马路就表示线程执行完成，看见红灯就表示线程被阻塞，不能执行。

### 应用场景

进行流量控制时使用，热别是共用资源有限的应用场景，比如数据库连接。

```java
package com.example.adtper02;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class SemaphoreTest {
    private static int THREAD_COUNT = 30;
    private static ExecutorService threadPool = Executors.newFixedThreadPool(THREAD_COUNT);
    private static Semaphore s = new Semaphore(10);

    public static void main(String[] args) {
        for (int i = 0; i < THREAD_COUNT; i++) {
            threadPool.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        s.acquire();
                        System.out.println("save data");
                        s.release();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }
            });
        }
        threadPool.shutdown();
    }
}

```

在代码中，虽然有30个线程在执行，但是只允许10个并发执行。

Semaphore的构造方法Semaphore（int permits）接受一个整型的数字，表示可用的许可证数量。

Semaphore（10）表示允许10个线程获取许可证，也就是最大并发数是10。

Semaphore的用法也很简单，首先线程使用Semaphore的acquire()方法获取一个许可证，使用完之后调用release()方法归还许可证。还可以用tryAcquire()方法尝试获取许可证。

### 其他方法
semaphore还提供一些其他方法，具体如下。

- intavailablePermits()：返回此信号量中当前可用的许可证数。
- intgetQueueLength()：返回正在等待获取许可证的线程数。
- booleanhasQueuedThreads()：是否有线程正在等待获取许可证。
- void reducePermits（int reduction）：减少reduction个许可证，是个protected方法。
- Collection getQueuedThreads()：返回所有等待获取许可证的线程集合，是个protected方法。

## 线程间交换数据的Exchanger

Exchanger是一个用于线程间写作的工具类。Exchanger用于线程间的数据交换。

1. 提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。
2. 这两个线程通过exchange方法交换数据，如果第一个线程先执行exchange()方法，它会一直等待第二个线程也执行exchange方法，当两个线程都到达同步点时，这两个线程就可以交换数据，将本线程生产出来的数据传递给对方。

### 应用场景

1. **Exchanger可以用于遗传算法** ，遗传算法里需要选出两个人作为交配对象，这时候会交换两人的数据，并使用交叉规则得出2个交配结果。

2. **Exchanger也可以用于校对工作** ，比如我们需要将纸制银行流水通过人工的方式录入成电子银行流水，为了避免错误，采用AB岗两人进行录入，录入到Excel之后，系统需要加载这两个Excel，并对两个Excel数据进行校对：

   ```java
   public class ExchangerTest {
       private static final Exchanger<String> exgr = new Exchanger<>();
       private static ExecutorService threadPool = Executors.newFixedThreadPool(2);
   
       public static void main(String[] args) {
           threadPool.execute(new Runnable() {
               @Override
               public void run() {
                   try {
                       String A = "银行流水A";
                       exgr.exchange(A);
                   } catch (InterruptedException e) {
                   }
               }
           });
           threadPool.execute(new Runnable() {
               @Override
               public void run() {
                   try {
                       String B = new String("银行流水B");
                       String A = exgr.exchange(B);
                       System.out.println("A和B数据是否一致：" + A.equals(B) + ", A录入的是：" + A + "，B录入是：" + B);
                   } catch (InterruptedException e) {
   
                   }
               }
           });
       }
   }
   ```

​	如果两个线程有一个没有执行exchange()方法，则会一直等待，如果担心有特殊情况发生，避免一直等待，可以使用exchange（V x，longtimeout，TimeUnit unit）设置最大等待时长。

## 小结

本章配合一些应用场景介绍JDK中提供的几个并发工具类，大家记住这个工具类的用途，一旦有对应的业务场景，不妨试试这些工具类。
