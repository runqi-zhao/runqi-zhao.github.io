---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
java线程池带来的好处：

1. **降低资源消耗。** 通过重复利用一创建的线程降低线程创建和销毁造成的消耗。
2. **提高响应速度。** 当任务到达时，任务可以不需要等到线程创建就能立即执行。
3. **提高线程的可管理性。** 线程是稀缺资源，如果无限制地创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一分配、调优和监控。但是，要做到合理利用线程池，必须对其实现原理了如指掌。

## 线程池的实现原理

线程池的处理流程图如下图所示：

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/NeatReader-1676885482555.png)

<center>线程池的主要处理流程</center>

1. 线程池判断核心线程池是否都在执行任务。
   1. 如果不是，则创建一个新的工作线程来执行任务。
   2. 如果核心线程都在执行任务，则进入下一个流程。
2. 线程池判断工作队列是否已经满。
   1. 如果工作队列没有满，则将新提交的任务存储在这个工作队列中。
   2. 如果已经满了，进入下一个流程。
3. 线程池判断线程池的线程是否都处于工作状态。
   1. 如果没有，则创建一个新的工作线程来执行。
   2. 如果都处于工作状态，则交给[[饱和策略]]处理这个任务。

ThreadPoolExecutor执行execute()方法的示意图，如下图所示：

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/NeatReader-1676886082354.png) ^lw05q1

ThreadPoolExecutor执行execute方法分下面4种情况。

1. 如果当前线程少于corePoolSize，则创建新线程执行任务（<font color="red">执行这一步需要获取全局锁。</font>）
2. 如果运行的线程等于或多余corePoolSize，则将任务加入BlockingQueue。
3. 如果无法将任务加入BlockingQueue（队列已满），则创建新的线程来处理（<font color="red">执行这一步需要获取全局锁。</font>）
4. 如果创建新线程将使当前运行的线程超出maximunPoolSize，任务被拒绝，并调用RejectedExecutionHandler.rejectedExecution()方法。

**目的**：在执行execute()方法时，**尽可能地避免获取全局锁**（那将会是一个严重的可伸缩瓶颈）。在ThreadPoolExecutor完成预热之后（当前运行的线程数大于等于corePoolSize），几乎所有的execute()方法调用都是执行步骤2，而步骤2不需要获取全局锁。

从源码角度分析线程池的工作原理：

```java
public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        // 如果线程数小于基本线程数，则创建线程并执行当前任务 
        if (poolSize >= corePoolSize || !addIfUnderCorePoolSize(command)) {
        // 如线程数大于等于基本线程数或线程创建失败，则将当前任务放到工作队列中。
        if (runState == RUNNING && workQueue.offer(command)) {
        if (runState != RUNNING || poolSize == 0)
            ensureQueuedTaskHandled(command);
        }
        // 如果线程池不处于运行中或任务无法放入队列，并且当前线程数量小于最大允许的线程数量，
        // 则创建一个线程执行任务。
        else if (!addIfUnderMaximumPoolSize(command))
            // 抛出RejectedExecutionException异常
        	reject(command); // is shutdown or saturated
    }
}
```

工作流程：

1. 创建线程池时，会将线程封装成工作线程Worker；
2. Worker在执行完任务后，还会玄幻获取工作队列里的任务来执行。

从Woker类里面的run()方法可以看出这点：

```java
public void run() {
    try {
        Runnable task = firstTask;
        firstTask = null;
        while (task != null || (task = getTask()) != null) {
          runTask(task);
          task = null;
        }
    } finally {
       workerDone(this);
    }
}
```

ThreadPoolExecutor中线程执行任务的示意图如下：

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/NeatReader-1676886767294.png)

<center>ThreadPoolExecutor执行任务示意图</center>

线程池中的线程执行任务分两种情况，如下。

1. 在execute()方法中创建一个线程时，会让这个线程执行当前任务。
2. 这个线程执行完上图中1的任务后，会反复从BlockingQueue获取任务来执行。

## 线程池的使用

### 线程池的创建

<font color="red">通过ThreadPoolExecutor来创建一个线程池。</font>

```java
new  ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime,milliseconds,runnableTaskQueue, handler);
```

建一个线程池时需要输入几个参数，如下：

1. corePoolSize（线程池的基本大小）：当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有基本线程。

2. runnableTaskQueue（任务队列）：用于保存等待执行的任务的阻塞队列。可以选择以下几个阻塞队列。
   - ArrayBlockingQueue：是一个基于数组结构的有界阻塞队列，此队列按FIFO（先进先出）原则对元素进行排序。
   - LinkedBlockingQueue：一个基于链表结构的阻塞队列，此队列按FIFO排序元素，吞吐量通常要高于ArrayBlockingQueue。静态工厂方法Executors.newFixedThreadPool()使用了这个队列。
   - SynchronousQueue：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于Linked-BlockingQueue，静态工厂方法Executors.newCachedThreadPool使用了这个队列。
   - PriorityBlockingQueue：一个具有优先级的无限阻塞队列。

3. maximumPoolSize（线程池最大数量）：线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。值得注意的是，如果使用了无界的任务队列这个参数无效。

4. ThreadFactory：用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字。使用开源框架guava提供的ThreadFactoryBuilder可以快速给线程池里的线程设置有意义的名字，代码如下。

   ```java
   new ThreadFactoryBuilder().setNameFormat("XX-task-%d").build();
   ```

5. RejectedExecutionHandler（[饱和策略]）：当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常。在JDK 1.5中Java线程池框架提供了以下4种策略。

   - bortPolicy：直接抛出异常。

   - CallerRunsPolicy：只用调用者所在线程来运行任务。

   - DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。

   - DiscardPolicy：不处理，丢弃掉。

     当然，也可以根据应用场景需要来实现RejectedExecutionHandler接口自定义策略。如记录日志或持久化存储不能处理的任务。

6. keepAliveTime（线程活动保持时间）：线程池的工作线程空闲后，保持存活的时间。所以，如果任务很多，并且每个任务执行的时间比较短，可以调大时间，提高线程的利用率。

7. TimeUnit（线程活动保持时间的单位）：可选的单位有天（DAYS）、小时（HOURS）、分钟（MINUTES）、毫秒（MILLISECONDS）、微秒（MICROSECONDS，千分之一毫秒）和纳秒（NANOSECONDS，千分之一微秒）。

### 向线程池提交任务

两种方式进行线程池的提交任务。

#### execute()方法

特点：没有返回值，无法判断线程是否执行成功。输入的是一个Runnable类的实例。

```java
threadsPool.execute(new Runnable() {
                        @Override
                        public void run() {
                                // TODO Auto-generated method stub
                        }
                });

```

#### submit()方法

特点：有返回值，返回一个futuer类星星的对象，通过这个future对象判断任务是否创建成功，并且可以通过future的get()方法来获取返回值，get()方法会阻塞当前线程直到任务完成，而使用get（long timeout，TimeUnit unit）方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。

```java
Future<Object> future = executor.submit(harReturnValuetask);
try {
        Object s = future.get();
} catch (InterruptedException e) {
        // 处理中断异常
} catch (ExecutionException e) {
        // 处理无法执行任务异常
} finally {
        // 关闭线程池
        executor.shutdown();
}

```

### 关闭线程池

通过shutdown与shutdownNow进行关闭。

#### shutdownNow

遍历线程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程，所以无法响应中断的任务可能永远无法终止。

shutdownNow首先将线程池的状态设置成STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表。

#### shutdown

遍历线程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程，所以无法响应中断的任务可能永远无法终止。

shutdown只是将线程池的状态设置成SHUTDOWN状态，然后中断所有没有正在执行任务的线程。

#### 二者如何进行关闭

只要调用了这两个关闭方法中的任意一个，isShutdown方法就会返回true。当所有的任务都已关闭后，才表示线程池关闭成功，这时调用isTerminaed方法会返回true。

至于应该调用哪一种方法来关闭线程池，应该由提交到线程池的任务特性决定，通常调用shutdown方法来关闭线程池，如果任务不一定要执行完，则可以调用shutdownNow方法。

### 合理的运用线程池

要想合理的配置线程池，首先分析任务特性，从以下几个角度分析：

- 任务的性质：CPU密集型任务、IO密集型任务和混合任务。
- 任务的优先级：高、中和低。
- 任务的执行时间：长、中和短。
- 任务的依赖性：是否依赖其他系统资源，例如数据库。

性质的不同任务可以用不同规模的线程池分开处理。

- CPU密集型任务应配置尽可能小的线程， 如配置$N_{cpu} + 1$个线程的线程池。
- IO密集型任务线程并不是一直在执行任务，则应配置尽可能多的线程，如$2\times N_{cpu}$
- 混合型的任务，如果可以拆分，将其拆分成一个CPU密集型任务和一个IO密集型任务：
  - 只要这两个任务执行的时间相差不是太大，那么分解后执行的吞吐量将高于串行执行的吞吐量。
  - 如果这两个任务执行时间相差太大，则没必要进行分解。可以通过Runtime.getRuntime().availableProcessors()方法获得当前设备的CPU个数。
- 优先级不同的任务可以使用优先级队列PriorityBlockingQueue来处理。它可以让优先级高的任务先执行。
  - 如果一直有优先级高的任务提交到队列里，那么优先级低的任务可能永远不能执行。
  - 执行时间不同的任务可以交给不同规模的线程池来处理，或者可以使用优先级队列，让执行时间短的任务先执行。
- 依赖数据库连接池的任务，因为线程提交SQL后需要等待数据库返回结果，等待的时间越长，则CPU空闲时间就越长，那么线程数应该设置得越大，这样才能更好地利用CPU。
- <font color ="red" size=6>建议使用有界队列。</font>有界队列能增加系统的稳定性和预警能力，可以根据需要设大一点儿，比如几千。

### 线程池的监控

通过线程池提供的参数进行监控，在监控线程池的时候可以使用以下属性。

- taskCount：线程池需要执行的任务数量。
- completedTaskCount：线程池在运行过程中已完成的任务数量，小于或等于taskCount。
- largestPoolSize：线程池里曾经创建过的最大线程数量。通过这个数据可以知道线程池是否曾经满过。如该数值等于线程池的最大大小，则表示线程池曾经满过。
- getPoolSize：线程池的线程数量。如果线程池不销毁的话，线程池里的线程不会自动销毁，所以这个大小只增不减。
- getActiveCount：获取活动的线程数。

通过扩展线程池进行监控。可以通过继承线程池来自定义线程池，重写线程池的beforeExecute、afterExecute和terminated方法，也可以在任务执行前、执行后和线程池关闭前执行一些代码来进行监控。

例如，监控任务的平均执行时间、最大执行时间和最小执行时间等。这几个方法在线程池里是空方法。

```java
protected void beforeExecute(Thread t, Runnable r) { }
```

