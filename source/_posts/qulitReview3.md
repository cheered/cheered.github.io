---
layout: pages
title: qulitReview3
date: 2019-06-23 00:16:08
tags: 
  - quilt
categories: 
  - project-quilt
---

## 第五个功能 写文章页面

1. 这个页面，。。写的时候。。有些难度来着

2. 难点主要是关联查询。要将这篇文章所勾选的标签和分类一并提交至数据库（即批量插入标签和分类，故mapper部分还涉及到动态SQL语句），这就涉及到了一对多的问题。

   - 为此，首先在建表阶段，要建立一个文章标签关联表和文章分类关联表。
   - 在各自关联对象的DAO层，需要添加一个批量插入的接口。然后去mapper文件里使用动态SQL语句（也就是对插入的值进行一个foreach循环）来实现。
   - 其次，要解决的问题是，提交文章后，按照逻辑来讲应该按照数据库里这个文章的ID，创建关联对象，然后再插入标签和分类。怎么得到这个文章的ID呢？在文章插入的mapper实现方法处添加keyProperty这个属性，用来获得插入成功后获得的主键然后设useGeneratedKeys为true，标识主键自增。*注意这两个属性只能在insert和update里面用。*
   - 但是插入添加了这个字段以后，返回值还是影响的条数，但是可以通过这个对象得到插入成功后的ID。
   - 在业务逻辑层，首先将除标签和分类之外，文章表里有的字段信息进行插入，如果插入成功，返回值为1，且categoryId或者tagId不为空，那么再继续插入标签和分类。
   - 创建一个关联对象的List，然后循环对其中的每一个关联对象进行插入设置。*在循环体里创建关联对象*（标签和分类类似）
   - 然后将这个list批量插入到表里。
   - 插入文章的时候只涉及了文章表和关联表，并不涉及具体的标签表和分类表。

3.最后去controller层，先把标签和分类信息渲染出来，然后接受所有前台传递过来的信息，新建一个文章对象进行接受，将这些信息注入后，调用service层写的插入文章方法，返回值为1，则成功。

4.前台JSP页面

   - 首先先对标签和分类进行循环输出（El表达式）
   - 然后对回调函数的输出进行简单修改。

---

## 第六个功能 所有文章页面


1. 然后编写接口：

   - 根据文章ID/分类ID/标签ID 均可以获得和删除关联。
   - 然后写对应的service。
   - 这里删除父类节点时，如果有子节点，就不允许删除。
   - 根据父节点获取子节点，然后如果子节点不为0，不允许删除。否则先删除关联，再删除此分类。

2. 关于获得文章信息（这里用Article对象就OK）

   - 写获得文章信息时，需要自定义一个接口，mapper文件处自定义查询字段。

3. 还需要再在service层实现根据文章ID获取所对应的标签/分类。

   - 先创建一个关联对象的list
   - 再创建一个tag对象的list
   - 将关联对象循环，拿到每一个在关联表里面对应的tagid
   - 根据拿到的这个tagid,去标签表里面查找，返回一个tag对象
   - 将这个tag对象添加到tag对象的list里面 taglist.add(tag)

4. 怎样在前台输出每条记录呢？这里用到了POJO包装类。

   - 在dto包里面建立一个包含了文章对象，标签列表，分类列表的包装类。（因为一个文章对应多个标签或者分类）
   
5. 在controller层，首先建立一个文章list得到所有文章，然后再建立一个包装对象的List,遍历文章的list,为每一个文章注入它所对应的标签和分类（调用service层根据文章ID获取分类和标签的方法，获得所有对应的标签和分类存到一个组里面，然后新建一个包装对象来接受，接受后将此对象添加到list里面），最后通过model，把包装对象的list渲染到前台页面。

6. 前台页面渲染时，关于时间的显示，需要按照更新时间倒序排列。
`<fmt:formatDate value="${articleListDto.article.createTime}" pattern="yyyy-MM-dd HH:mm:ss" />`

7. 删除功能，需要先删除和文章，分类的关联，然后删除此文章。

8. 修改功能，点击修改按钮，进入一个类似于写文章的页面，但是需要渲染，将本文章原有信息渲染成默认值。这里的难点还是对于标签和分类的渲染。先得到用户选择的所有标签和分类的list，然后在前台通过script代码，得到每一个标签的ID。

`
   
    var tagList = new Array();
    var categoryList = new Array();

    <c:forEach items="${tags}" var="tag">
         tagList.push(${tag.id});
    </c:forEach>

    <C:forEach items="${categories}" var="category">
         categoryList.push(${category.id});
    </C:forEach>

    for (var i = 0; i < tagList.length ; i++) {

        $('input[value=' + tagList[i] + '][name="articleTagId"]').attr("checked","checked");
    }

    for (var i = 0; i < categoryList.length ; i++) {

        $('input[value=' + categoryList[i] + '][name="articleCategoryId"]').attr("checked","checked");
    }
`

9. 修改方法的具体实现类似于写文章的功能。

---

## 用户评论及回复

1. 删除：自定义一个根据PID删除的接口，即如果有子节点（即回复），删除时先把其下的回复删除。先调用根据PID删除的方法，然后调用原本的删除方法。

---

## 渲染后台主页

1. 首先关于查询各种总数以及最新八条记录，尽量重新定义接口，不要让数据库全查出来再去取。

2. 关于显示最近八条记录，需要在SQL语句中实现

`select * from aaa order by create_time desc(降序排列) limit 8 `


---

后记，于6月27日晚，后台功能基本完成
成长很多，继续努力哈哈。
