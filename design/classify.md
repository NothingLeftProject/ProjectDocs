# Document.Design.Core.Classify

- inbox里有了stuff，我们还要对其进行分类整理，因为stuff不一定是一件要做的事情，stuff是灵活的，多样的
- 然后为了避免使用者慌乱添加一些奇怪的分类类别或者分得太细而失去原本的目标，一般就**不把添加分类类别的按钮设计得很明显**
- 取而代之的就是两个默认的分类类别：「可以行动的」「不可以行动的」，很简单，为了方便，我们提供的前端操作也要像**分豆子**一样简洁明了
- 不过这只是初步的，我们还应该进行稍微详细的分类
  - 比如，对于「不可以行动的」，进一步分为「参考资料」「未来做/追踪」「垃圾」
  - 对于「可以行动的」，进一步按照能否在两分钟以内执行而操作，可以则立马行动，不可以则留在原位等待**组织**
- 关键的问题出现了——分类的标准！
  - 上面可以看到，如果是未来要做的，其实是属于「不可以行动的」，所以**前端有着很重要的责任**
    - 前端有义务在分类的类别上提供一个清晰的tips，简介的几个短语，让人明白这个stuff究竟应该属于哪里
    - 而且stuff的内容也要清晰地显示着

#### 分类在前端设计上的思考

- 这里仅仅是对实现前端在「分类」这一步骤上的思考，并不是整个前端的设计。前端设计会有别的篇章讲到
- 现在我们看看都有什么要求：
  - 前端有义务在分类的类别上提供一个清晰的tips，简洁的**几个短语**，让人明白这个stuff究竟应该属于哪里
  - 而且stuff的内容也要**清晰地**显示着
  - 前端一定要**简洁快速**，让stuff像**流水线**一样过去，分错了也可以**快速撤回**
  - 还有**保存**，用缓存来保存着一部分，然后再一齐操作，这样能防止丢失而又不会减慢速度



### 实现-ClassificationManager

- 即「分类管理器」——「ClassificationManager」

#### 1、add_many_stuffs

- 作用：添加多个stuff到一个分类列表中
- 参数：account, cl_id, stuff_ids(list)
  - 注意，如果你只有cl_name，那就需要使用cl_name_converter来通过分类名获取分类id
- 处理流程：
  - 验证参数合法性：
    - 其实在所有的命令当中，我们都需要验证参数的合法性，验证参数的合法性主要有以下几个内容：
    - 1.账户是否存在
    - 2.stuff_id or cl_id是否存在
    - 3.特定条件验证
    - 正如上面提到的，所有命令基本都是需要「账户」「id」的验证，所以提供了general_existence_judgment
  - 主程序
    - 就是先从数据库获取该分类的info(document)
    - 分离出stuffsList并填入验证过存在的stuff_id
    - 更新到数据库
      - 注意BUG：如果document中添加了新的key，那么就需要删掉重新添加；否则无效

#### 2、remove_many_stuffs

- 作用：从一个分类中删除多个stuff
- 参数：account, cl_id, stuff_ids(list)
- 处理流程：
  - 验证参数合法性
  - 就是反过来从append变成remove的add_many_stuffs而已，还有不用验证stuff是否存在

#### 3、get_classifications_info

- 作用：获取多个分类的信息
- 参数：account, cl_ids(list), designated_keys(list)，get_all(bool), result_type(str)
  - designated_keys: 可选参数；获取哪些信息，一个列表，包含了存在的key
    - 要参考有哪些可选的keys，请参考：[json/standard_classification_info_keys]()
  - get_all: 可选参数；是否获取所有classification的所有信息
    - 为True时，designated_keys仍然生效，而cl_ids不生效
  - result_type: 可选参数；选填"list"/"dict"，默认"list"；返回类型的选择
    - list则按照给出了cl_ids进行排序对应返回，get_all启用则是按照分类创建的时间顺序
    - dict则按照{cl_id: cl_info}的模式
- 处理流程：
  - 参数合法性验证
    - 独立验证：designated_keys与cl_ids
  - 主程序
    - 先判断designated_keys是否为None，不是则进行不合法的key的去除
    - 再判断返回类型是什么并建立相应的result
      - 再判断是否获取全部分类信息
      - get_all的True与False唯一区别就是是否检查cl_id的存在

