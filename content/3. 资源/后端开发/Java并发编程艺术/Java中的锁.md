---
id: 5
title: Java中的锁
sidebar_label: Java中的锁
description: 总结与《java并发之美》
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## Lock接口

与synchronized的区别：

1. Lock在进行使用时需要<font color="red">显示</font>地获取和释放锁。synchronized关键字会隐式地获取锁，但是将锁地获取和释放<font color="red">固化</font>了。

2. Lock拥有了锁获取与释放地可操作性。

   1. 尝试非阻塞获取锁；

   2. 可中断获取锁；

   3. 超时获取锁。

      上述三点具体描述看下表：

      Lock接口提供的synchronized关键字所不具备的主要特性如表所示。

      | 特性               | 描述                                                         |
      | ------------------ | ------------------------------------------------------------ |
      | 尝试非阻塞地获取锁 | 当前线程尝试获取锁，如果这一时刻没有被其他线程获取到，则成功获取并持有锁 |
      | 能被中断地获取锁   | 与synchronized不同，获取到锁的线程能够响应中断，当获取线程被中断时，终端一场将会被抛出，同时索贿被释放。 |
      | 超时获取锁         | 在指定截止时间之前获取锁，如果截至时间到了仍旧无法获取锁，则返回 |

举例说明Lock与synchrized的区别：

- 在某个场景中，手把手进行获取和释放；
  1. 先获得锁A，然后获得锁B；
  2. 当锁B获得后，释放锁A同时获得锁C；
  3. 当锁C获得后，在释放B同时获取D，以此类推。
- Lock可以手动（显示）得释放，在释放时相对容易很多。

Lock的使用的方式代码如下：

```java
Lock lock = new ReentrantLock();
lock.lock();
try {
} finally {
    lock.unlock();
}
```

<font size=6>请注意：</font>

1. 在finally块中释放锁，<font color="red">目的是保证在获取到锁之后，最终能够被释放。</font>
2. 不要将锁得获取放在try之中，如果在获取锁时发生了异常，异常抛出得同时，会导致锁释放。

Lock是一个接口，它定义了锁获取和释放的基本操作，Lock的API如下所示

| 方法名称                                                     | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| void lock()                                                  | 获取锁，调用该方法当前线程将会获取锁，当锁获得后，从该方法返回 |
| void lockInterruptibly() throw InterruptedException          | 可中断地获取锁，调用该方法立刻返回，如果获取则返回true，否则返回false |
| boolean tryLock(long time,TimeUnit unit)throws InterruptedException | 超时的获取锁，当前线程在以下3种情况会返回：<br />①当前线程在超时时间内获得了锁<br />②当前线程在超时时间内被中断<br />③超时时间结束，返回false |
| void unlock                                                  | 释放锁                                                       |
| Condition newCondition()                                     | 获取等待通知组件，该组件和当前的锁绑定，当前线程只有获得了锁，才能调用该组件的wait()方法。而调用之后，当前线程将释放锁。 |

随后的章节会详细介绍同步器
AbstractQueuedSynchronizer以及常用**Lock接口的实现ReentrantLock。Lock接口的实现基本都是**
**通过聚合了一个同步器的子类来完成线程访问控制的。**

## 队列同步器

队列同步器（AbstractQueueSynchronizer），是用于构建锁或者其他同步组件的基础框架。具有以下特性：

1. 使用一个int成员变量表示同步状态；
2. 通过内置的FIFO队列来完成资源获取线程的排队工作。

使用方式为：继承。子类需要重写三个方法：

- getState()
- setState()
- compareAndSetSate()

以上三个方法，<font color="red">都能保证线程是安全的。</font>

子类推荐定义为自定义同步组建的静态内部类。

队列同步器是实现锁的关键，在锁的实现种聚合同步器，利用同步器实现锁的语义。

锁与队列同步器的关系：

1. 锁是面向使用者的，定义了使用者与锁交互的接口（比如可以允许两个线程并行访问），隐藏实现细节；
2. 队列同步器面向的 的是锁的实现者，简化了锁的实现方式，屏蔽了同步状态管理、线程的排队、等待与唤醒等底层操作。

### 队列同步器的接口与示例

同步器的设计基于模板方法(Template Method)模式。

重写同步器指定的方法时，需要使用同步器提供的如下3个方法来访问或修改同步状态。

- getState()：获取同步器状态；
- setSate()：设置当前同步状态；
- compareAndSetSate(int expect, int update)：使用CAS设置当前状态，该方法能够保证状态设置的原子性。

同步器可重写的方法与描述如表5-3所示。

| 方法名称                                    | 描述                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| protected boolean tryAcquire(int arg)       | 独占获取同步状态，实现该方法需要查询当前状态并且判断同步状态是否符合预期，然后在进行CAS |
| protected boolean tryRelease(int arg)       | 独占式释放同步状态，等待获取同步状态的线程将有机会获取同步状态 |
| protected int tryAcquireShared(int arg)     | 共享式获取同步状态，返回大于等于0地值，表示获取成功，反之，  |
| protected boolean tryReleaseSgared(int arg) | 共享式获取同步状态，返回大于等于0的值，表示获取成功，反之获取失败 |
| protected boolean isHeldExclusively()       | 当前同步器是否在独占模式下被线程占用，一般该方法表示是否呗房前线程所独占 |

实现自定义同步组件时，将会调用同步器提供的模板方法，这些（部分）模板方法与描述如表所示:

