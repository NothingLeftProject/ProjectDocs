# Document.Design.Core



## 核心层

- 不管这个系统（NL）如何实现，它总是离不开一个核心，这个核心就是实现GTD的代码，没有了它，这个项目就没了灵魂
- 那么，这个核心是怎样的呢，其内部结构究竟如何，我们就需要好好探讨一下了
- 我们不妨根据GTD的四个步骤来把核心分为四个基本部分吧

### 收集-COLLECT

- 开始GTD的第一件事情就是收集stuff，把你要做的事情都记录下来，放到一个叫做**INBOX**的地方
- 因此，为了实现这个功能，我们需要提供一个能实现「收集」这个功能的接口
- 实现「收集」这个功能的文件就叫「InboxManager」吧，挺贴切的
- 现在，让我们看看它的基本设计吧！
  - [InboxManager](/design/inbox_manager.md)

### 分类-CLASSIFY

- 作为在收集完成堆stuff之后，将其处理并整理，起着至关重要的分流作用的分类，我们采用「ClassificationManager」来实现
- 对stuff的添加删除、classification的添加删除、查找、判断等等都在这里实现
  - 现在，去看看它究竟是怎么样的吧：[ClassificationManager](/design/classification_manager.md)