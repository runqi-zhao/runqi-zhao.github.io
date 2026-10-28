---
type: knowledge
status: active
moc:
  - "[[AI MOC]]"
area:
  - "[[职业发展]]"
created: 2026-09-27
updated: 2026-09-27
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
# Python 基础

返回 [[AI MOC]]

## 它解决什么问题

Python 是 AI 学习和工程实践中常用的编程语言。本笔记作为基础语法入口，为后续的数据处理、机器学习和大模型应用做准备。

## 学习路径

1. 变量、字符串、数字与类型转换
2. 列表、元组、字典与集合
3. 条件判断与循环
4. 函数、参数、返回值与模块
5. 类、对象、继承与组合
6. 文件读写、异常处理与 JSON
7. 单元测试与代码重构

## 核心原则

- 从最小可运行示例开始，再逐步增加条件和数据结构。
- 根据数据关系选择结构：有序元素用列表，按名称查值用字典。
- 用 `for` 遍历已有集合，用 `while` 表达依赖退出条件的过程。
- 函数负责计算并返回结果，输入和输出尽量放在调用处。
- 分别验证正常输入、边界条件和错误输入。

## 最小示例

```python
def summarize_scores(scores: list[int]) -> dict[str, float]:
    """返回成绩数量和平均值。"""
    if not scores:
        return {"count": 0, "average": 0.0}

    return {
        "count": len(scores),
        "average": sum(scores) / len(scores),
    }


result = summarize_scores([80, 90, 100])
print(result)
```

预期输出：

```text
{'count': 3, 'average': 90.0}
```

## 后续笔记

- [[Python 变量与数据类型]]
- [[Python 容器]]
- [[Python 条件与循环]]
- [[Python 函数与模块]]
- [[Python 类与对象]]
- [[Python 文件、异常与测试]]

> 上述链接可以在学习到相应主题时再创建，不需要一次写完。

## 与 AI 学习的连接

- 数据处理：NumPy、pandas
- 机器学习：scikit-learn
- 深度学习：PyTorch
- 大模型应用：模型 API、RAG、Agent

## 下一步

- [x] 创建 [[Python 变量与数据类型]]，完成第一个基础主题 ✅ 2026-09-27


# 总结
以上内容纯属AI生成，现在有了Java的相关基础，所以大致看了看语法，这个不需要太操心