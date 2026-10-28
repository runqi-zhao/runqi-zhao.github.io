---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
当程序更新一个变量时，如果多线程同时更新这个变量，可能得到期望之外的值，比如变量i=1，A线程更新i+1，B线程也更新i+1，经过两个线程操作之后可能i不等于3，而是等于2。因为A和B线程在更新变量i的时候拿到的i都是1，这就是线程不安全的更新操作，通常我们会使用synchronized来解决这个问题，synchronized会保证多线程不会同时更新变量i。

而Java从JDK 1.5开始提供了java.util.concurrent.atomic包（以下简称Atomic包），这个包中的原子操作类提供了一种用法简单、性能高效、线程安全地更新一个变量的方式。因为变量的类型有很多种，所以在Atomic包里一共提供了13个类，属于4种类型的原子更新方式，分别是原子更新基本类型、原子更新数组、原子更新引用和原子更新属性（字段）。

Atomic包里的类基本都是使用Unsafe实现的包装类。

## 原子更新基本类

使用原子的方式更新基本类型，Atomic包提供了以下3个类。

- AtomicBoolean：原子更新布尔类型。
- AtomicInteger：原子更新整型。
- AtomicLong：原子更新长整型。

以上3个类提供的方法几乎一模一样，所以本节仅以AtomicInteger为例进行讲解。

### AtomicInteger

 AtomicInteger的常用方法如下。

- int addAndGet(int ddelta)：以原子的方式将输入的数值与实例中值（AtomicInteger里面value）相加，并返回结果。
- boolean compareAndSet(int expect，int update)：如果输入的数值等于预期值，则以原子方式将该值设置为输入的值。
- int getAndIncrement()：以原子方式将当前值加1，注意，这里返回的是自增前的值。
- void lazySet(int newValue)：最终会设置成newValue，使用lazySet设置值后，可能导致其他线程在之后的一小段时间内还是可以读到旧的值。关于该方法的更多信息可以参考并发编程
  网翻译的一篇文章《AtomicLong.lazySet是如何工作的？》，文章地址是“http://ifeve.com/how-does-atomiclong-lazyset-work/”。
- int getAndSet(int newValue)：以原子方式设置为newValue的值，并返回旧值。

一段测试代码：

```java
import java.util.concurrent.atomic.AtomicInteger;
public class AtomicIntegerTest {
        static AtomicInteger ai = new AtomicInteger(1);
        public static void main(String[] args) {
                System.out.println(ai.getAndIncrement());
                System.out.println(ai.get());
        }
}
```

输出结果如下：

```
1
2
```

getAndIncrement是如何实现原子操作的,源代码如下：

```java
public final int getAndIncrement() {
        for (;;) {
            int current = get();
            int next = current + 1;
            if (compareAndSet(current, next))
                return current;
        }
}
public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

1. 先获取AtomicInteger里存储的数值；
2. AtomicInteger的当前数值进行加1操作；
3. 调用compareAndSet方法来进行原子更新操作，该方法先检查当前数值是否等于current：
   1. 等于意味着AtomicInteger的值没有被其他线程修改过，则将AtomicInteger的当前数值更新成next的值；
   2. 不等compareAndSet方法会返回false，程序会进入for循环重新进行compareAndSet操作。

### 其他3个

Atomic包提供了3种基本类型的原子更新，但是Java的基本类型里还有char、float和double等。那么问题来了，如何原子的更新其他的基本类型呢？Atomic包里的类基本都是使用Unsafe实现的。

```java
/**
 * 如果当前数值是expected，则原子的将Java变量更新成x
 * @return 如果更新成功则返回true
 */
public final native boolean compareAndSwapObject(Object o,
                                                 long offset,
                                                 Object expected,
                                                 Object x);
public final native boolean compareAndSwapInt(Object o, long offset,
                                              int expected,
                                              int x);
public final native boolean compareAndSwapLong(Object o, long offset,
                                               long expected,
                                               long x);
```

Unsafe只提供了3种CAS方法：

- compareAndSwapObject
- compareAndSwapInt
- compareAndSwapLong

再看AtomicBoolean源码，发现

1. 先把Boolean转换成整型，
2. 使用compareAndSwapInt进行CAS。

所以原子更新char、float和double变量也可以用类似的思路来实现。

## 原子更新数组

通过原子的方式更新数组里的某个元素，Atomic包提供了以下4个类。

- AtomicIntegerArray：原子更新整型数组里的元素。
- AtomicLongArray：原子更新长整型数组里的元素。
- AtomicReferenceArray：原子更新引用类型数组里的元素。
- AtomicIntegerArray类主要是提供原子的方式更新数组里的整型，其常用方法如下。
- int addAndGet（int i，int delta）：以原子方式将输入值与数组中索引i的元素相加。
- boolean compareAndSet（int i，int expect，int update）：如果当前值等于预期值，则以原子方式将数组位置i的元素设置成update值。

以上几个类提供的方法几乎一样，所以本节仅以AtomicIntegerArray为例进行讲解。

```java
public class AtomicIntegerArrayTest {
        static int[] value = new int[] { 1， 2 };
        static AtomicIntegerArray ai = new AtomicIntegerArray(value);
        public static void main(String[] args) {
                ai.getAndSet(0， 3);
                System.out.println(ai.get(0));
                System.out.println(value[0]);
        }
}
```

以下是输出的结果。

```
3
1
```

需要注意的是，<font color="red">数组value通过构造方法传递进去，然后AtomicIntegerArray会将当前数组复制一份，所以当AtomicIntegerArray对内部的数组元素进行修改时，不会影响传入的数组。</font>

## 原子更新引用类型

原子更新基本类型的AtomicInteger，只能更新一个变量，如果要原子更新多个变量，就需要使用这个原子更新引用类型提供的类。Atomic包提供了以下3个类。

- AtomicReference：原子更新引用类型。
- AtomicReferenceFieldUpdater：原子更新引用类型里的字段。
- AtomicMarkableReference：原子更新带有标记位的引用类型。可以原子更新一个布尔类型的标记位和引用类型。构造方法是AtomicMarkableReference(V initialRef，booleaninitialMark)。

以上几个类提供的方法几乎一样，所以本节仅以AtomicReference为例：

```java
public class AtomicReferenceTest {
        public static AtomicReference<user> atomicUserRef = new
            AtomicReference<user>();
        public static void main(String[] args) {
                User user = new User("conan"， 15);
                atomicUserRef.set(user);
                User updateUser = new User("Shinichi"， 17);
                atomicUserRef.compareAndSet(user， updateUser);
                System.out.println(atomicUserRef.get().getName());
                System.out.println(atomicUserRef.get().getOld());
        }
        static class User {
                private String name;
                private int old;
                public User(String name， int old) {
                        this.name = name;
                        this.old = old;
               }
                public String getName() {
                        return name;
                }
                public int getOld() {
                        return old;
                }
        }
}
```



1. 构建一个user对象；
2. 把user对象设置进AtomicReferenc中；
3. 调用compareAndSet方法进行原子更新操作，实现原理同AtomicInteger里的compareAndSet方法。

代码执行后输出结果如下。

```java
Shinichi
17
```

## 原子更新字段类型

如果需原子地更新某个类里的某个字段时，就需要使用原子更新字段类，Atomic包提供了以下3个类进行原子字段更新。

- AtomicIntegerFieldUpdater：原子更新整型的字段的更新器。
- AtomicLongFieldUpdater：原子更新长整型字段的更新器。
- AtomicStampedReference：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于原子的更新数据和数据的版本号，可以解决使用CAS进行原子更新时可能出现的ABA问题。

要想原子地更新字段类需要两步。

1. 第一步，因为原子更新字段类都是抽象类，每次使用的时候必须使用静态方法newUpdater()创建一个更新器，并且需要设置想要更新的类和属性。
2. 第二步，更新类的字段（属性）必须使用public volatile修饰符。

以上3个类提供的方法几乎一样，所以本节仅以AstomicIntegerFieldUpdater为例进行讲解，

```java
public class AtomicIntegerFieldUpdaterTest {
        // 创建原子更新器，并设置需要更新的对象类和对象的属性
        private static AtomicIntegerFieldUpdater<User> a = AtomicIntegerFieldUpdater.
        newUpdater(User.class， "old");
        public static void main(String[] args) {
                // 设置柯南的年龄是10岁
                User conan = new User("conan"， 10);
                // 柯南长了一岁，但是仍然会输出旧的年龄
                System.out.println(a.getAndIncrement(conan));
                // 输出柯南现在的年龄
                System.out.println(a.get(conan));
        }
        public static class User {
                private String name;
                public volatile int old;
                public User(String name， int old) {
                        this.name = name;
                        this.old = old;
                }
                public String getName() {
                        return name;
                }
                public int getOld() {
                        return old;
                }
        }
}
```

代码执行后输出如下。

```
10
11
```

## 总结

本章介绍了JDK中并发包里的13个原子操作类以及原子操作类的实现原理，读者需要熟悉这些类和使用场景，在适当的场合下使用它。