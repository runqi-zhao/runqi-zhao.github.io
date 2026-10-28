---
annotation-target: Java并发编程艺术/0_电子书/Java并发编程的艺术.pdf
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---



>%%
>```annotation-json
>{"created":"2023-06-10T03:29:28.912Z","updated":"2023-06-10T03:29:28.912Z","document":{"title":"Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf","link":[{"href":"urn:x-pdf:63666633333833623062363362336362633564356135326361386131656636366339376436373062363861376337386532336263323839353039653062346566"},{"href":"vault:/02_%E5%9B%BE%E4%B9%A6%E5%BA%93/java/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf"}],"documentFingerprint":"63666633333833623062363362336362633564356135326361386131656636366339376436373062363861376337386532336263323839353039653062346566"},"uri":"vault:/02_%E5%9B%BE%E4%B9%A6%E5%BA%93/java/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf","target":[{"source":"vault:/02_%E5%9B%BE%E4%B9%A6%E5%BA%93/java/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf","selector":[{"type":"TextPositionSelector","start":10418,"end":10475},{"type":"TextQuoteSelector","exact":"Java编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致地更新，线程应该确保通过排他锁单独获得这个变量","prefix":"与实现原理Java语言规范第3版中对volatile的定义如下：","suffix":"。Java语言提供了volatile，在某些情况下比锁要更加方便"}]}]}
>```
>%%
>*%%PREFIX%%与实现原理Java语言规范第3版中对volatile的定义如下：%%HIGHLIGHT%% ==Java编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致地更新，线程应该确保通过排他锁单独获得这个变量== %%POSTFIX%%。Java语言提供了volatile，在某些情况下比锁要更加方便*
>%%LINK%%[[#^t6hw2q1zgve|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^t6hw2q1zgve


>%%
>```annotation-json
>{"created":"2023-06-10T06:09:27.635Z","text":"这里最好再看看 不是很理解","updated":"2023-06-10T06:09:27.635Z","document":{"title":"Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf","link":[{"href":"urn:x-pdf:63666633333833623062363362336362633564356135326361386131656636366339376436373062363861376337386532336263323839353039653062346566"},{"href":"vault:/02_%E5%9B%BE%E4%B9%A6%E5%BA%93/java/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf"}],"documentFingerprint":"63666633333833623062363362336362633564356135326361386131656636366339376436373062363861376337386532336263323839353039653062346566"},"uri":"vault:/02_%E5%9B%BE%E4%B9%A6%E5%BA%93/java/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf","target":[{"source":"vault:/02_%E5%9B%BE%E4%B9%A6%E5%BA%93/java/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf","selector":[{"type":"TextPositionSelector","start":17279,"end":17289},{"type":"TextQuoteSelector","exact":"使用缓存锁保证原子性","prefix":"其他处理器的请求将被阻塞住，那么该处理器可以独占共享内存。（2）","suffix":"root2022/11/30，16:23:14root2022/"}]}]}
>```
>%%
>*%%PREFIX%%其他处理器的请求将被阻塞住，那么该处理器可以独占共享内存。（2）%%HIGHLIGHT%% ==使用缓存锁保证原子性== %%POSTFIX%%root2022/11/30，16:23:14root2022/*
>%%LINK%%[[#^lnky6of8t78|show annotation]]
>%%COMMENT%%
>这里最好再看看 不是很理解
>%%TAGS%%
>#不是很理解的点
^lnky6of8t78


>%%
>```annotation-json
>{"created":"2023-06-10T06:15:51.470Z","updated":"2023-06-10T06:15:51.470Z","document":{"title":"Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf","link":[{"href":"urn:x-pdf:63666633333833623062363362336362633564356135326361386131656636366339376436373062363861376337386532336263323839353039653062346566"},{"href":"vault:/02_%E5%9B%BE%E4%B9%A6%E5%BA%93/java/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf"}],"documentFingerprint":"63666633333833623062363362336362633564356135326361386131656636366339376436373062363861376337386532336263323839353039653062346566"},"uri":"vault:/02_%E5%9B%BE%E4%B9%A6%E5%BA%93/java/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf","target":[{"source":"vault:/02_%E5%9B%BE%E4%B9%A6%E5%BA%93/java/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf","selector":[{"type":"TextPositionSelector","start":19845,"end":19850},{"type":"TextQuoteSelector","exact":"ABA问题","prefix":"。CAS虽然很高效地解决了原子操作，但是CAS仍然存在三大问题。","suffix":"，循环时间长开销大，以及只能保证一个共享变量的原子操作。1）AB"}]}]}
>```
>%%
>*%%PREFIX%%。CAS虽然很高效地解决了原子操作，但是CAS仍然存在三大问题。%%HIGHLIGHT%% ==ABA问题== %%POSTFIX%%，循环时间长开销大，以及只能保证一个共享变量的原子操作。1）AB*
>%%LINK%%[[#^myiyczuzken|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^myiyczuzken


>%%
>```annotation-json
>{"created":"2023-06-12T03:07:43.117Z","updated":"2023-06-12T03:07:43.117Z","document":{"title":"Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf","link":[{"href":"urn:x-pdf:3602bfcad3a1059b9907c0508e07d8f2"},{"href":"vault:/02_%E5%9B%BE%E4%B9%A6%E5%BA%93/java/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf"}],"documentFingerprint":"3602bfcad3a1059b9907c0508e07d8f2"},"uri":"vault:/02_%E5%9B%BE%E4%B9%A6%E5%BA%93/java/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf","target":[{"source":"vault:/02_%E5%9B%BE%E4%B9%A6%E5%BA%93/java/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf","selector":[{"type":"TextPositionSelector","start":39872,"end":40343},{"type":"TextQuoteSelector","exact":"volatile 总结 :特性 “下 解决:增强编译器规则:望体如下;② 原子性: 单个读写具有原子性 ——∞掉第一个操作与 happensbefore的关系2以州紧材......π⑥写读→释放→获取←保证写生于读(happensbetore)3,如何进行变量的写λ与读取:编译器在生成字节码时, 会使用内屏障禁止特定类型重排序① 写 volatile 将变量写入共享内存 之前⑦插入storestorevolatile写(防止酒与上面的普通潮重排序)① ⻅性: 写λ对于读取可⻅!解决小法: Jsk-133 增强语法, 限制 ∞②写之后插入Sforelodadl防止止面rolatile 与普通读甲排序)θ读Volatile 将读取共享内存的变量 ③读后插入Loadlodl防止下普通读与Volaline 读重排序)会存在什么问题!④ 读之后指入 Loadsforel 防止下普通与 Volative 试重排序)当产生普通读旧与volafive读写胡情况河省略适当步骤混杂时, 会重排序 存在问题: 允许Volative 变量与普通变量排—","prefix":"tz的文章《Java理论与实践：正确使用Volatile变量》。","suffix":"3.5 锁的内存语义众所周知，锁可以让临界区互斥执行。这里将介绍"}]}]}
>```
>%%
>*%%PREFIX%%tz的文章《Java理论与实践：正确使用Volatile变量》。%%HIGHLIGHT%% ==volatile 总结 :特性 “下 解决:增强编译器规则:望体如下;② 原子性: 单个读写具有原子性 ——∞掉第一个操作与 happensbefore的关系2以州紧材......π⑥写读→释放→获取←保证写生于读(happensbetore)3,如何进行变量的写λ与读取:编译器在生成字节码时, 会使用内屏障禁止特定类型重排序① 写 volatile 将变量写入共享内存 之前⑦插入storestorevolatile写(防止酒与上面的普通潮重排序)① ⻅性: 写λ对于读取可⻅!解决小法: Jsk-133 增强语法, 限制 ∞②写之后插入Sforelodadl防止止面rolatile 与普通读甲排序)θ读Volatile 将读取共享内存的变量 ③读后插入Loadlodl防止下普通读与Volaline 读重排序)会存在什么问题!④ 读之后指入 Loadsforel 防止下普通与 Volative 试重排序)当产生普通读旧与volafive读写胡情况河省略适当步骤混杂时, 会重排序 存在问题: 允许Volative 变量与普通变量排—== %%POSTFIX%%3.5 锁的内存语义众所周知，锁可以让临界区互斥执行。这里将介绍*
>%%LINK%%[[#^ug1g4trk5j|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^ug1g4trk5j


>%%
>```annotation-json
>{"created":"2023-07-11T10:36:40.385Z","updated":"2023-07-11T10:36:40.385Z","document":{"title":"Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf","link":[{"href":"urn:x-pdf:78a4070e19c9e807f21da36351150e5f"},{"href":"vault:/02_%E5%9B%BE%E4%B9%A6%E5%BA%93/java/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf"}],"documentFingerprint":"78a4070e19c9e807f21da36351150e5f"},"uri":"vault:/02_%E5%9B%BE%E4%B9%A6%E5%BA%93/java/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf","target":[{"source":"vault:/02_%E5%9B%BE%E4%B9%A6%E5%BA%93/java/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%9A%84%E8%89%BA%E6%9C%AF.pdf","selector":[{"type":"TextPositionSelector","start":116868,"end":116907},{"type":"TextQuoteSelector","exact":"共享式获取与独占式获取最主要的区别在于同一时刻能否有多个线程同时获取到同步状态","prefix":":13Mobile User2023/6/30，15:32:50","suffix":"。以文件的读写为例，如果一个程序在对文件进行读操作，那么这一时刻"}]}]}
>```
>%%
>*%%PREFIX%%:13Mobile User2023/6/30，15:32:50%%HIGHLIGHT%% ==共享式获取与独占式获取最主要的区别在于同一时刻能否有多个线程同时获取到同步状态== %%POSTFIX%%。以文件的读写为例，如果一个程序在对文件进行读操作，那么这一时刻*
>%%LINK%%[[#^lkm91ro8mf|show annotation]]
>%%COMMENT%%
>
>%%TAGS%%
>
^lkm91ro8mf
