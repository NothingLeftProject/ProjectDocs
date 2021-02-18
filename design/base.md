# Document.Design.Base

- 在这里，包含着对于基础操作的设计，是使核心得以活动的肉体
- 在这里，有关于「经典系统」和「一体系统」的数据库实现的设计等



## 数据库设计

### 经典系统数据库

- 经典系统的数据库有两个，分别是MongoDB和Memcached
- 很明显，MongoDB才有作为主数据库的资格

#### MongoDB

- 为了实现通过Python操作MongoDB，我写了一个类：MongoDBManipulator
  - 位于[./backend/database/mongodb.py](https://github.com/NothingLeftProject/NothingLeft/blob/master/backend/database/mongodb.py)
- 现在，我们来介绍一下它
  - 这个类提供了各种对数据库的操作，查询，更新，检测等等，采用的是第三方库：pymongo
  - 然后这个类的设置在setting.json->"databaseSettings"->"mongodb"

##### __init__

- ```python
  def __init__(self, log, setting, database_name="default"):
  ```

- 首先是初始化函数
- 它具有一个可以调控的参数：database_name: 也就是使用哪个数据库，对应设置中的"adress"里的key
  - 如果database_name无法找到，会在log里报错
- init会生成一个操作对象：self.server，然后自动获取目前有的数据库名称

##### add_database

- ```python
  def add_database(self, db_name):
  
      """
      添加数据库
      :param db_name: 数据库名
      :return: bool
      """
      self.log.add_log("MongoDB: try add database: " + db_name, 1)
      try:
          self.server[db_name]
      except:
          return False
      else:
          self.get_database_names_list()
          return True
  ```

- 这个函数会添加一个数据库

  - 我们知道MongoDB的结构：
    - 数据库->集合->记录

- 在启动该项目的时候，maintainer会初始化并添加user, user_group, stuff数据库

- 本函数添加完以后还会更新数据库名称列表，失败则返回False

##### add_collection

- 与上面的add_database大同小异，就不列举了
- 只是会有两个参数：db_name, coll_name

##### delete_collection

- 就是删除而已，与add_collection也大同小异

##### get_database_names_list / get_collection_names_list

- 就是获取现有的数据库/集合名称列表到self.database_names_list和self.collection_names_list = []

  - 但要注意：self.collection_names_list是一个字典，它的结构如下：

    - ```json
      {
          "user": ["test", "root"],
          "user_group": []
      }
      ```

##### is_database_extst / is_collection_exist

- 这两个函数分别是验证某个数据库(db)/集合(coll)存不存在的，他们都有一个可选参数：update
  - 首先，这两个函数实现的原理就是检测某个db/coll在不在self.database/collection_names_list里
  - 然而，我们知道self.database/collection_names_list可能会与实际有偏差，这个时候就需要update参数来看看是不是要先更新一下self.database/collection_names_list
  - 不过就算不更新，这两个函数在第一次判断到「不存在」之后还会自动更新一遍，再检测一遍以防止遗漏

##### add_one_document

- ```python
  def add_one_document(self, db_name, coll_name, docu):
  
      """
      添加单个文档
      :param db_name: 数据库名
      :param coll_name: 集合名
      :param docu: 要插入的文档内容 dict
      :return: False or class
      """
      try:
          db = self.server[db_name]
      except:
          self.log.add_log("MongoDB: no database named " + db_name + " or something else wrong", 3)
          return False
      else:
          try:
              coll = db[coll_name]
          except:
              self.log.add_log("MongoDB: no collection named " + coll_name + " or something else wrong", 3)
              return False
          else:
              try:
                  result = coll.insert_one(docu)
              except:
                  self.log.add_log("MongoDB: add one document fail", 3)
                  return False
              else:
                  self.log.add_log("MongoDB: add one document success", 1)
                  return result
  ```

- 就是添加一个记录到db-xx/coll-xx里面

- 这个docu必须是一个字典，因为是一个记录

  - 为了方便日后的查询，推荐这样

    - ```json
      {"_id": 0, "xxxx": xxxx}
      ```

      xxxx才是真实的你要记录的数据，我推荐的是指定_id

##### add_many_documents

- 就是向db-xx/coll-xx里一次性添加**多个**记录

- 所以参数：docu_s必须是个列表，然后列表里面有的是一个记录，很正常

##### get_document(重点)

- 这是个重点，不仅是因为这个函数很重要，更是因为这个函数的使用比较复杂

- 我们看一个使用示例：

  - ```python
    value = self.mongodb_manipulator.parse_document_result(
            self.mongodb_manipulator.get_document(db_name, coll_name, query={key: 1}, mode=2),
            [key]
    )[0][key]
    ```

- 可以看到，要获取一个数据，我们要用到两个函数：parse_document_result和get_document

- 那么，就让我们看看get_document是怎样使用的先：

  - ```python
    def get_document(self, db_name, coll_name, query=None, mode=0):
    ```

  - 他有两个关键参数：query和mode，根据mode不同，query也不同

    query=None也就等于{}

  - mode有0, 1, 2三种可选值

    - mode=0: 获取这个集合下的全部数据，query不生效

    - mode=1: 获取有符合query值的记录返回

      - query在这时是一个dict

      - 「符合值」是指符合query中的key-value

      - 如：一个集合里有如下记录：

        - ```json
           [{"name": "yyh", "age": 15}, {"name": "yqq", "age": 14}]
          ```

        - 那么要查询yyh这个人的年龄，我们就可以指定mode=1, query={"name": "yyh"}，这样就可以得到{"name": "yyh", "age": 15}了

        - 当然也可以说要查询有哪些人的年龄是15的

    - mode=2: 比较复杂，根据query不同，效果不同

      - query格式为{key: 0/1}；key的值只可以是0或者1且**除了_id，1和0不能共存**

        - 0是指这个key不返回**且其它都要返回**

        - 1是指这个key一定要返回**且其它都不要**

        - 比如集合里有记录：

        - ```json
           [{"_id": 0, "name": "yyh", "age": 15}, {"_id": 1, "name": "yqq", "age": 14}]
          ```

        - 现在你想要获取age这个key的值，那你可以使query如下

          - {"age": 1}
          - {"name": 0, "_id": 0}
          - {"__id": 0, "age": 1}  // 这其实就是所谓的除了_id，1和0不能共存

        - 但不可以如下：

          - {"name": 0, "age": 1}

        - 到这里，我觉你应该已经理解了

    - mode2和mode1的区别就在于：

      - mode1返回的是符合query要求的整个记录
      - mode2返回的是符合query要求的那个key或反选的key

- 所以我们为什么还要多一个parse_document_result呢？

  - 这是因为get_document返回的不是一个简单的list，是一个类，一个有list性质的类
  - 当然我们可以用list()来使它变成一个list，不过我选择写一个函数来解析...为什么？
  - 这是因为mode2才可以符合我的需求，我们看一个例子：
    - 执行登录操作：要从数据库里获取用户的密码来查看是否正确
    - 而这个password字段在db-user/coll-用户名 这个集合下面的{"_id": 1, "password": "xxxxxxxx"}这个记录里
    - 这时候，用mode1是不可能的，因为我不知道password里的值...但是要查询password对应的记录的_id又太麻烦，所以用了mode=2？嗯？我有点怀疑了...虽然确实运行了，但好像mode1也可以的啊...不管了...总之就是用解析器就对了

- 所以我们现在来看看parse_document_result：

  - ```python
    def parse_document_result(self, documents, targets, debug=True):
    ```

  - 解释一下参数：

    - documents就是get_document返回的
    - targets是一个list，是你要获取的keys

  - 他的原理就是找到documents里有targets中的key的记录并放在一个列表里返回

    - 如，有一个get_document的返回如下：

      - ```json
        [
            {	
                "_id": "xxx",
            	"content": "xxx",
            	"description": "xxx",
            	"createDate": "xxx",
            	"stuffId": "xxx",
            	"tags": [],
            	"links": [],
            	"time": "xxx",
            	"place": "xxx",
            	"level": 0,
            	"status": "xxx"
        	}
        ]
        ```

      - 那么要获取content就可以使targets = ["content"]

      - 然后content = result->0->"content"，因为result是一个list，所以有个0

        - 关于result必须是个list是因为给出的documents里可能有多个document都存在targets中要求的某个key

        - 就像targets = ["content"]，但get_document的返回有多个记录：

        - ```json
          [
              {	
                  "_id": "xxx",
              	"content": "xxx"
          	},
          	{	
                  "_id": "aaa",
              	"content": "aaa"
          	}
          ]
          ```

        - 那么这时候解析结果->0->"content" = "xxx"; -> 1 -> "content" = "aaa"