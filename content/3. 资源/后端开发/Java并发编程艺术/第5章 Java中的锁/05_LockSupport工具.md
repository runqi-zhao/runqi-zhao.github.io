---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
回顾[[02_队列同步器]]，当需要阻塞或唤醒一个线程的时候，都会使用LockSupport工具类来完成相应工作。LockSupport定义了一组的公共静态方法，这些方法提供了最基本的线程阻塞和唤醒功能，而LockSupport也成为构建同步组件的基础工具。

LockSupport定义了一组以park开头的方法用来阻塞当前线程，以及unpark(Thread thread)方法来唤醒一个被阻塞的线程。Park有停车的意思，假设线程为车辆，那么park方法代表着停车，而unpark方法则是指车辆启动离开，这些方法以及描述如表所示。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230503105646.png)

在Java 6中，LockSupport增加了park(Object blocker)、parkNanos(Object blocker,long nanos)和parkUntil(Object blocker,long deadline)3个方法，用于实现阻塞当前线程的功能，其中参数blocker是用来标识当前线程在等待的对象（以下称为阻塞对象），该对象主要用于问题排查和系统监控。

下面的示例中，将对比parkNanos(long nanos)方法和parkNanos(Object blocker,long nanos)方法来展示阻塞对象blocker的用处，代码片段和线程dump（部分）如表5-11所示。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230503105742.png)

从表5-11的线程dump结果可以看出，代码片段的内容都是阻塞当前线程10秒，但从线程dump结果可以看出，有阻塞对象的parkNanos方法能够传递给开发人员更多的现场信息。这是由于在Java 5之前，当线程阻塞（使用synchronized关键字）在一个对象上时，通过线程dump能够查看到该线程的阻塞对象，方便问题定位，而Java 5推出的Lock等并发工具时却遗漏了这一点，致使在线程dump时无法提供阻塞对象的信息。因此，在Java 6中，LockSupport新增了上述3个含有阻塞对象的park方法，用以替代原有的park方法。



