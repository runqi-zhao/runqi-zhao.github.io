---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
本节将会介绍Fork/Join框架的基本原理、算法、设计方式、应用与实现等。

## 基本原理

Fork：将大任务切割成小任务；

Join：将小任务合并成大任务。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230218163759.png)

<center>Fork Join的运行流程图</center>

### 工作窃取算法

工作窃取（work-stealing）算法是指某个线程从其他队列里窃取任务来执行。

目的：将任务分别放入不同的队列，执行的结束时间也不相同，有的队列会先执行结束，因此，“窃取”其他队列中的未执行的任务进行执行。

过程：会使用双端队列，被窃取任务线程永远从双端队列的**头部**拿任务执行，而窃取任务的线程永远从双端队列的**尾部**拿任务执行。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230218164049.png)

<center>工作窃取运行流程</center>

优点：充分利用线程进行并行计算，减少了线程间的竞争。

缺点：在某些情况下还是存在竞争，比如双端队列里只有一个任务时。并且该算法会消耗了更多的系统资源，比如创建多个线程和多个双端队列。

## Fork/Join框架的设计

分为两步：

1. <font color="red">分割任务。</font>首先我们需要有一个fork类来把大任务分割成子任务，有可能子任务还是很大，所以还需要不停地分割，直到分割出的子任务足够小。
2. <font color="red">执行任务并合并结果。</font>分割的子任务分别放在双端队列里，然后几个启动线程分别从双端队列里获取任务执行。子任务执行完的结果都统一放在一个队列里，启动一个线程从队列里拿数据，然后合并这些数据。

Fork/Join使用两个类来完成以上两件事情。

1. ForkJoinTask：使用ForkJoin框架，必须首先创建一个ForkJoin任务。它提供在任务中执行fork()和join()操作的机制。通常情况下，我们不需要直接继承ForkJoinTask类，只需要继承它的子类，Fork/Join框架提供了以下两个子类。
   1. RecursiveAction：用于没有返回结果的任务。
   2. RecursiveTask：用于有返回结果的任务。
2. ForkJoinPool：ForkJoinTask需要通过ForkJoinPool来执行。

任务分割出的子任务会添加到当前工作线程所维护的双端队列中，进入队列的头部。当一个工作线程的队列里暂时没有任务时，它会随机从其他工作线程的队列的尾部获取一个任务。

## 实例

让我们通过一个简单的需求来使用Fork/Join框架，需求是：计算1+2+3+4的结果。

使用Fork/Join框架首先要考虑到的是如何分割任务，如果希望每个子任务最多执行两个数的相加，那么我们设置分割的阈值是2，由于是4个数字相加，所以Fork/Join框架会把这个任务fork成两个子任务，子任务一负责计算1+2，子任务二负责计算3+4，然后再join两个子任务的结果。因为是有结果的任务，所以必须继承RecursiveTask，实现代码如下。

```java
public class CountTask extends RecursiveTask<Integer> {
    public static final int THRESHOLD = 2; //阈值
    private int start;
    private int end;

    public CountTask(int start, int end) {
        this.start = start;
        this.end = end;
    }
    @Override
    protected Integer compute() {
        int sum = 0;
        // 如果任务足够小就计算任务
        boolean canCompute = (end - start) < THRESHOLD;
        if (canCompute) {
            for (int i = start; i <= end; i++) {
                sum += i;
            }
        } else {
            // 如果任务大于阈值，就分裂成两个子任务计算
            int middle =(start + end) / 2;
            CountTask leftTask = new CountTask(start, middle);
            CountTask rightTask = new CountTask(middle + 1, end);
            //执行子任务
            leftTask.fork();
            rightTask.fork();
            //等子任务执行完，并得到其结果
            int leftResult = leftTask.join();
            int rightResult = rightTask.join();
            // 合并子任务
            sum = leftResult + rightResult;
        }
        return sum;
    }

    public static void main(String[] args) {
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        //生成一个任务，1+2+3+4
        CountTask task = new CountTask(1, 4);
        //执行一个任务
        Future<Integer> result = forkJoinPool.submit(task);
        try {
            System.out.println(result.get());
        } catch (InterruptedException | ExecutionException e) {
        }
    }
}
```

通过这个例子，我们进一步了解ForkJoinTask；

ForkJoinTask与一般任务的主要区别在于它需要实现compute方法；在此方法中：

- 首先需要判断任务是否足够小，如果足够小就直接执行任务。
- 如果不足够小，就必须分割成两个子任务；
  - 每个子任务在调用fork方法时，又会进入compute方法，看看当前子任务是否需要继续分割成子任务，
    - 如果不需要继续分割，则执行当前子任务并返回结果。
  - 使用join方法会等待子任务执行完并得到其结果

## 异常处理

ForkJoinTask提供了isCompletedAbnormally()方法来检查任务是否已经抛出异常或已经被取消了，并且可以通过ForkJoinTask的getException方法获取异常。代码如下：

```java
if(task.isCompletedAbnormally())
{
    System.out.println(task.getException());
}
```

getException方法返回Throwable对象，如果任务被取消了则返回CancellationException。如果任务没有完成或者没有抛出异常则返回null。

## 实现原理

ForkJoinPool由ForkJoinTask数组和ForkJoinWorkerThread数组组成；

- ForkJoinTask数组负责将存放程序提交给ForkJoinPool的任务；
- ForkJoinWorkerThread数组负责执行这些任务。

### ForkJoinTask的fork方法实现原理

当调用ForkJoinTask的fork方法时，程序会调用ForkJoinWorkerThread的pushTask方法异步的执行这个任务，然后立刻返回结果。

```java
public final ForkJoinTask<V> fork() {
        ((ForkJoinWorkerThread) Thread.currentThread())
            .pushTask(this);
        return this;
}
```

流程如下：

1. pushTask方法把当前任务存放在ForkJoinTask数组队列里。
2. 再调用ForkJoinPool的signalWork()方法唤醒或创建一个工作线程来执行任务。

代码如下。

```java
final void pushTask(ForkJoinTask<> t) {
    ForkJoinTask<>[] q; int s, m;
    if ((q = queue) != null) {    // ignore if queue removed
        long u = (((s = queueTop) & (m = q.length - 1)) << ASHIFT) + ABASE;
        UNSAFE.putOrderedObject(q, u, t);
        queueTop = s + 1;      // or use putOrderedInt
        if ((s -= queueBase) <= 2)
            pool.signalWork();
        else if (s == m)
            growQueue();
    }
}
```

### ForkJoinTask的join方法实现原理

Join方法的主要作用是阻塞当前线程并等待获取结果。ForkJoinTask的join方法的实现如下：

```java
public final V join() {
        if (doJoin() != NORMAL)
            return reportResult();
        else
            return getRawResult();
}
private V reportResult() {
        int s; Throwable ex;
        if ((s = status) == CANCELLED)
            throw new CancellationException();
        if (s == EXCEPTIONAL && (ex = getThrowableException()) != null)
            UNSAFE.throwException(ex);
        return getRawResult();
    }
```

1. 调用了doJoin()方法；
2. 通过doJoin()方法得到当前任务的状态来判断返回什么结果doJoin()有以下四种状态：
   1. 已完成(NORMAL)：任务是已完成，则直接返回任务结果；
   2. 被取消(CANCELLED)：任务是已取消，直接抛出异常CancellationException；
   3. 信号(SIGNAL)
   4. 异常(EXCEPTIONAL)：任务是抛出异常，直接抛出对应的异常；

doJoin()方法实现如下：

```java
private int doJoin() {
    Thread t; ForkJoinWorkerThread w; int s; boolean completed;
    if ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread) {
        if ((s = status) < 0)
            return s;
        if ((w = (ForkJoinWorkerThread)t).unpushTask(this)) {
            try {
                completed = exec();
            } catch (Throwable rex) {
                return setExceptionalCompletion(rex);
            }
            if (completed)
                return setCompletion(NORMAL);
        }
        return w.joinTask(this);
    }
    else
        return externalAwaitDone();
}
```

在doJoin()方法里：

- 通过查看任务的状态，看任务是否已经执行完成；
  1. 如果执行完成，则直接返回任务状态；如果任务顺利执行完成，则设置任务状态为NORMAL；
  2. 如果没有执行完，则从任务数组里取出任务并执行；如果出现异常，则记录异常，并将任务状态设置为EXCEPTIONAL。

