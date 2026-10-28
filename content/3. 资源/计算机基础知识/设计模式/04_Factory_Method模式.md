---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## Factory Method模式

在Template Method模式中，我们在父类中规定处理的流程，在子类中实现具体的处理。如果我们将该模式用于生成实例，他就演变成本章中我们要学习的Factory Method模式。

Factory有“工厂”的意思。用Template Method模式来构建生成实力的工厂，这就是Factory Method模式。

在Factory Method模式中，**父类决定实例生成方式，但并不决定所需要生成的具体类，具体的处理全部交给子类负责。** 这样就可以**将生成实例的框架和实际负责生成实例的类解耦。**

## 例子

例子表明的是制作身份证的过程。

Product类和Factory类属于framework包（生成实例的框架）。这两个类组成了生成实例的框架。

IDCard类和IDCardFactory类负责实际的加工处理，他们属于idcard包（加工处理）。

Main类是用于测试程序行为的类。

| 包        | 名字          | 说明                                        |
| --------- | ------------- | ------------------------------------------- |
| framework | Product       | 只定义抽象方法use的抽象类                   |
| framework | Factory       | 实现create方法的抽象类                      |
| idcard    | IDCard        | 实现了use方法的类                           |
| idcard    | IDCardFactory | 实现了createProduct、registerProdut方法的类 |
| 无名      | Main          | 测试                                        |

![image-20221008160741895](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221008160741895-1665216463675-1.png)

product是抽象类，用来表示“产品”

```java
package framework;

public abstract class Product {
    public abstract void use();
}
```

Factory使用了Template Method模式。该类还声明了用于“生成产品”的createProduct抽象方法和用于“注册产品”的registerProduct抽象方法。“生成产品”和“注册产品”的具体处理则被交给了Factory类的子类负责。

在这个框架中，我们定义了工厂是用来“调用crate方法生成Product实例”的。而create方法实现是先调用createProduct生成产品，接着调用registerProduct注册产品。

具体的实现内容根据Factory Method模式使用的场景不同而不同。但是，只要是Factory Method模式，在生成实例是就一定会使用到Template Method模式。

```java
package framework;

public abstract class Factory {
    public final Product create(String owner) {
        Product p = createProduct(owner);
        registerProduct(p);
        return p;
    }

    protected abstract Product createProduct(String owner);
    protected abstract void registerProduct(Product product);
}
```

IDCard用来编写表示ID卡的类。

```java
public class IDCard extends Product {
    private String owner;

    // 不使用public原因：
    // 在idcard包外不能通过new来得出IDCard的实例。这样就可以墙皮外部必须通过IDCardFactory
    // 来生成IDCard的实例。
    IDCard(String owner) {
        System.out.println("制作" + owner + "的ID卡");
        this.owner = owner;
    }

    @Override
    public void use() {
        System.out.println("使用" + owner + "的ID卡");
    }

    public String getOwner() {
        return owner;
    }

}
```

IDCaredFactory实现了createProduct方法和registerProduct方法。

```java
public class IDCaredFactory extends Factory {
    private List<String> owners = new ArrayList<>();
    @Override
    protected Product createProduct(String owner) {
        return new IDCard(owner);
    }

    @Override
    protected void registerProduct(Product product) {
        owners.add(((IDCard)product).getOwner());
    }

    public List getOwners() {
        return owners;
    }
}
```

Main用于测试

```java
public class Main {
    public static void main(String[] args) {
        Factory factory = new IDCaredFactory();
        Product card1 = factory.create("小明");
        Product card2 = factory.create("小红");
        Product card3 = factory.create("小刚");
        card1.use();
        card2.use();
        card3.use();
    }
}
```

## 登场角色

在Factory Method模式中有以下角色登场，其中，父类（框架）这一方的Creator角色和Product角色的关系与子类（具体加工）这一方的ConcreteCreator角色和ConcreteProduct角色的关系是平行的。

![image-20221008163028017](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221008163028017-1665217829093-3.png)

- Product（产品）

  属于框架这一方，是一个抽象类。定义了在Factory Method模式中生成的那些实例所持有的API，但是具体的处理由子类ConcreteProduct角色决定。

- Creator（创建者）

  属于框架的一方，负责生成Product角色的抽象类，但具体的处理则由子类ConcreteCreator角色决定。

  Creator角色对于实际负责生成实例的ConcreteCreator角色一无所知，唯一知道的是，只要调用Product角色和生成的实例的方法就可以生成Product的实例。**不用new关键字生成实例，而是通过调用生成实例的专用方法来生成实例，这样就可以防止父类与其他具体类耦合。**

- ConcreteProduct（具体的产品）

  属于具体加工这一方，决定不管了具体的产品。

- ConcreteCreator（具体的创建者）

  属于具体加工的一方，负责生成具体的产品。

## 总结

### 框架与具体加工

总结一句：框架与具体加工没有任何依赖，换一个具体加工我们仍然可以实现

### 生成实例

- 指定为抽象方法
- 为其实现默认处理
- 在其中抛出异常

## 相关设计模式

- [[03_Template_Method模式]]
- [[05_Singleton模式]]
- [[11_Composite模式]]
- [[01_iterator模式]]

在设计模式中，多个类和接口扮演各自的角色，互相协作工作，这种称为协作关系。
