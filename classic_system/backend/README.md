# Document.ClassicSystem.Backend

- 经典系统的后端的设计与API文档呢
- 首先，我们要了解后端被分为哪几个部分，他们之间是怎么协作的



### API

- 为什么叫API而不是HTTP服务器呢？

  这是因为在./backend/api下有的不只是http服务器，它总实现的一个功能是向外提供API接口，不仅限于http上的

  所以才叫做**API**而不是HTTP服务器

- 请不要把这个部分和API协议混淆了，本系统的HTTP-API协议会在[别的地方](classic_system/backend/http_api_document/README.md)详细说明

- 本部分实现的功能是：

  - 以Flask为框架搭建的web服务所提供http api
  - 单纯的命令行操作所提供的api

- 他们的用处各不相同，都非常有存在的必要

  而且要知道，所有的函数都必须为了他们而有一定的格式形态

  所以如果你想要在这上面开发更多一些什么，这个部分对你来说是很重要的

  （不过这整个架构是我想出来的，所以也有些小自豪，想让你们看看，有建议请务必告诉我）