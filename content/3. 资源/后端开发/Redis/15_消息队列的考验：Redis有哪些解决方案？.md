---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
Redis可以用来做消息队列吗

答案：可以。

首先你得知道消息队列的存取需求。

## 消息队列的消息存取需求

消息队列在进行存取的时候，分成生产者与消费者。

![img](https://static001.geekbang.org/resource/image/f4/62/f470bb957c1faff674c08b1fa65a3a62.jpg?wh=3000*857)

消息队列在进行传输消息的时候，必须满足三个需求：消息保序、处理重复的消息和保证消息可靠性。

### 需求一：消息保存

消息保存的时候需要遵循顺序原则，