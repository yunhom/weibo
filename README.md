# weibo
仿照weibo做的一个Java web项目。使用SpringBoot加MyBatis开发。主要内容包括：Java语言、Spring入门、数据库交互MyBatis集成、用户注册登录管理、资讯发布、图片上传、评论中心、Redis实现“点赞”和“点踩”功能、异步设计、JavaWeb项目测试

## 项目基础配置

    创建一个SpringBoot工程。
    
    生成Maven项目,pom.xml包含web,mysql,redis,MyBatis依赖。
    
    应用名称是weibo，小组id是com.tyella。
    

## 基本框架开发
    
    创建基本的controller,serivce,model层。
    
    controller中使用注解配置，requestmapping,responsebody
    
    用于解决请求转发以及响应内容的渲染。
    
    
## 数据库的配置

    使用mysql创建数据库和表。创建数据库wiebo，首先创建news和user两张表。
    
    后续开发增加其他功能时再新建其他表。
    
    接下来写controller,dao,service。
    
    application.peoperties增加spring配置数据库链接地址。
    
## 用户注册登录以及使用token
    
    完成用户注册和登录的controller，service和dao层代码。
    
    新建数据表login_ticket用来存储ticket字段。该字段在用户登录成功时被生成并存入数据库，并被设置为cookie，
    
    下次用户登录时会带上这个ticket，ticket是随机的UUID，过期时间以及有效状态。
    
    使用拦截器interceptor来拦截所有的用户请求，判断请求中是否有有效的ticket，如果有的话则将用户信息写入ThreadLocal.
    
    所有线程的threadlocal都被存在一个叫做hostholder的实例中，根据该实例就可以在全局任意位置获取用户信息。
    
    该ticket的功能类似session，也是通过cookie写回浏览器，浏览器请求时再通过cookie传递，区别是该字段是存在数据库中的。
    
    通过用户访问权限拦截器来拦截用户的越界访问，比如用户没有管理员权限就不能访问管理员页面。
    
    配置了用户的webconfiguration来设置启动时的配置，这里可以将上述的两个拦截器加到启动项里。
    
    配置了JSON工具类和MD5工具类，并且使用Java自带的盐生成api将用户密码加密成密文。保证密码安全。
    
    
## 上传图片至云存储，以及新增一条分享内容（相当于发微博）

    图片的本地上传，要求前端请求发送二进制图片文件流，content-type是multipart类型。后端Spring用multipartFile来接收对象
    
    接收对象后验证文件的格式和后缀是否正确，将文件名统一用UUID来命名，并加上后缀名，这样可以避免文件名重复
    
    可以直接使用IOStreamUtil的copy方法将文件直接拷贝到本地，返回一个本地访问路径即可
    
    但是本地存储大量图片显然不合适，所以可以使用阿里云提供的对象存储服务来存储图片
    
    对于阿里云的OSS，首先建立一个bucket，并且获取地址，key和password等信息，添加maven依赖，直接使用一个简单的文件上传例子
    
    就可以实现上传功能，并且返回一个指向该图片存储地址的URL。
    
    新增一个分享内容的步骤（相当于发微博）：选择上传图片，调用图片上传接口并返回URL，然后输入内容，将这些数据一并保存到数据库
    
## 新增评论和站内信功能

    首先建立表comment和message分别代表评论和站内信
    
    依次开发model，dao，service和controller
    
    评论的逻辑是每一条资讯下面都有评论，显示评论数量，具体内容，评论人等信息
    
    消息的逻辑是，两个用户之间发送一条消息，有一个唯一的会话id，这个会话里可以有多条这两个用户的交互信息。
    
    通过一个用户id可以获取该用户的会话列表，再根据会话id获取具体的会话内的多条消息
    
    逻辑清楚之后，再加上一些附加功能，比如显示未读消息数量，根据时间顺序排列会话和消息
    
## 新增“点赞”和“点踩”功能，使用Redis实现

    首先了解一下Redis的基础知识，数据结构，jedis的使用
    
    根据业务封装好jedis的增删差改操作，放在util包中。以此为基础开发点赞和点踩功能
    
    根据需求确定key字段，格式是"like：实体类型：实体id"和"dislike：实体类型：实体id"。这样可以将喜欢一条资讯的人
    
    存在一个集合，不喜欢的存在另一个集合。通过统计数量可以获得点赞和点踩数。
    
    一般点赞点踩操作是先修改Redis的值并获取返回值，然后再异步修改MySQL数据库的likecount数值。
    
    这样既可以保证点赞操作快速完成，也可保证数据一致性。