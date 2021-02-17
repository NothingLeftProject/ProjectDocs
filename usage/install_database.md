# Document.Usage.InstallDatabase

- 在这里，你将学会安装MongoDB和Memcached数据库
- 一共有两种方法，方法一更快捷，方法二没啥优势，但也会有人选择



## 方法一

- 通过宝塔面板安装，只要装的上面板，一般就不会有什么问题

- 首先安装宝塔linux面板

  - ```bash
    yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh
    ```

  - 自动安装完成后，打开http://server_address:8888/访问route 来访问

  - 访问route和初始账户与密码都在安装完成之后打印在了bash中

- 第二步就是进入「软件商店」->「运行环境」->「第二页」，找到MongoDB和Memcached然后安装

- 安装完以后原处点开对应的「设置」，保证状态为「开启」

- 然后MongoDB的配置项的「BindIP」设为0.0.0.0以保证全范围监听

- 现在，两个数据库就安装好了，你可以下载MongoDB compass来使用MongoDB

- 然后，请记住这两个数据库的地址

  - MongoDB: http://server_address:27017
  - Memcached: http://server_address:11211
  - 后面的端口是可以修改的，只要对应就好了

- 还有一件很重要的事情——开放安全组

  - 在面板中找到「安全」然后加入MongoDB和Memcached的服务端口并放行

- 完成！



## 方法二

- 一般安装法，不好弄，不适合小白，其实Baidu上随便一搜，找到适合的安装方法就好了
- 所以我也懒得写多一份：
  - [MongoDB安装](https://www.runoob.com/mongodb/mongodb-linux-install.html)
  - [Memcached安装](https://www.runoob.com/memcached/memcached-install.html)