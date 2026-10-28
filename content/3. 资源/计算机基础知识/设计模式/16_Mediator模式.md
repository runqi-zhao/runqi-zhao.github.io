---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## Mediator模式

请大家想象一下一个乱糟糟的开发小组的工作状态。小组中的10个成员虽然一起协同工作，但是意见难以统一，总是互相指挥，导致工作始进度始终滞后。不仅如此，他们还都十分在意编码细节，经常为此争执不下。这时，我们就需要一个中立的仲裁者站出来说：“各位，请大家将情况报告给我，我来负责仲裁。我会从团队整体出发进行考虑，然后下达指示。但我不会评价大家的工作细节。”这样，当出现争执时大家就会找仲裁者进行商量，仲裁者会负责统一大家的意见。

最后，整个团队的交流过程就变为了组员向仲裁者报告，仲裁者向组员下达指示。组员之间不再相互询问和相互指示。

在本章中，我们将要学Mediator模式。Mediator的意思是“仲裁者”“中介者”。一方面，当发生麻烦事情的时候，通知仲裁者；当发生涉及全体组员的事情时，也通知仲裁者。当仲裁者下达指示时，组员会立即执行。团队组员之间不再互相沟通并私自做出决定，而是发生任何事情都向仲裁者报告。另一方面，仲裁者站在整个团队的角度上对组员上报的事情做出决定。这就是Mediator模式。

在Mediator模式中，“仲裁者”被称为Mediator,各组员被称为Colleague。

## 例子

使用一个GUI程序，站是一个登录对话框，用户在输入正确的用户名和密码后可以登录。对话框的使用方法如下：

- 可以选择作为游客访问(Guest)或是作为用户登录
- 作为用户登录时，需要输入正确的用户名（Username）和密码（Password）
- 点击OK按钮时，登录，点击Cancel按钮后就退出程序

那么存在以下几种情况：

- 如果选择作为游客访问，那么禁用用户名输入框和密码输入框，使用户无法输入
- 如果选择作为用户登录，那么启用用户名输入框和密码输入框，使用户可以输入
- 如果在用户名输入框中一个字符都没有输入，那么禁用密码输入框，使用户无法输入密码
- 如果在用户名输人框中输入了至少一个字符，那么启用密码输入框，使用户可以输入密码（当然，如果选择作为游客访问，那么密码框依然是禁用状态)
- 只有当用户名输人框和密码输入框中都至少输入一个字符后，OK按钮才处于启用状态，可以被按下。用户名输入框或密码输人框中一个字符都没有被输入的时候，禁用OK按钮，使其不可被按下（当然，如果选择作为游客访问，那么OK按钮总是处于启用状态）
- Cancel按钮总是处于启用状态，任何时候都可以按下该按钮

**向上面这样要调整多个对象之间的关系是，就要使用到Medoator模式**。即不让各个对象之间互相通信，而是增加一个仲裁者通信。然后，**将控制显示的逻辑处理交给仲裁者负责**。

| 名字               | 说明                                                  |
| ------------------ | ----------------------------------------------------- |
| Mediator           | 定义”仲裁者“的接口(API)的接口                         |
| Colleague          | 定义”组员“的接口(API)的接口                           |
| ColleagueButton    | 表示按钮的类。它实现了Colleague接口                   |
| ColleagueTextFeild | 表示文本输入框的类。它实现了Colleague接口             |
| ColleagueCheckbox  | 表示勾选框（此处是单选按钮）类。它实现了Colleague接口 |
| LoginFrame         | 表示登录对话框的类。它实现了Mediator接口              |
| Main               | 测试程序行为的类                                      |

![image-20221020103228487](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221020103228487-1666233149561-1.png)

![image-20221020103241292](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221020103241292-1666233162289-3.png)

Mediator接口表示仲裁者接口。

createColleagues方法用于生成Mediator要管理的组员。

colleagueChanged方法会被各个Colleagues组员调用。

```java
public interface Mediator {
    public abstract void createColleagues();
    public abstract void colleagueChanged();
}
```

Colleagues表示向仲裁者进行报告的组员的接口。

```java
public interface Colleague {
    public abstract void setMediator(Mediator mediator);
    public abstract void setColleagueEnabled(boolean enabled);
}
```

ColleagueButton类实现了Colleague接口，与LoginFrame共同工作。

```java
public class ColleagueButton extends Button implements Colleague {
    private Mediator mediator;
    public ColleagueButton(String caption) {
        super(caption);
    }

    @Override
    public void setMediator(Mediator mediator) {
        this.mediator = mediator;
    }

    @Override
    public void setColleagueEnabled(boolean enabled) {
        setEnabled(enabled);
    }
}
```

ColleagueTextFeild类不仅实现了Colleagues接口，还是先了TextListener接口

```java
public class ColleagueTextField extends TextField implements TextListener, Colleague {
    private Mediator mediator;
    public ColleagueTextField(String text, int columns) {
        super(text, columns);
    }
    @Override
    public void setMediator(Mediator mediator) {
        this.mediator = mediator;
    }

    @Override
    public void setColleagueEnabled(boolean enabled) {
        setEnabled(enabled);
        setBackground(enabled ? Color.white : Color.lightGray);
    }

    @Override
    public void textValueChanged(TextEvent e) {
        mediator.colleagueChanged();
    }
}
```

ColleagueCheckbox类作为单选按钮使用

```java
public class ColleagueCheckbox extends Checkbox implements ItemListener, Colleague {
    private Mediator mediator;

    public ColleagueCheckbox(String caption, CheckboxGroup group, boolean state) {
        super(caption, group, state);
    }

    @Override
    public void setMediator(Mediator mediator) {
        this.mediator = mediator;
    }

    @Override
    public void setColleagueEnabled(boolean enabled) {
        setEnabled(enabled);
    }

    @Override
    public void itemStateChanged(ItemEvent e) {
        mediator.colleagueChanged();
    }
}
```

LoginFrame类是仲裁者类，会进行以下处理：

- 设置背景色
- 设置布局管理器
- 调用createColleagues方法生成Colleagues
- 配置Colleague
- 设置初始状态
- 显示

整个示例程序中最重要的方法当属LoginFrame类的colleagueChanged方法。该方法负责前面讲到过的“设置控件的启用/禁用的复杂逻辑处理”。请大家回忆一下之前学习过的ColleagueButton、ColleagueCheckbox、ColleagueTextField等各个类。这些类中虽然都有设置自身的启用/禁用状态的方法，但是并没有“具体什么情况下需要设置启用/禁用”的逻辑处理。它们都只是简单地调用仲裁者的colleagueChanged方法告知仲裁者“剩下的就拜托给你了”。也就是说，**所有最终的决定都是由仲裁者的colleagueChanged方法下达的**。

```java
public class LoginFrame extends Frame implements ActionListener, Mediator {
    private ColleagueCheckbox checkGuest;
    private ColleagueCheckbox checkLogin;
    private ColleagueTextField textUser;
    private ColleagueTextField textPass;
    private ColleagueButton buttonOK;
    private ColleagueButton buttonCancel;

    public LoginFrame(String title) {
        super(title);
        setBackground(Color.lightGray);
        setLayout(new GridLayout(4, 2));
        createColleagues();
        add(checkGuest);
        add(checkLogin);
        add(new Label("Username"));
        add(textUser);
        add(new Label("Password"));
        add(textPass);
        add(buttonOK);
        add(buttonCancel);
        colleagueChanged();;
        pack();;
        show();
    }

    @Override
    public void createColleagues() {
        CheckboxGroup group = new CheckboxGroup();
        checkGuest = new ColleagueCheckbox("Guest", group, true);
        checkLogin = new ColleagueCheckbox("Login", group, false);
        textUser = new ColleagueTextField("", 10);
        textPass = new ColleagueTextField("", 10);
        textPass.setEchoChar('*');
        buttonOK = new ColleagueButton("OK");
        buttonCancel = new ColleagueButton("Cancel");
        checkGuest.setMediator(this);
        checkLogin.setMediator(this);
        textUser.setMediator(this);
        textPass.setMediator(this);
        buttonOK.setMediator(this);
        buttonCancel.setMediator(this);
        checkGuest.addItemListener(checkGuest);
        checkLogin.addItemListener(checkLogin);
        textUser.addTextListener(textUser);
        textPass.addTextListener(textPass);
        buttonOK.addActionListener(this);
        buttonCancel.addActionListener(this);
    }

    @Override
    public void colleagueChanged() {
        if (checkGuest.getState()) {
            textUser.setColleagueEnabled(false);
            textPass.setColleagueEnabled(false);
            buttonOK.setColleagueEnabled(true);
        } else {
            textUser.setColleagueEnabled(true);
            userpassChanged();
        }
    }

    private void userpassChanged() {
        if (textUser.getText().length() > 0) {
            textPass.setColleagueEnabled(true);
            if (textPass.getText().length() > 0) {
                buttonOK.setColleagueEnabled(true);
            } else {
                buttonOK.setColleagueEnabled(false);
            }
        } else {
            textPass.setColleagueEnabled(false);
            buttonOK.setColleagueEnabled(false);
        }
    }


    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println(e.toString());
        System.exit(0);
    }
}
```

Main类进测试

```java
public class Main {
    public static void main(String[] args) {
        new LoginFrame("Mediator Sample.");
    }
}
```

## 登场角色

### Mediator(仲裁者、中介者)

负责定义与Colleague角色进行通信和做出决定的接口。

### ConcreteMediator(具体的仲裁者、中介者)

负责是u先Mediator角色的接口。

### Colleague(同事)

负责定义与Mediator角色及逆行通信的接口。

### ConcreteColleague(具体的同事)

负责实现Colleague角色的接口。

![image-20221020110816277](https://img-1312072469.cos.ap-nanjing.myqcloud.com//image-20221020110816277-1666235297268-5.png)

## 总结

### 当发生分散灾难时

示例程序中的LoginFrame类的colleagueChanged方法（代码清单l6-6)稍微有些复杂。如果发生需求变更，该方法中很容易发生Bug。不过这并不是什么问题。因为即使colleagueChanged方法中发生了Bug,由于其他地方并没有控制控件的启用7禁用状态的逻辑处理，
因此只要调试该方法就能很容易地找出Bug的原因。

请试想一下，如果这段逻辑分散在ColleagueButton类、co1 leagueTextField类和ColleagueCheckbox类中，那么无论是编写代码还是调试代码和修改代码，都会非常困难。通常情况下，面向对象编程可以帮助我们分散处理，避免处理过于集中，也就是说可以“分而治之”。但是在本章中的示例程序中，把处理分散在各个类中是不明智的。如果只是将应当分散的处理分散在各个类中，但是没有将应当集中的处理集中起来，那么这些分散的类最终只会导致灾难。

### 通信线路的增加

假设现在有A和B这2个实例，它们之间互相通信（相互之间调用方法），那么通信线路有两条，即A→B和A←-B。如果是有A、B和C这3个实例，那么就会有6条通信线路，即A→B、A←一B、B→C、B←-C、C→A和C-A。如果有4个实例，会有12条通信线路；5个实例就会有20条通信线路，而6个实例则会有30条通信线路。如果存在很多这样的互相通信的实例，那么程序结构会变得非常复杂。

可能会有读者认为，如果实例很少就不需要Mediator模式了。但是需要考虑到的是，即使最初实例很少，很可能随着需求变更实例数量会慢慢变多，迟早会暴露出问题。

### 哪些角色可以复用

ConcreteColleague角色可以复用，但ConcreteMediator角色很难复用。

例如，假设我们现在需要制作另外一个对话框。这时，我们可将扮演ConcreteColleague角色的ColleagueButton类、ColleagueTextField类和ColleagueCheckbox类用于新的对话框中。这是因为在ConcreteColleague角色中并没有任何依赖于特定对话框的代码。

在示例程序中，依赖于特定应用程序的部分都被封装在扮演ConcreteMediator角色的LoginFrame类中。依赖于特定应用程序就意味着难以复用。因此，LoginFrame类很难在其他对话框中被复用。

## 相关设计模式

- [[15_Facade模式]]
- [[17_Observer模式]]
