这几天因项目需要要搭建一个小型网站，选用Django，自己学习了一下

与传统的web MVC(model,controller,view)设计模式不同，Django的设计模式是MTV(Model,Template,View):

### web 中的MVC架构
所谓的MVC就是把Web应用分成模型M，控制器C和视图V三层，他们之间以一种插件式，松耦合的方式连接在一起，模型负责业务对象与数据库的映射(ORM)，视图负责与用户的交互(页面)，控制器接收用户的输入调用模型和视图完成用户的请求。

![图片](<images/1.jpg>)

### Django 中的MTV架构
MTV特使为了保证各个组件之间松耦合的关系，M代表模型，负责业务对象和数据库的关系映射ORM，T代表模板负责如何把页面展示给用户HTML，V代表视图负责业务逻辑，并在适当的时候调用Model和Template，还需要一个URL分发器，他的作用是将不同的URL分发给不同的页面进行处理，View再调用相应的Model和Template

Django对http请求的处理流程：
1. web 服务器(中间键)收到一个http请求。
2. Django 在URLconf你查找对应的视图View函数来处理http请求
3. 视图函数调用相应的数据模型来存取数据，调用相应的模板向用户展示页面
4. 视图函数处理结束后返回一个http的响应给web服务器
5. web服务器将相应发送给客户端

![图片](<images/3.jpg>)

进行http请求详细的处理流程：
1. 浏览器发送请求 基本上使字节类型的字符串 到web服务器
2. web服务器Nginx把这个请求转交给WSGI example:uWSGI，或者直接地从文件系统中取出一个文件
3. 不像web服务器那样，WSGI服务器可以直接运行python应用，请求生成一个被称为environ的python字典，而且可以选择传递过去几个中间件的层，最综到达Django应用。
4. URLconf中含有属于应用的urls.py选择一个视图处理URL请求，这个请求已经变成了HTTPRequest，1个python字典对象
5. 被选择的那个视图通常要做下面列出的一件或多件事情
通过模型与数据库对话
使用模板渲染HTML或者任何格式化过的响应
返回一个纯文本的响应
抛出一个异常
6. HTTPResponse对象离开Django后，被渲染为一个字符串
7. 在浏览器见到一个美化的web页面

![图片](<images/2.png>)

常用的设计模式：
1. 命令模式
2. 观察者模式
3. Django RestframWork中的REST设计

<meta http-equiv="refresh" content="5.0">
