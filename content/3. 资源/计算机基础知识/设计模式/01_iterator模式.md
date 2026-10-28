---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
## 从for循环说起

在java种我们要遍历arr数组时，可以使用以下语句：

```java
for (int i = 0; i < arr.length; i++) {
    System.out.println(arr[i]);
}
```

在这里面，我们可以根据arr的次序将数组进行输出，将这个过程抽象出来的设计模式，就是iterator模式（迭代器模式）。

## 一个简单例子

实现目的：将书(book)放入到书架(BookSelf)上面，然后将书的名字显示的输出出来。

![image-20221005104459594](./assets/image-20221005104459594-1664937901259-1.png)

各个类的含义如下：

| 名字              | 说明             |
| ----------------- | ---------------- |
| Aggregate         | 表示集合的接口   |
| Iterator          | 遍历集合的接口   |
| Book              | 表示书的类       |
| BookShelf         | 表示书架的类     |
| BookShelfIterator | 遍历书架的类     |
| Main              | 测试程序行为的类 |

```java
public interface Aggregate {
    public abstract Iterator iterator();
}
```

在Aggregate接口中声明的方法只有一个-iterator方法。该方法会生成一个用于遍历集合的迭代器。

想要遍历集合中的元素是，可以调用iterator方法来生成一个实现了Iterator接口的类的实例。

接下来看Iterator接口

```java
public interface Iterator {
    public abstract boolean hasNext();
    public abstract Object next();
}
```

Iterator接口相当于for循环遍历，在这里面，我们定义了两个方法：

1. hasNext()，判断是否还有下个元素。当来到最后一个元素时，返回false
2. next()，将当前元素进行返回，同时“隐形“的传递到下一个元素。（之所以说隐含，是因为在这里面我们并不知道元素的类型，只知道方法名，想要知道如何进行传递，只有等方法中进行具体的传递才能实现）

接下来看实体类-Book类

```java
public class Book {
    private String name;
    public Book(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

这个方法很简单，使用具体的Book类，将书的名字进行赋值，注意，将name设置成私有变量是为了防止外部不小心修改了该部分的值。

接下来看书架类-BookSelf类

```java
public class BookSelf implements Aggregate {
    private ArrayList<Book> books;

    public BookSelf() {
        books = new ArrayList<>();
    }

    public Book getBookAt(int index) {
        return books.get(index);
    }

    public void appendBook(Book book) {
        books.add(book);
    }

    public int getLength() {
        return books.size();
    }

    //iterator方法会生成并返回BookShelfIterator类的实例作为BookShelf类对应的Iterator。当不想要遍历书架时，就会调用到这个方法
    @Override
    public Iterator iterator() {
        return new BookSelfIterator(this);
    }
}
```

该类的作用是处理集合，因此在进行实现的时候，需要实现Aggregate接口，同时，只要我们实现了Aggregate接口，就需要实现里面的iterator方法。

接下来就是用于遍历书架的类-BookSelfIterator类

```java
public class BookSelfIterator implements Iterator {
    private BookSelf bookSelf;
    private int index;

    public BookSelfIterator(BookSelf bookSelf) {
        this.bookSelf = bookSelf;
        index = 0;
    }

    @Override
    public boolean hasNext() {
        if (index < bookSelf.getLength()) {
            return true;
        } else {
            return false;
        }
    }

    @Override
    public Object next() {
        Book book = bookSelf.getBookAt(index);
        index++;
        return book;
    }
}
```

因为BookShelfIterator类需要发挥Iterator的作用，所以它就要实现了Iterator接口。

bookShelf字段表示BookShelfIterator所需要遍历的书架。index字段表示迭代器当前所指向的书的下标。

构造函数会将接收到的BookShelf的实力保存在bookShelf字段中，并将index初始化为0。

hasNext方法是Iterator接口中所声明的方法。该方法将会判断属甲种是否还有书，有为true，没有为false

next方法会返回迭代器当前所指向的书（Book的实例），并让迭代器指向下一本书。

如此，就能实现如同for循环一样的遍历。

最后，看测试类-Main类

```java
public class Main {
    public static void main(String[] args) {
        BookSelf bookSelf = new BookSelf();
        bookSelf.appendBook(new Book("A"));
        bookSelf.appendBook(new Book("B"));
        bookSelf.appendBook(new Book("C"));
        bookSelf.appendBook(new Book("D"));
        Iterator it = bookSelf.iterator();
        while (it.hasNext()) {
            Book book = (Book)it.next();
            System.out.println(book.getName());
        }

    }
}
```

## iterator中登场的角色

- iterator（迭代器）

  定义了逐个遍历元素的接口（API）。在示例程序中，由Iterator接口扮演了这个角色，它定义了hasNext和next两个方法，其中，hashNext方法用于判断是否存在下一个元素，next方法则用于获取该元素。

- ConcreteIterator（具体的迭代器）

  实现Iterator角色定义的接口，再是程序中，由BookShelfIterator类扮演这个角色。该角色中半酣了遍历集合所必需的信息。在示例程序中，BookShelf类的实例保存在bookShelf字段中，被指向的书的下表保存在index字段中。

- Aggregate（集合）

  定义了创建Iterator觉得接口，这个API是一个方法，创建出“按顺序访问保存在我内部元素的人”。示例程序中，由Aggregate接口扮演这个角色，里面定义了iterator方法。

- ConcreteAggregate（具体的集合）

  定义了Aggregate角色定义的接口。会创建出具体的iterator角色，也就是BookSelfIterator类，在示例程序中，由BookSelfIterator实现这个角色，它实现了iterator这个方法。

![image-20221005164052223](./assets/image-20221005164052223-1664959253636-1.png)

## 要点总结

### 代码的复用性

为什么要存在iterator模式？使用循环不是也可以实现相应的功能吗？

一个很重要的原因是：**iterator模式可以将实现与遍历进行分离**，具体如下代码：

```java
while (it.hasNext()) {
    Book book = (Book)it.next();
    System.out.println(book.getName());
}
```

在这里，我们并没有使用任何BookSelf的方法，只是依赖与iterator方法中hashNext()与next()，因此，当我们改变BookShelf的表示方法（比如将数组改成ArrayList进行表示），**只要BookShelf的iterator方法能够正确的返回Iterator实例**，即使循环不修改，也能够进行正常的工作。

这种特性我们称为**可复用性**，所谓的“可复用”，就是指将类实现为“组件”，**当一个组件发生改变时，不需要对其他的组件行修改或是只需要很小的修改就可以应对。**

### 难以理解抽象类和接口

如果我们只是使用具体的类来解决具体的问题（如我们这里不使用Aggregate接口与Iterator接口，直接实现ConcreteAggregate角色与ConcreteIterator角色），很容易导致类之间的强耦合买这些类也很暗以作为组件被再次利用，为了**弱化类之间的耦合**，促进类更加容易作为组件再次利用，需要引进抽象类和接口。

**不要只是用具体类来编程，要优先使用抽象类和接口编程**。

### 接口对应

例子中，BookShelfIterator类定义为BookShelf类的ConcreteIterator角色的。BookShelfIterator类知道BookShelf是如何具体实现的。也正是因为如此，才能调用获取下一本书的getBookAt的方法。也就是说，如果BookShelf的实现发生了改变，即getBookAt方法这个接口发生变化时，我们必须修改BookShelfiterator类。

正如Aggregate和Iterator这两个接口是对应的一样，ContereteAggregate和ConcreteIterator这两个也是对应的。

### 下一个与最后一个

next是返回当前元素，并指向下一个元素。

hashNext是确认接下来是否可以调用next方法。

### 多个Iterator

”**将遍历功能置于Aggregate角色之外**“是Iterator模式的一个特征。根据这个特征，可以针对一个ConcreteAggregate角色编写多个ConcreteIterator角色

### 多种迭代器

遍历方法可以有多种多样：

- 从前向后
- 从后向前
- 跳跃式遍历

根据不同的要求，可以实现不同的遍历

### 不需要删除

java由自动回收机制，因此我们并不需要实现deleteIterator方法。

### 相关设计模式

- [[13_Visitor模式]]
- [[11_Composite模式]]
- [[04_Factory_Method模式]]





