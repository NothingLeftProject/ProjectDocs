# Document.Usage
- 使用，就是用户们最关心的地方啦，怎么快速上手，就在这里
- 当然，不同系统的安装方法不同，这里都有的哦



## 经典系统的安装

- 经典系统，由「后端」和「前端」组成
- 后端本质上是一个服务端，接受http请求并处理，前端根据协议发送http请求以进行操作
- 后端使用python实现
- 前端存在多种实现方式，我们这里提供一套web UI，安装在服务器用apache运行就可以使用
- 当然也可以使用别的前端，您自己开发的，符合我们[后端API协议](classic_system/backend/README.md)的就可以了

### 后端安装

- 我们推荐你在类似centos等的linux系统上安装NL-Backend，windows并不大适合，但也确实可以运行

#### 后端环境准备

- 首先，后端是需要一个数据库来支撑了，这个数据库，不，有两个，它们是：MongoDB和Memcached
- 所以，首先我们要安装好两个数据库并将数据库地址填入后端的/data/json/setting.json中
- [STEP.1-安装数据库](usage/install_database.md)

- 现在我们进入第二步

  - 首先安装最基本的Python，要求python3.6+

  - 要安装的python库有：

    - flask
    - pymongo
    - socket
    - pymemcache

  - 安装python库command：

    ```bash
    python3 -m pip install flask pymongo socket pymemcache
    ```

### 安装后端

- 找一个你喜欢的地方：

- ```bash
  git clone https://github.com/NothingLeftProject/NothingLeft
  ```

- 然后修改一下配置

  - 用你的方法打开NothingLeft/backend/data/json/setting.json
  - 将"databaseSettings"->"mongodb"->"address"->"default"和"databaseSettings"->"memcached"->"address"->"default"改成在步骤一里面获得的地址

- setting.json的作用不止如此，现在允许我向你介绍一下：

- ```json
  {
    "httpPort": 6000,  // 服务端端口
    "hostIp": "192.168.3.119",  // 服务端监听地址
    "databaseSettings": {
      "mongodb": {
        "address": {
          "default": "192.168.3.156:27017"  // 默认MongoDB数据库地址
        }
      },
      "memcached": {
        "address": {
          "default": "192.168.3.156:11211"  // 默认Memcached数据库地址
        }
      }
    },
    "allowSimultaneousOnline": true,  // 允许一个用户的多个同时在线
    "allowSignup": true,  // 允许自由注册，决定前端开不开放
    "loginValidTime": 24  // 登录后有效时间，单位：小时
  }
  ```



### 启动后端

- 贼简单的启动后端的方法：

- ```bash
  cd NothingLeft
  python3 run.py
  ```

- 然后控制台就会输出很多信息啦，很方便的，log可以在./backend/data/log找到每天对应的log



### 安装前端

- 还没开发好前端呢，不急不急，但是基本就是把前端丢到nginx/apache服务的文件夹下面，然后http访问，然后添加后端地址就可以了

