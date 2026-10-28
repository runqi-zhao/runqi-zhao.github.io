---
id: JSTL标签库
title: JSTL标签库
sidebar_label: JSTL标签库
description: JSTL标签库
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

JSTL 标签库全称是指JSP Standard Tag Library JSP 标准标签库。是一个不断完善的开放源代码的JSP 标
签库。

EL 表达式主要是为了替换jsp 中的表达式脚本，而标签库则是为了替换代码脚本。这样使得整个jsp 页面
变得更佳简洁。

JSTL 由五个不同功能的标签库组成。

| 功能范围         | URI                                    | 前缀 |
| ---------------- | -------------------------------------- | ---- |
| 核心标签库--重点 | http://java.sun.com/jsp/jstl/core      | c    |
| 格式化           | http://java.sun.com/jsp/jstl/fmt       | fmt  |
| 函数             | http://java.sun.com/jsp/jstl/functions | fn   |
| 数据库(不使用)   | http://java.sun.com/jsp/jstl/sql       | sql  |
| XML(不使用)      | http://java.sun.com/jsp/jstl/xml       | x    |

在jsp 标签库中使用taglib 指令引入标签库

```jsp
CORE 标签库
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
XML 标签库
<%@ taglib prefix="x" uri="http://java.sun.com/jsp/jstl/xml" %>
FMT 标签库
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
SQL 标签库
<%@ taglib prefix="sql" uri="http://java.sun.com/jsp/jstl/sql" %>
FUNCTIONS 标签库
<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions" %>
```

## JSTL标签库的使用步骤

1. 先导入jstl 标签库的jar 包。

   taglibs-standard-impl-1.2.1.jar
   taglibs-standard-spec-1.2.1.jar

2. 第二步，使用taglib 指令引入标签库。
   `<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>`

## core 核心库使用

### `<c:set />`（使用很少）

作用：set 标签可以往域中保存数据

```jsp
<%--
i.<c:set />
作用：set 标签可以往域中保存数据
域对象.setAttribute(key,value);
scope 属性设置保存到哪个域
page 表示PageContext 域（默认值）
request 表示Request 域
session 表示Session 域
application 表示ServletContext 域
var 属性设置key 是多少
value 属性设置值
--%>
保存之前：${ sessionScope.abc } <br>
<c:set scope="session" var="abc" value="abcValue"/>
保存之后：${ sessionScope.abc } <br>
```

### `<c:if />`

if标签用来做if判断。

```jsp
<%--
ii.<c:if />
if 标签用来做if 判断。
test 属性表示判断的条件（使用EL 表达式输出）
--%>
<c:if test="${ 12 == 12 }">
<h1>12 等于12</h1>
</c:if>
<c:if test="${ 12 != 12 }">
<h1>12 不等于12</h1>
</c:if>
```

### `<c:choose> <c:when> <c:otherwise>`标签

作用：多路判断。跟switch ... case .... default 非常接近

```jsp
<%--
iii.<c:choose> <c:when> <c:otherwise>标签
作用：多路判断。跟switch ... case .... default非常接近

choose标签开始选择判断
when标签表示每一种判断情况
    test属性表示当前这种判断情况的值
otherwise标签表示剩下的情况

<c:choose> <c:when> <c:otherwise>标签使用时需要注意的点：
    1、标签里不能使用html注释，要使用jsp注释
    2、when标签的父标签一定要是choose标签
--%>
<%
  request.setAttribute("height", 180);
%>
<c:choose>
  <%-- 这是html注释 --%>
  <c:when test="${ requestScope.height > 190 }">
    <h2>小巨人</h2>
  </c:when>
  <c:when test="${ requestScope.height > 180 }">
    <h2>很高</h2>
  </c:when>
  <c:when test="${ requestScope.height > 170 }">
    <h2>还可以</h2>
  </c:when>
  <c:otherwise>
    <c:choose>
      <c:when test="${requestScope.height > 160}">
        <h3>大于160</h3>
      </c:when>
      <c:when test="${requestScope.height > 150}">
        <h3>大于150</h3>
      </c:when>
      <c:when test="${requestScope.height > 140}">
        <h3>大于140</h3>
      </c:when>
      <c:otherwise>
        其他小于140
      </c:otherwise>
    </c:choose>
  </c:otherwise>
</c:choose>
```

### `<c:forEach />`

作用：遍历输出使用。

遍历1到10，输出

示例代码：

```jsp
<%--1.遍历1 到10，输出
begin 属性设置开始的索引
end 属性设置结束的索引
var 属性表示循环的变量(也是当前正在遍历到的数据)
for (int i = 1; i < 10; i++)
--%>
<table border="1">
    <c:forEach begin="1" end="10" var="i">
        <tr>
            <td>第${i}行</td>
        </tr>
    </c:forEach>
```

遍历Object 数组:

```jsp
  <%-- 2.遍历Object 数组
        for (Object item: arr)
        items 表示遍历的数据源（遍历的集合）
        var 表示当前遍历到的数据
     --%>
    <%
        request.setAttribute("arr", new String[]
                {"13124325434532","12342345234","13443214321"});
    %>
    <c:forEach items="${requestScope.arr}" var="item">
        ${item}<br>
    </c:forEach>
```

遍历Map集合

```jsp
 <%
        Map<String,Object> map = new HashMap<>();
        map.put("key1", "value1");
        map.put("key2", "value2");
        map.put("key3", "value3");
        // for ( Map.Entry<String,Object> entry : map.entrySet()) {
        // }
        request.setAttribute("map", map);
    %>
    <c:forEach items="${ requestScope.map }" var="entry">
        <h1>${entry.key} = ${entry.value}</h1>
    </c:forEach>
```

遍历List 集合---list 中存放Student 类，有属性：编号，用户名，密码，年龄，电话信息:

```java
public class Student {
    //4.编号，用户名，密码，年龄，电话信息
    private Integer id;
    private String username;
    private String password;
    private Integer age;
    private String phone;
}
```

```jsp
<%--4.遍历List 集合---list 中存放Student 类，有属性：编号，用户名，密码，年龄，电话信息--%>
<%
List<Student> studentList = new ArrayList<Student>();
for (int i = 1; i <= 10; i++) {
studentList.add(new Student(i,"username"+i ,"pass"+i,18+i,"phone"+i));
}
request.setAttribute("stus", studentList);
%>
<table>
<tr>
<th>编号</th>
<th>用户名</th>
<th>密码</th>
<th>年龄</th>
<th>电话</th>
<th>操作</th>
</tr>
<%--
items 表示遍历的集合
var 表示遍历到的数据
begin 表示遍历的开始索引值
end 表示结束的索引值
step 属性表示遍历的步长值
varStatus 属性表示当前遍历到的数据的状态
for（int i = 1; i < 10; i+=2）
--%>
<c:forEach begin="2" end="7" step="2" varStatus="status" items="${requestScope.stus}" var="stu">
<tr>
<td>${stu.id}</td>
<td>${stu.username}</td>
<td>${stu.password}</td>
<td>${stu.age}</td>
<td>${stu.phone}</td>
<td>${status.step}</td>
</tr>
</c:forEach>
</table>
```

