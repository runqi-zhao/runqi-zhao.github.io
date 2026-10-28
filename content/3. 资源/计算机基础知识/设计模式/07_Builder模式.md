---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## Builder模式

在建造大楼时，需要先打牢地基，搭建框架，然后自下而上地一层已成盖起来，通常，在建造这种具有复杂结构的物体时，是很难一气呵成的，我们需要首先建造组成这个物体的各个部分，然后分阶段将他们组装起来。

用于组装具有复杂结构的实例，为Builder模式

## 例子

用Builder编写“文档”的程序。这里编写出的文档具有以下结构

- 含有一个标题
- 含有几个字符串
- 含有条目项目

Builder类种定义了决定文档结构的方法，然后Director类使用该方法编写一个具体的文档。

Builder是抽象类，没有任何的实际处理，仅仅是声明了抽象方法。Builder类的子类决定了用来编写文档的具体处理

- TextBuilder类：使用纯文本（欧通字符串）编写文档
- HTMLBuilder类：使用HTML编写文档

Director使用TextBuilder类可以编写纯文本文档，使用HTMLBuilder可以编写HTML文档。

| 名字        | 说明                                 |
| ----------- | ------------------------------------ |
| Builder     | 定义了决定文档结构的方法的抽象类     |
| Director    | 编写1个文档的类                      |
| TextBuilder | 使用纯文本（普通字符串）编写文档的类 |
| HTMLBuilder | 使用HTML编写文档的类                 |
| Main        | 测试程序行为的类                     |

![image-20221011110227998](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221011110227998-1665457349213-1.png)

Builder是抽象类，其中四个方法是写标题、字符串、条目

```java
public abstract class Builder {
    public abstract void makeTitle(String title);
    public abstract void makeString(String str);
    public abstract void makeItems(String[] items);
    public abstract void close();
}
```

Director是使用Builder类种声明的放啊来编写的文档。

Director类的构造函数的参数是Builder类型的。实际上不会讲Builder类的实例作为参数传递给Director类。这是他因为Builder类是抽象类，是无法生成其实例的。实际上传递给Director类的是Builder类的子类的实例。

construct方法是编写文档的方法。调用这个方法就可以编写文档。

```java
public class Director {
    public Builder builder;
    public Director (Builder builder) {
        this.builder = builder;
    }
    public void construct() {
        builder.makeTitle("Greeting");
        builder.makeString("从早上至下午");
        builder.makeItems(new String[]{
                "早上好。",
                "下午好。",
        });
        builder.makeString("晚上");
        builder.makeItems(new String[]{
                "晚上好。",
                "晚安。",
                "再见。",
        });
        builder.close();
    }
}
```

TextBuilder为纯文本编写类

```java
public class TextBuilder extends Builder {
    //这里为什么使用StringBuffer？不适用String，很重要一个知识点，请看我的博客：
    private StringBuffer buffer = new StringBuffer();
    @Override
    public void makeTitle(String title) {
        buffer.append("=======================\n");
        buffer.append("《"+ title + "》\n");
        buffer.append("\n");
    }

    @Override
    public void makeString(String str) {
        buffer.append("🤔" + str + "\n");
        buffer.append("\n");
    }

    @Override
    public void makeItems(String[] items) {
        for (int i = 0; i < items.length; i++) {
            buffer.append("😒" + items[i] + "\n");
        }
        buffer.append("\n");
    }

    @Override
    public void close() {
        buffer.append("=======================\n");
    }

    public String getResult() {
        return buffer.toString();
    }
}
```

HtmlBuiler显示html页面

```java
public class HtmlBuilder extends Builder {
    private String filename;
    private PrintWriter writer;
    @Override
    public void makeTitle(String title) {
        filename = title + ".html";
        try {
            writer = new PrintWriter(new FileWriter(filename));
        } catch (IOException e) {
            e.printStackTrace();
        }
        writer.println("<html><head><title>" + title + "</title></head><body>");
        writer.println("<h1>" + title + "</h1>");
    }

    @Override
    public void makeString(String str) {
        writer.println("<p>" + str + "</p>");
    }

    @Override
    public void makeItems(String[] items) {
        writer.println("<ul>");
        for (int i = 0; i < items.length; i++) {
            writer.println("<li>" + items[i] + "</li>");
        }
        writer.println("</ul>");
    }

    @Override
    public void close() {
        writer.println("</body></html>");
        writer.close();
    }

    public String getResult() {
        return filename;
    }
}
```

最后使用Main方法进行测试

Director角色并不关心使用的是TextBuilder还是HtmlBuilder，它只是用Builder

```java
public class Main {
    public static void main(String[] args) {
        if (args.length != 1) {
            usage();
            System.exit(0);
        }
        if (args[0].equals("plain")) {
            TextBuilder textBuilder = new TextBuilder();
            Director director = new Director(textBuilder);
            director.construct();
            String result = textBuilder.getResult();
            System.out.println(result);
        } else if (args[0].equals("html")) {
            HtmlBuilder htmlBuilder = new HtmlBuilder();
            Director director = new Director(htmlBuilder);
            director.construct();
            String filename = htmlBuilder.getResult();
            System.out.println(filename + "文档编写完成");
        } else {
            usage();
            System.exit(0);
        }
    }

    public static void usage() {
        System.out.println("Usage: java Main plain   编写纯文本文档");
        System.out.println("Usage: java Main html    编写HTML文档");
    }
}
```

## 出场角色

- Builder（建造者）

  Builder角色负责定义用于生成实例的接口。Builder角色准备了用于生成实例的方法。

- ConcreteBuilder（具体的建造者）

  负责实现Builder角色的接口的类（API）。定义了在生成实例时实际被调用的方法。此外，ConcreteBuilder角色还定义了获得最终生成结果的方法。

- Director（监工）

  负责使用Builder角色的接口（API）来生成实例。并不依赖于ConcreteBuilder角色。为了确保ConcreteBuilder角色如何被定义的，Director角色都能正常工作，只调用再Builder角色中被定义的方法。

- Client（使用者）

  使用了Builder模式（Builder模式中不包含Client角色）。在示例程序中，由Main类扮演此角色。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221011112034660-1665458435669-3.png)

![image-20221011112101926](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221011112101926-1665458462849-5.png)

## 总结

### 谁知道什么

在面向对象编程中，“谁知道什么”是非常重要的。也就是说，我们需要在编程时注意哪个类可以使用哪个方法以及使用这个方法到底好不好。

再次回忆以下实例过程。

Main类并不知道（没有调用）Builder类，他只是调用了Director类的construct方法。这样，Direcot类就开始工作，并完成文档的编写。

另一方面，Director类知道Builder类，他调用Builder类的方法来编写文档，但是它并不知道“真正”使用的是哪个类。

Director类不知道自己使用的究竟是Builder类的哪个子类也好，这是因为“**只有不知道的子类才能替换**”/

这是因为不知道才能替换，正是因为可以替换，组件才具有高价值。作为设计人员，必须注重这种“**可替换性**”。

### 设计时能够决定和不能决定的事

在Builder类中，需要声明编辑文档（实现功能）所必需的所有方法。Director类中使用的方法都是Builder类提供的。因此，在Builder类中应当定义哪些方法是非常重要的。

而且，Builder类还必须能够应对将来子类可能增加的需求。在示例程序中，我们只编写了支持纯文本文档的子类和支持HTML文件的子类。但是将来可能还会希望能够编写其他形式（例如XXX形式)的文档。那时候，到底能不能编写出支持XXXX形式的XXXXBui1der类呢？应该不
需要新的方法吧？

虽然类的设计者并不是神仙，他们无法准确地预测到将来可能发生的变化。但是，我们还是有必要让设计出的类能够尽可能灵活地应对近期可能发生的变化。

### 代码的可读性

在编程时，虽然有时需要从零开始编写代码，但更多时候我们都是在现有代码的基础上进行增加和修改。这时，我们需要先阅读现有代码。不过，只是阅读抽象类的代码是无法获取很多信息的（虽然可以从方法名中获得线索)。

让我们再回顾一下示例程序。即使理解了Builder抽象类，也无法理解程序整体。至少必须在阅读了Director的代码后才能理解Builder类的使用方法(Builder类的方法的调用方法)然后再去看看TextBuilder类和HTMLBui1der类的代码，就可以明白调用Bui1der类的方法后具体会进行什么样的处理。

如果没有理解各个类的角色就动手增加和修改代码，在判断到底应该修改哪个类时，就会很容易出错。例如，如果修改Builder类，那么就会对Director类中调用Builder类方法的地方和Builder类的子类产生影响。或是如果不小心修改了Director类，在其内部调用了TextBuilder类的特有的方法，则会导致其失去作为可复用组件的独立性，而且当将子类替换为HTMLBuilder时，程序可能会无法正常工作。

## 相关设计模式

- [[03_Template_Method模式]]

  在Builder模式中，Director角色控制Builder角色。

  在Template Method模式中，父类控制子类。

- [[11_Composite模式]]

  有些情况下Builder模式生成的实例构成了Composite模式。

- [[08_Abstract_Fatctory模式]]

  Builder模式和Abstract Factory模式都用于生成复杂的实例。

- [[15_Facade模式]]

  在Builder模式中，Director角色通过组合Builder角色中的复杂方法向外部提供可以简单生成

  实例的接口(API)(相当于示例程序中的construct方法)。

  Facade模式中的Facade角色则是通过组合内部模块向外部提供可以简单调用的接口(API)。

