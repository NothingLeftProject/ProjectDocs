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

  - 1、首先是收集——pick_stuff，也就是记录stuff到inbox的函数

    - 简单理解就是根据用户给出的信息添加stuff到数据库里去

    - 我们的数据库在「经典系统」中是采用MongoDB实现的，存储stuff的数据库就叫「inbox」可以了，接着不同用户就是不同的collection（集合），集合中的记录按json来，记录用户的不同stuff

      - 大概这样：db(stuff) / coll({account}) / docu({stuff:json})

    - 函数的参数就是创建一个stuff所需的所有信息，然后从模板中加载填入信息然后上传，最后返回一个stuff_id

      - stuff_id就用md5(content+salt)生成
      - 但为了防止内容相同的stuff的id在巧合情况下还是重合了，我们需要创建一个存储所有stuff_id的记录在_id = 0的位置
      - 如果重复了，就更改salt，直到没有重复为止
      - 这个salt是随机数，以减少撞车次数，同样也比salt为递增数好

    - 接着就是stuff的信息模板

      - ```json
        {
            "content": "",  // 必填，必须是一句话
            "description": "",  // 可选，对基本内容的补充
            "create_date": "",  // 自动生成: YYYY-MM-DD/HH-mm
            "stuff_id": "",  // 自动生成
            "tags": [],  // 可选，标签，方便分类
            "links": [],  // 可选，链接url什么的
            "time": "",  // 可选，stuff执行的时间，如果有
            "place": "",  // 可选，执行stuff的地点
            "level": 0,  // 可选，stuff的优先级
            "status": ""  // 自动生成（也可自填），stuff的状态
        }
        ```

        

  - 2、第二就是修改——modify_stuff，日后stuff的信息

    - 这样的函数需要的参数无非就是「修改对象」和「修改内容」，那么为了更准确，我们需要account, stuff_id, info
      - account: 要修改的stuff的所属用户
      - stuff_id: 这个stuff的id
      - info: 要修改的信息，一个字典，如{"level": 1}

  - 3、第三就是获取——get_stuff，能写入，也要能读取

    - 我们的stuff信息是作为一个记录(docu)存储在数据库中的，而获取一个指定的信息也很容易：

      - ```python
        get_document(db_name, coll_name, query={"stuff_id": stuff_id}, mode=1)
        ```

      - 这个函数是经典系统的[backend.database.mongodb.MongoDBManipulator](https://github.com/NothingLeftProject/NothingLeft/blob/master/backend/database/mongodb.py)类的

    - 为了方便，我们允许以此获取多个stuff，从而参数有：account, stuff_ids, get_all=False

      - stuff_ids应该为list类型，返回自然也是个list
      - get_all就是获取这个账户下面的所有stuff，get_all=True的话，就不需要stuff_ids了

  - 4、第四就是——get_stuff_id了，这很重要

    - stuff_id除了可以在生成和获取stuff时得到，其它就没有了，然而获取stuff_id也需要id，所以就诞生了这个函数

    - 我们有几种不同的方式来获取不同的stuff_id以满足不同需求

      - 对于首页展示，「最近添加的stuff」「还没有完成的stuff」「还没有分类的stuff」等等这样一系列要求

      - 我们提供参数: mode来完成这件事情

        - mode可以是数字或字符串，他们是对应的；字符串是为了方便使用，数字是为了减少出错的几率

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
        def get_stuff_id(self, mode, start_index=None, end_index=None)
        ```

      - 如果你不是要获取全部内容的话，一定要填写start_index，end_index不填写就像[start_index:]而已

      - 将start_index和end_index理解为[start_index:end_index]就可以了，实际上也是这么实现的

