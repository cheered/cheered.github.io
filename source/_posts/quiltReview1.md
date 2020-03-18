---
title: quiltReview1
date: 2019-06-09 13:09:55
tags: 
   - quilt
categories: 
   - project-quilt
---

## 被子时间线

## 2019.6.6 （周四晚）

根据页面内容/功能 分析内容及结构，新建数据库的表及字段

写SQL语句时需要注意

1. 创建数据库 （CREATE DATABASE XXX ）
2. 必须标明使用数据库（ USE XXX ）
3. 先判断原先是否存在这张表，存在的话需要删除。（ DROP TABLE IF EXISTS `XXX` ）
4. 写个COMMIT对功能进行说明。。
5. 主键最好单独写出 （PRIMARY KEY （id））
6. 显式的指定一下编码格式 （ENGINE=InnoDB DEFAULT CHARSET=utf8 ）
7. 哦对了 很重要的一点。记住不写注释会被打死

---

## 2019.6.7（周五） 今天是端午节鸭

（这其实是2019/6/19 写的  .. 严重托更 so sorry..）

1. 新建一个maven项目  （后缀是webapp的）
2. 然后根据maven的规则建立目录结构（src+test）

>src

>>main

>>>java

>>>resource

>>test

>>>java

>>>resource

>pom.xml

3. 写POM 文件 （嗯，，这个，是个可重复利用的东西）
4. 把基础环境自动化构建出来。

- 要注意谨慎使用 mvn clean.
---

### 由于时隔十天后继续写，就不按时间线了 统一总结一下。

---
+ 写在所有功能之前的：

+ 所有静态资源的引用路径之前要加上下文环境

   <%=request.getContextPath()%>

---

## 第一个功能是 用户修改密码

1. 首先关于密码 有一个新的知识点是md5加密

   (用md5()包起来就行，但是要引入资源,引入即可,不需修改POM文件)

2. 第二个要考虑的是怎么写，从哪里开始的问题。

   1. 从哪里开始？这其实是个显而易见的问题。当然是根据页面上的信息写。（当然对于自己设计前台页面的话，根据功能或者设定写。）

   2. 本次的页面上提供了四个字段，用户名，当前密码，新密码，确认新密码。所以在service的方法里，参数是上面这三个值，而不是整个封装的用户对象。

   3. 用户可以修改自己的用户名和密码，***并且从session里拿到这个用户的密码，与原密码进行比对，如果相等则确认这个用户***，从而进行修改。（判断的这个逻辑放到service里）

   4. 然后建立一个新的USER对象用来接受修改后的值。

   5. 给新的user对象注入值，然后用userDao 调用更新方法进行修改。根据返回值判断是否更新成功。

   6. 最后如果更新成功，查出更新后的对象并返回。

   7. controller层，用userService调用修改刚刚自定义的修改密码的方法（返回的是更新后的对象，所以新建一个用户对象来接受）。然后判断一下如果接受到的对象为空，则失败，如果不为空，则将这个对象注入到session里，返回成功。

   - 注意controller层，要用try-catch 注意抛上来的异常。
   - 注意controller接受的值是从前台传递过来的，名字一定要相同。

4. 最后修改前台页面的相关数据。

   - 前台 ajax 的URL，指向的是修改的controllerAPI
   - datatype 要写 json。

---

## 第二个功能 是修改用户个人信息

1. 本次设定是规定一个用户名和密码，然后根据这个事先设定进行登陆。所以能标识这个用户的其实就是用户名和密码。

2. 修改用户信息，调用已有的更新用户全部信息的方法即可，所以dao层不需要再自定义新的接口。

3. service层定义一个修改用户信息的方法，注意传入的参数是user的集合，这里没有什么判断的逻辑，直接根据传入的对象参数，调用更新方法即可。

4. controller层，新建一个用户对象，将传入的值进行注入更新，（注意关于头像的注入，要先判断一下不为空且长度大于0，然后调用写好的工具类进行注入。） 注入后调用service层的更新方法，如果返回值为1，代表更新成功，返回success。

   - 对于所有markdown文本里面的图片上传，抽取一个公共的文件上传controller，这里的返回值是一个map，当文件为空或长度小于等于0时，上传失败，否则，将path注入，并且返回。然后对于所有的前台页面，上传路径都相同。
   >imageUploadURL: "<%=request.getContextPath()%>/admin/file/upload",

5. 本功能的一个重难点是，图片上传（也就是文件上传）。更改头像处同步虽整个资料进行提交，个人简介编写处，需要用异步实现，且给用户返回一个地址。

6. 既然有文件上传，先写一个工具类fileuploadutils（工具类，静态成员变量，静态方法）*另外注意文件上传是MultipartFile格式*

   1. 先用SimpleDateFormat指定日期格式。
   2. 先写关于个人简介处的图片上传。（主要有三个步骤）
   
      1. 首先是指定文件保存的路径（这里根据当前日期的年月日保存，便于查找）
      >String path = "/static/upload/" + dateFormat.format(new Date());

      2. 第二步是将整个自己自定义的路径，转换成编译后运行的路径。
      >String realPath = session.getServletContext().getRealPath(path);

      （注意系统实际运行的路径，和自己所编写的路径，是不太一样的，这里要将图片根据运行路径存储，才能访问到。如果直接按路径保存，是访问不到的。 getRealPath()的意义。）

      3. 第三步是指定这个图片保存到服务器的名字。（UUID加它原来的名字）
      >String fileName = UUID.randomUUID() + "_" + file.getOriginalFilename();

      4. 第四步，根据它这个realPath,和新的fileName,创建一个新的文件对象saveFile。

      5. 第五步判判断一下这个saveFile的父文件夹是否存在，不存在的话要创建mkdirs().
      > if (!saveFile.getParentFile().exists()){

            saveFile.getParentFile().mkdirs();
        }

        这里不能直接判断!saveFile.exists(),例如一个目录为F:/static/upload/1.jpg

        saveFile 仅是1.jpg，getParentFile是F:/static/upload/

      6. 存在，保存。
      >file.transferTo(saveFile);

