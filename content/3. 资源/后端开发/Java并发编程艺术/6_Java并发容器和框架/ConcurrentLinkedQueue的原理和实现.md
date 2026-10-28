---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
要想实现一个线程安全的队列有两种方式：

1. 使用阻塞算法。可以用一个锁（入队和出队用一把锁）或两个锁（入队和出队永不同的锁）等方式实现。
2. 使用非阻塞算法。通过过使用循环CAS的方式来实现。

## ConcurrentLinkedQueue的结构

ConcurrentLinkedQueue是一个基于连接节点的无界线程安全队列，使用先进先出的规则对节点进行排序。

ConcurrentLinkedQueue的结构类图如下：

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/1689817964081.jpg)

ConcurrentLinkedQueue由head节点和tail节点组成，每个节点（Node）由节点元素（item）和指向下一个节点（next）的引用组成，节点与节点之间就是通过这个next关联起来，从而组成一张链表结构的队列。默认情况下head节点存储的元素为空，tail节点等于head节点。

## 入队列

### 入队列的过程

将节点的内容加到队列的尾部。

过程如下图：

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/1689818077660.jpg)

多线程入队的过程中，tail指向的结点可能不是为节点，因此，多线程进行入队的时候，需要暂停入队操作，然后重新获取尾节点。

入队过程主要做了两件事情：

1. 定位出尾节点。
2. 使用CAS算法将入队节点设置成尾节点的next节点。

### 定位尾节点

定位尾节点的代码如下：

```java
final Node<E> succ(Node<E> p) {
	Node<E> next = p.getNext();
	return (p == next) head : next;
}
```

### 设置入队节点为尾节点

p.casNext（null，n）方法用于将入队节点设置为当前队列尾节点的next节点，如果p是null，表示p是当前队列的尾节点，如果不为null，表示有其他线程更新了尾节点，则需要重新获取当
前队列的尾节点。

### HOPS的设计意图

意图：使用hops变量来控制并减少tail系欸但的更新频率，并不是每次节点如对后都将tail节点更新为尾节点，而是当tail节点和尾节点的距离杜宇等于常量HOPS的值（默认等于1）是才更新tail节点，tail和尾节点的距离越长，使用CAS更新tail节点的次数越少，但是举例越长带来的负面效果就是每次入队时定位尾节点的时间的时间越长，因此循环体需要使用多循环此役来定位出尾节点，但是这样仍然可以提高入队效率。

<font color="red"> 简单一句话，HOPS的设计意图是为了提高入队时的效率</font>

## 出队列

并不是每次head节点都指向头节点，在继续宁出对时候，需要使用hops变量减少使用CAS更新的次数，提高出队效率。

对应的出队代码如下：

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
				updateHead(h, (q != null) q : p);
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

