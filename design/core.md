# Document.Design.Core



## 核心层

- 不管这个系统（NL）如何实现，它总是离不开一个核心，这个核心就是实现GTD的代码，没有了它，这个项目就没了灵魂
- 那么，这个核心是怎样的呢，其内部结构究竟如何，我们就需要好好探讨一下了
- 我们不妨根据GTD的四个步骤来把核心分为四个基本部分吧

### 收集-COLLECT

- 开始GTD的第一件事情就是收集stuff，把你要做的事情都记录下来，放到一个叫做**INBOX**的地方

- 因此，为了实现这个功能，我们需要提供一个能实现「收集」这个功能的接口

- 实现「收集」这个功能的文件就叫「InboxManager」吧，挺贴切的

- 看看它应该具备什么接口（函数）

#### 1、add_stuff

  - 首先就是收集——也就是记录stuff到inbox的函数。简单理解就是根据用户给出的信息添加stuff到数据库里去

  - 我们的数据库在「经典系统」中是采用MongoDB实现的，存储stuff的数据库就叫「inbox」可以了，接着不同用户就是不同的collection（集合），集合中的记录按json来，记录用户的不同stuff

    - 大概这样：db(stuff) / coll({account}) / docu({stuff:json})

  - 函数的参数就是创建一个stuff所需的所有信息，然后从模板中加载填入信息然后上传，最后返回一个stuff_id

    - stuff_id就用md5(content+salt)生成
    
    - 但为了防止内容相同的stuff的id在巧合情况下还是重合了，我们需要创建一个存储所有stuff_id的记录在_id = 0的位置
  
  - 如果重复了，就更改salt，直到没有重复为止
  
  - 这个salt通过下面这个函数生成，以减少撞车次数，同样也比salt为递增数好
  
    - ```python
      def generate_random_key(self):
      
          """
          生成随机钥匙
          :return:
          """
          maka = string.digits + string.ascii_letters
          maka_list = list(maka)
          x = [random.choice(maka_list) for i in range(6)]
          return ''.join(x)
      ```
    
    
  位于——[./backend/data/encryption.py](https://github.com/NothingLeftProject/NothingLeft/blob/master/backend/data/encryption.py)
  
- 接着就是stuff的信息模板
  
  - ```json
      {
        "content": "",  // 必填，必须是一句话
          "description": "",  // 可选，对基本内容的补充
          "create_date": "",  // 自动生成: YYYY-MM-DD/HH-mm
          "stuff_id": "",  // 自动生成
          "tags": [],  // 可选，标签，方便分类
        "links": [],  // 可选，链接url什么的
          "time": None,  // 可选，stuff执行的时间，如果有
        "place": None,  // 可选，执行stuff的地点
          "level": 0,  // 可选，stuff的优先级，数字，生效必须>0
        "status": "wait_classify"  // 自动生成（也可自填），stuff的状态
      }
    ```
  
      
  
#### 2、modify_stuff

- 第二就是修改日后stuff的信息
  
- 这样的函数需要的参数无非就是「修改对象」和「修改内容」，那么为了更准确，我们需要account, stuff_id, info
    - account: 要修改的stuff的所属用户
    - stuff_id: 这个stuff的id
  - info: 要修改的信息，一个字典，如{"level": 1}
  
#### 3、get_many_stuffs

- 第三就是获取——能写入，也要能读取
  
- 我们的stuff信息是作为一个记录(docu)存储在数据库中的，而获取一个指定的信息也很容易：
  
  - ```python
      get_document(db_name, coll_name, query={"stuff_id": stuff_id}, mode=1)
    ```
  
  - 这个函数是经典系统的[backend.database.mongodb.MongoDBManipulator](https://github.com/NothingLeftProject/NothingLeft/blob/master/backend/database/mongodb.py)类的
  
  - 但是其使用比较复杂，可以查阅：[MongoDBManipulator](design/base.md)里的get_document段落
  
- 为了方便，我们允许以此获取多个stuff，从而参数有：account, stuff_ids, get_all=False, result_type="list"
  
    - stuff_ids应该为list类型，返回自然也是个list
    - get_all就是获取这个账户下面的所有stuff，get_all=True的话，就不需要stuff_ids了
    - result_type是为了满足不同需求的返回存在的，其值可以为dict/list
      - 并且如果get_all=True且result_type="dict"，就会要求检测setting.json->"inboxSettings"->"allowGetAllStuffsInDict"视为为True，False则不予执行
    

#### 4、get_stuff_id
- 获取stuff_id，这是非常重要的
  
- stuff_id除了可以在生成和获取stuff时得到，其它就没有了，然而获取stuff_id也需要id，所以就诞生了这个函数
  
- 我们有几种不同的方式来获取不同的stuff_id以满足不同需求
  
    - 对于首页展示，「最近添加的stuff」「还没有完成的stuff」「还没有分类的stuff」等等这样一系列要求
    
  - 我们提供参数: mode，来完成这件事情
  
    - mode可以是数字(id)或字符串，他们是对应的；字符串是为了方便使用，数字是为了减少出错的几率
  
    - 那么不同mode下的底层逻辑究竟是什么呢？
  
      - 看看我们的要求都有什么样的规律
        - 最近添加的|还没有完成的|还没有分类的
      - 没错，这些stuff的分类都有着明显的依赖要素，而这些要素就可以在stuff的信息里找到
      - 打开NL前端首页显示的是「还没有分类的stuff」，显示多少个——没错，列表，用index来决定显示多少个
      - 然而，我们总不能每次请求get_stuff_id时都重新计算一遍符合要求的stuff有哪些，然后生成一个list
      - 取而代之的方案就是预设好这些列表，存储在数据库中，一个账户被创建时，其stuff数据库下的记录就会相应地有了基本的预设列表
      - 而这些列表存储的就是符合不同要求的stuff的stuff_id，按照时间顺序排列
      - 所以实现不同list的计算最简单地方法就是在stuff信息更改时做出操作
        - 如某个stuff的状态从"wait_classification"变更为"wait_organization"，这样它就从「还没有分类的stuff」的列表里出去了
      - 为了更方便地监控stuff信息变更情况以操作，我们为一些常用操作提供单独函数，而其核心无一例外都是modify_stuff
  
    - 因此，本函数应该像这样：
  
    - ```python
      def get_stuff_id(self, account, mode, start_index=None, end_index=None)
      ```
  
    - 如果你不是要获取全部内容的话，一定要填写start_index，end_index不填写就像[start_index:]而已
  
    - 将start_index和end_index理解为[start_index:end_index]就可以了，实际上也是这么实现的

#### 5、delete_many_stuffs

- 有add就有delete，通过stuff_ids和account删除就可以了

#### 6、generate_preset_stuff_list

- 就是上面提到的预设的储存着stuff_id的list，提供一个生成的函数，大概有用
- 参数：list_name=None 和 account
  - 要生成的列表的名字（和mode一样，可以是字符串或者id），也就是可以指定生成哪些列表
  - 必须要是个列表
  - 如果不指定的话，默认为None，就是全部都生成一遍

#### 7、关于预设列表

- 看看我们都有哪些预设列表以及他们的名称和id是什么呢



### 整理(分类)-CLASSIFY

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

#### 前端设计的思考

- 这里仅仅是对实现前端在「分类」这一步骤上的思考，并不是整个前端的设计。前端设计会有别的篇章讲到
- 现在我们看看都有什么要求：
  - 前端有义务在分类的类别上提供一个清晰的tips，简洁的**几个短语**，让人明白这个stuff究竟应该属于哪里
  - 而且stuff的内容也要**清晰地**显示着
  - 前端一定要**简洁快速**，让stuff像**流水线**一样过去，分错了也可以**快速撤回**
  - 还有**保存**，用缓存来保存着一部分，然后再一齐操作，这样能防止丢失而又不会减慢速度