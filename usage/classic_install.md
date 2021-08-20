# Document.Usage.ClassicInstall

## 经典系统的安装

- NL经典系统，由「后端」和「前端」组成
- 后端就是中心。各个前端作为客户端，连接到后端才可以正常工作
- 前端可以有多种形式，只要符合「NLC-API」协议即可，这套协议保证了前端和后端之间的连接
  - 我们目前为NL经典系统提供「桌面前端」「安卓前端」「网页前端」
  - 只要您遵循「NLC-API」协议，就可以开发出自己喜欢的前端

### 安装后端

- 后端作为服务端，其运行的主机需要具备「一些必备的软件，如数据库」以及「一个能被客户端访问的公网ip（或局域网ip）」
- 我们建议你使用centos系统，不建议使用windows

#### 后端环境准备

- 首先是安装数据库，安装完毕后请记下数据库的ip地址与端口，填入backend/data/json/setting.json中
- [安装数据库](/usage/install_database.md)

- 现在是安装一些基本的环境依赖

  - 请安装python3.6+ centos: `yum install python3`

  - 要安装的python库有：

    - flask
    - pymongo
    - socket
    - pymemcache
    - 可以执行：`python3 -m pip install flask pymongo socket pymemcache`


#### 下载源码

- 创建一个文件夹以存放NL后端

- ```bash
  git clone https://github.com/NothingLeftProject/NothingLeft-Backend.git
  ```

- 然后修改一下配置

  - 用你的方法打开backend/data/json/setting.json，内容如下图所示
  - 首先删除"hostIp"的值
  - 再替换mongodb和memcached默认数据库的地址

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
    "allowSimultaneousOnline": true,  // 允许一个账户多个客户端同时在线
    "allowSignup": true,  // 是否允许自由注册
    "loginValidTime": 24  // 登录信息过期时间，单位：小时
  }
  ```

#### 启动后端

- 输入一下命令，不可以省略cd，一定不可以！

- ```bash
  cd NothingLeft
  python3 run.py
  后台运行：nohup python3 run.py > output.log 2>&1 &
  ```

- 然后控制台就会输出很多信息啦，很方便的，log可以在backend/data/log下找到每天对应的log信息

### 安装前端

- 还没开发好前端呢，不急不急，但是基本就是把前端丢到nginx/apache服务的文件夹下面，然后http访问，然后添加后端地址就可以了

