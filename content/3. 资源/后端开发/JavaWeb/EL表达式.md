---
id: EL表达式
title: EL表达式
sidebar_label: EL表达式
description: EL表达式
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## 概念与作用

EL 表达式的全称是：Expression Language。是表达式语言。

EL 表达式的作用：EL表达式主要是代替jsp 页面中的表达式脚本在jsp 页面中进行数据的输出。

因为EL 表达式在输出数据的时候，要比jsp 的表达式脚本要简洁很多。

```jsp
<body>
<% request.setAttribute("key1", "value");
%>
<%= request.getAttribute("key1")==null ? "" : request.getAttribute("key1")%><br/>
EL表达式输出key值是:${key1}
</body>
```

EL 表达式的格式是：`${表达式}`

EL 表达式在输出null值的时候，输出的是空串。jsp表达式脚本输出null 值的时候，输出的是null 字符串。

## EL表达式搜索域数据的顺序

EL表达式主要是在jsp页面中输出数据。

主要是输出域对象中的数据。

当四个域中都有相同的key的数据的时候，EL表达式会按照四个域的从小到大的顺序去进行搜索，找到就输出。

```jsp
<%
    //往四个域中都保存了相同的key的数据。
    request.setAttribute("key", "request");
    session.setAttribute("key", "session");
    application.setAttribute("key", "application");
    pageContext.setAttribute("key", "pageContext");
%>
${ key }
```

## EL表达式输出Bean普通属性，数组属性、List 集合

需求——输出Person 类中普通属性，数组属性、list 集合属性和map集合属性。

```java
public class Person {
    // i.需求——输出Person 类中普通属性，数组属性。list 集合属性和map 集合属性。
    private String name;
    private String[] phones;
    private List<String> cities;
    private Map<String,Object> map;
    
    public int getAge() {
        return 18;
    }
}
```

输出的代码:

```jsp
<%
    Person person = new Person();
    person.setName("runqi");
    person.setPhones(new String[]{"1863434343434", "17289127", "13123143345"});
    List<String> cities = new ArrayList<>();
    cities.add("北京");
    cities.add("上海");
    cities.add("深圳");
    person.setCities(cities);
    Map<String, Object> map = new HashMap<>();
    map.put("key1", "value1");
    map.put("key2", "value2");
    map.put("key3", "value3");
    person.setMap(map);
    pageContext.setAttribute("p", person);
%>
输出Person：${ p }<br/>
输出Person 的name 属性：${p.name} <br>
输出Person 的phones 数组属性值：${p.phones[2]} <br>
输出Person 的cities 集合中的元素值：${p.cities} <br>
输出Person 的List 集合中个别元素值：${p.cities[2]} <br>
输出Person 的Map 集合: ${p.map} <br>
输出Person 的Map 集合中某个key 的值: ${p.map.key3} <br>
输出Person 的age 属性：${p.age} <br>
</body>
```

## EL表达式——运算

语法：`${运算表达式}` ，EL表达式支持如下运算符：

### 关系运算

| 关系运算符 | 说明     | 范例                            | 结果  |
| ---------- | -------- | ------------------------------- | ----- |
| == 或eq    | 等于     | `${ 5 == 5 }` 或 `${ 5 eq 5 }`  | true  |
| != 或ne    | 不等于   | `${ 5 !=5 }` 或`${ 5 ne 5 }`    | false |
| < 或lt     | 小于     | `${ 3 < 5 }` 或`${ 3 lt 5 }`    | true  |
| > 或gt     | 大于     | `${ 2 > 10 }` 或`${ 2 gt 10 }`  | false |
| <= 或le    | 小于等于 | `${ 5 <= 12 }` 或`${ 5 le 12 }` | true  |
| >= 或ge    | 大于等于 | `${ 3 >= 5 }` 或`${ 3 ge 5 }`   | false |

### 逻辑运算

| 逻辑运算符 | 说明     | 范例                                                     | 结果  |
| ---------- | -------- | -------------------------------------------------------- | ----- |
| && 或and   | 与运算   | `${ 12 == 12 && 12 < 11 }` 或`${ 12 == 12 and 12 < 11 }` | false |
| `\|\|` 或or | 或运算   | `${ 12 == 12 || 12 < 11 }` 或`${ 12 == 12 or 12 < 11 }`  | true  |
| ! 或not    | 取反运算 | `${ !true }` 或`${not true }`                            | false |

### 算术运算

| 算术运算符 | 说明 | 范例                                | 结果 |
| ---------- | ---- | ----------------------------------- | ---- |
| +          | 加法 | `${ 12 + 18 }`                      | 30   |
| -          | 减法 | `${ 18 - 8 }`                       | 10   |
| *          | 乘法 | `${ 12 * 12 }`                      | 144  |
| /或div     | 除法 | `${ 144 / 12 }` 或`${ 144 div 12 }` | 12   |
| %或mod     | 取模 | `${ 144 % 10 }` 或`${ 144 mod 10 }` | 4    |

### empty运算

empty 运算可以判断一个数据是否为空，如果为空，则输出true，不为空输出false。

以下几种情况为空：

1. 值为null 值的时候，为空
2. 值为空串的时候，为空
3. 值是Object 类型数组，长度为零的时候
4. list 集合，元素个数为零
5. map 集合，元素个数为零

```jsp
<body>
<%
    //1. 值为null值的时候，为空
    request.setAttribute("emptyNUll", null);
    //2. 值为空串的时候，为空
    request.setAttribute("emptyStr", "");
    //3.值为Object类型数组，长度为零的时候
    request.setAttribute("emptyArr", new Object[]{});
    //4.list集合，元素个数为零
    List<String> list = new ArrayList<>();
    request.setAttribute("emptyList", list);
    //5. map集合，元素个数为零
    Map<String, Object> map = new HashMap<>();
//    map.put("key1","value1");
    request.setAttribute("emptyMap", map);
%>
${empty emptyNUll} <br/>
${ empty emptyStr } <br/>
${ empty emptyArr } <br/>
${ empty emptyList } <br/>
${ empty emptyMap } <br/>
</body>
```

### 三元运算

表达式1？表达式2：表达式3

如果表达式1 的值为真，返回表达式2 的值，如果表达式1 的值为假，返回表达式3 的值。

示例：

```jsp
${ 12 != 12 ? "true":"false" }
```

### “.”点运算和[] 中括号运算符

.点运算，可以输出Bean 对象中某个属性的值。

[]中括号运算，可以输出有序集合中某个元素的值。

并且[]中括号运算，还可以输出map 集合中key 里含有特殊字符的key 的值。

```jsp
<body>
<%
    Map<String, Object> map = new HashMap<>();
    map.put("a.a.a","aaaValue");
    map.put("b+b+b","bbbValue");
    map.put("c-c-c","cccValue");
    request.setAttribute("map",map);
%>
${map['a.a.a']}<br/>
${map["b+b+b"]}<br/>
${map['c-c-c']}<br/>
</body>
```

## EL表达式的11个隐含对象

EL表达式中11个隐含对象，是EL表达式中自己定义的，可以直接使用。

| 变量             | 类型                 | 作用                                                    |
| ---------------- | -------------------- | ------------------------------------------------------- |
| pageContext      | PageContextImpl      | 它可以获取jsp 中的九大内置对象                          |
| pageScope        | Map<String,Object>   | 它可以获取pageContext 域中的数据                        |
| requestScope     | Map<String,Object>   | 它可以获取Request 域中的数据                            |
| sessionScope     | Map<String,Object>   | 它可以获取Session 域中的数据                            |
| applicationScope | Map<String,Object>   | 它可以获取ServletContext 域中的数据                     |
| param            | Map<String,String>   | 它可以获取请求参数的值                                  |
| paramValues      | Map<String,String[]> | 它也可以获取请求参数的值，获取多个值的时候使用。        |
| header           | Map<String,String>   | 它可以获取请求头的信息                                  |
| headerValues     | Map<String,String[]> | 它可以获取请求头的信息，它可以获取多个值的情况          |
| cookie           | Map<String,Cookie>   | 它可以获取当前请求的Cookie 信息                         |
| initParam        | Map<String,String>   | 它可以获取在web.xml 中配置的`<context-param>`上下文参数 |

### EL获取四个特定域中的属性

pageScope 		=> 	pageContext 域
requestScope  	=> 	Request 域
sessionScope  	=> 	Session 域
applicationScope  => 	ServletContext 域

```jsp
<body>
<%
  pageContext.setAttribute("key1", "pageContext1");
  pageContext.setAttribute("key2", "pageContext2");
  request.setAttribute("key2", "request");
  session.setAttribute("key2", "session");
  application.setAttribute("key2", "application");
%>
${ applicationScope.key2 }
```

### pageContext 对象的使用

1. 协议：
2. 服务器ip：
3. 服务器端口：
4. 获取工程路径：
5. 获取请求方法：
6. 获取客户端ip 地址：
7. 获取会话的id 编号：

```jsp
<body>
<%--
request.getScheme() 它可以获取请求的协议
request.getServerName() 获取请求的服务器ip 或域名
request.getServerPort() 获取请求的服务器端口号
getContextPath() 获取当前工程路径
request.getMethod() 获取请求的方式（GET 或POST）
request.getRemoteHost() 获取客户端的ip 地址
session.getId() 获取会话的唯一标识
--%>
<%
  pageContext.setAttribute("req", request);
%>
<%=request.getScheme()%><br>
1.协议： ${req.scheme}<br>
2.服务器ip：${ pageContext.request.serverName }<br>
3.服务器端口:${pageContext.request.serverPort}<br>
4.获取工程路径:${pageContext.request.contextPath}<br>
5.获取请求方法：${pageContext.request.method}<br>
6.获取客户端ip地址:${pageContext.request.remoteHost}<br>
7.获取会话的id编号:${pageContext.session.id}<br>
</body>
```

#### EL表达式其他隐含对象的使用

| param       | `Map<String,String>`   | 它可以获取请求参数的值                         |
| ----------- | ---------------------- | ---------------------------------------------- |
| paramValues | `Map<String,String[]>` | 它也可以获取请求参数的值，获取多个值的时候使用 |

```jsp
输出请求参数username 的值：${ param.username } <br>
输出请求参数password 的值：${ param.password } <br>
输出请求参数username 的值：${ paramValues.username[0] } <br>
输出请求参数hobby 的值：${ paramValues.hobby[0] } <br>
输出请求参数hobby 的值：${ paramValues.hobby[1] } <br>
```

请求地址：

`http://localhost:8080/tomcat/h.jsp?username=runqi&password=666666&hobby=java&hobby=cpp`

| header       | `Map<String,String>`   | 它可以获取请求头的信息                         |
| ------------ | ---------------------- | ---------------------------------------------- |
| headerValues | `Map<String,String[]>` | 它可以获取请求头的信息，它可以获取多个值的情况 |

示例代码：

```jsp
输出请求头【User-Agent】的值：${ header['User-Agent'] } <br>
输出请求头【Connection】的值：${ header.Connection } <br>
输出请求头【User-Agent】的值：${ headerValues['User-Agent'][0] } <br>
```

`cookie 	Map<String,Cookie> 	它可以获取当前请求的Cookie 信息`

```jsp
获取Cookie 的名称：${ cookie.JSESSIONID.name } <br>
获取Cookie 的值：${ cookie.JSESSIONID.value } <br>
```

`initParam 	Map<String,String> 	它可以获取在web.xml 中配置的<context-param>上下文参数`

```jsp
<context-param>
<param-name>username</param-name>
<param-value>root</param-value>
</context-param>
<context-param>
<param-name>url</param-name>
<param-value>jdbc:mysql:///test</param-value>
</context-param>
```

示例代码：

```jsp
输出&lt;Context-param&gt;username 的值：${ initParam.username } <br>
输出&lt;Context-param&gt;url 的值：${ initParam.url } <br>
```

