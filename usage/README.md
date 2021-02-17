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