---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
## 介绍

在并发编程中，有时需要使用线程安全的队列。实现一个线程安全的队列有两种方式：

1. 使用阻塞算法，队列可以用一把锁（入队和出队用同一把锁）或两个锁（入队和出队不同的锁）等方式来实现。
2. 使用非阻塞算法。可以采用循环CAS的方式来实现。

ConcurrentQueue是一个基于链连接节点的无界线程安全队列，采用非阻塞算法，**先进先出**，采用了 **“wait-free”(即CAS)** 来实现。

## 结构

结构如下：

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230214164909.png)

<center>ConcurrentLinkedQueue的类图</center>

从类图中得知，ConcurrentLinkedQueue有head节点与tail节点组成，每个节点（Node）有节点元素（item）和指向下一个节点（next）的引用组成，节点之间通过next串联起来，组成了一个链表，默认情况下**head为空，tail指向head。**

```java
private transient volatile Node<E> tail = head;
```

## 入队

### 过程

入队列就是将入队节点添加到队列的尾部，举个例子：

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230214165629.png)

<center>队列添加元素的快照图</center>

图中的添加节点的过程如下。

- 添加元素1。队列更新head节点的next节点为元素1节点。又因为tail节点默认情况下等于head节点，所以它们的next节点都指向元素1节点。
- 添加元素2。队列首先设置元素1节点的next节点为元素2节点，然后更新tail节点指向元素2节点。
- 添加元素3，设置tail节点的next节点为元素3节点。
- 添加元素4，设置元素3的next节点为元素4节点，然后将tail节点指向元素4节点。

从上面可以看出一个很重要的结论：

<font color="red" size=6>tail节点不总是尾节点！如果tail节点的next节点不为空，则将入队节点设置成tail节点，如果tail节点的next节点为空，则将入队节点设置成
tail的next节点。</font>

上面展示的是单线程插入，如果是多个线程进行插入，则可能出现下面几种情况：

	1. 一个线程进行插入，没有其他干扰；
	1. 一个线程进行插入，另一个线程也进行插入，但是当前线程先插入结束；
	1. 一个线程正在插入，必须线获取尾节点，然后设置尾节点的下一个节点为入队节点；此时另一个线程也进行插入且成功，那么当前线程就需要暂停，重新获得尾节点。

从源码层面分析，如下：

```java
public boolean offer(E e) {
        if (e == null) throw new NullPointerException();
        // 入队前，创建一个入队节点
        Node<E> n = new Node<E>(e);
        retry:
        // 死循环，入队不成功反复入队。
        for (;;) {
            // 创建一个指向tail节点的引用
            Node<E> t = tail;
            // p用来表示队列的尾节点，默认情况下等于tail节点。
            Node<E> p = t;
            for (int hops = 0; ; hops++) {
            // 获得p节点的下一个节点。
                Node<E> next = succ(p);
    // next节点不为空，说明p不是尾节点，需要更新p后在将它指向next节点
                if (next != null) {
                    // 循环了两次及其以上，并且当前节点还是不等于尾节点
                    if (hops > HOPS && t != tail)
                        continue retry; 
                    p = next;
                }
                // 如果p是尾节点，则设置p节点的next节点为入队节点。
                else if (p.casNext(null, n)) {
                    /*如果tail节点有大于等于1个next节点，则将入队节点设置成tail节点，
                      更新失败了也没关系，因为失败了表示有其他线程成功更新了tail节点*/
if (hops >= HOPS)
                        casTail(t, n); // 更新tail节点，允许失败
                    return true;
                }
                // p有next节点,表示p的next节点是尾节点，则重新设置p节点
                else {
                    p = succ(p);
                }
            }
        }
    }
```

从源代码角度来看，整个入队过程主要做两件事情：

1. 定位出尾节点；
2. 使用CAS算法将入队节点设置成尾节点的next节点，如不成功则重试。

### 定位尾节点

已知tail不一定总是尾节点，所以每次入队都必须通过tail节点找到尾节点，则有下面几种情况：

1. tail是尾节点；
2. tail不是尾节点。

上面代码中循环体中的第一个if就是判断tail是否有next节点，有则表示next节点可能是尾节点。获取tail节点的next节点需要注意的是p节点等于p的next节点的情况，只有一种可能就是p节点和p的next节点都等于空，表示这个队列刚初始化，正准备添加节点，所以需要返回head节点。获取p节点的next节点代码如下。

```java
final Node<E> succ(Node<E> p) {
    Node<E> next = p.getNext();
    return (p == next)  head : next;
}
```

### 设置入队节点为尾节点

p.casNext（null，n）方法用于**将入队节点设置为当前队列尾节点的next节点**，

- 如果p是null，表示p是当前队列的尾节点，
- 如果不为null，表示有其他线程更新了尾节点，则需要重新获取当前队列的尾节点。

### HOPS的设计意图

使用hops的设计意图有以下几点：

<font color="red" size=6>通过hops变量来控制并减少tail节点的更新频率，并不是每次节点入队后都将tail节点更新为尾节点，而是当tail节点和尾节点的举例大于等于常量HOPS的值（默认为1）时才更新tail节点。</font>

存在问题：

虽然可以减少CAS更新tail的频率，但是距离过长，每次增加一个节点的时间也就越长，因为循环体需要多循环一次来定位出尾节点，但是这样仍然能提高入队的效率，因为从本质上来看它通过增加对volatile变量的读操作来减少对volatile变量的写操作，而对volatile变量的写操作开销要远远大于读操作，所以入队效率会有所提升。

在java8中，这里的源码进行了更改：

```java
public boolean offer(E e) {
        checkNotNull(e);
        // 入队前，创建一个入队节点
        final Node<E> newNode = new Node<E>(e);

        //创建一个指向tail节点的引用
        // p用来表示队列的尾节点，默认情况下等于tail节点
        for (Node<E> t = tail, p = t;;) {
            //获得p节点的小一个节点。
            Node<E> q = p.next;
            // q节点为空吗，代表p已经时最后一个节点
            if (q == null) {
                // p is last node
                // 如果p是尾节点，则是指p节点的next节点为入队节点。
                if (p.casNext(null, newNode)) {
                    // Successful CAS is the linearization point
                    // for e to become an element of this queue,
                    // and for newNode to become "live".
                    // 如果tail节点有大于等于1个next节点，则将入队节点设置成tail节点，
                    // 更新失败了也没关系，因为失败了表示有其他线程成功更新tail节点
                    if (p != t) // hop two nodes at a time
                        //更新tail节点，允许失败
                        casTail(t, newNode);  // Failure is OK.
                    return true;
                }
                // Lost CAS race to another thread; re-read next
            }
            else if (p == q)
                // We have fallen off list.  If tail is unchanged, it
                // will also be off-list, in which case we need to
                // jump to head, from which all live nodes are always
                // reachable.  Else the new tail is a better bet.
                p = (t != (t = tail)) ? t : head;
            else
                // Check for tail updates after two hops.
                p = (p != t && t != (t = tail)) ? t : q;
        }
    }
```

从上面可以看出，不再使用hops，而是如注释所说：

>  我们已经掉出了列表。 如果尾部没有变化，它也会脱离列表，在这种情况下，我们需要跳到头部，从那里所有的活节点都可以到达。 否则，新的尾部是一个更好的选择。

注意：入队方法永远返回true，所以不要通过返回值判断入队是否成功。

## 出队

出队的示意图如下：

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230214174207.png)

<center>队列出节点快照图</center>

并不是每次出队时都更新head节点：

- 当head节点里有元素时，直接弹出head节点里的元素，而不会更新head节点。
- 只有当head节点里没有元素时，出队操作才会更新head节点。这种做法也是通过hops变量来减少使用CAS更新head节点的消耗，从而提高出队效率。

让我们再通过源码来深入分析下出队过程。

```java
public E poll() {
        Node<E> h = head;
// p表示头节点，需要出队的节点
        Node<E> p = h;
        for (int hops = 0;; hops++) {
                // 获取p节点的元素
                E item = p.getItem();
                // 如果p节点的元素不为空，使用CAS设置p节点引用的元素为null,
                // 如果成功则返回p节点的元素。
                if (item != null && p.casItem(item, null)) {
                        if (hops >= HOPS) {
                                // 将p节点下一个节点设置成head节点
                                Node<E> q = p.getNext();
                                updateHead(h, (q != null)  q : p);
                        }
                        return item;
                }
                // 如果头节点的元素为空或头节点发生了变化，这说明头节点已经被另外
                // 一个线程修改了。那么获取p节点的下一个节点
                Node<E> next = succ(p);
                // 如果p的下一个节点也为空，说明这个队列已经空了
                if (next == null) {
    // 更新头节点。
                        updateHead(h, p);
                        break;
                }
                // 如果下一个元素不为空，则将头节点的下一个节点设置成头节点
                p = next;
        }
        return null;
}
```

首先获取头节点的元素，然后判断头节点元素是否为空，如果为空，表示另外一个线程已经进行了一次出队操作将该节点的元素取走，如果不为空，则使用CAS的方式将头节点的引用设置成null，如果CAS成功，则直接返回头节点的元素，如果不成功，表示另外一个线程已经进行了一次出队操作更新了head节点，导致元素发生了变化，需要重新获取头节点。