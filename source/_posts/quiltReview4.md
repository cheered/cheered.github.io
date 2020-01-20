---
layout: pages
title: quiltReview4
date: 2019-07-01 14:44:37
tags:
---

# 功能性的模块基本完成，接下来就是一些页面的渲染

### 对于页面的渲染，主要就是对JSTL中<c:foreach> 和 <c:if> 等一些循环控制语句标签的使用。以及在指定位置插入特定数据。
```
<c:forEach items="${commentList}" var="comment">
            <c:if test="${comment.commentPid == 0}">
```
### ajax 的使用
```
$.ajax({
                    url:'<%=request.getContextPath()%>/admin/comment/add',
                    data: comment,
                    cache: false,
                    dataType: "json",
                    success: function (data) {
                        if (data.state == 1) {layer.alert("回复成功", {icon: 1});
                        } else{layer.msg("回复失败");} });
                layer.close(index);}});});
```
### 首页分页功能的实现
```
 <c:choose>
   <c:when test="${pagedResult.pageNo == 1 && pagedResult.pages == 1}">
   <span class="page-number current">${pagedResult.pageNo}</span>
   </c:when>

   <c:when test="${pagedResult.pageNo == 1 && pagedResult.pages >1 }">
   <span class="page-number current">${pagedResult.pageNo}</span><a class="page-number" href="<%=request.getContextPath()%>/${pagedResult.pageNo+1}"> >> </a>
   </c:when>

  <c:when test="${pagedResult.pageNo != 1 && pagedResult.pageNo != pagedResult.pages }">
  <a class="page-number" href="<%=request.getContextPath()%>/${pagedResult.pageNo-1}"> << </a>
   <span class="page-number current">${pagedResult.pageNo}</span> <a class="page-number" href="<%=request.getContextPath()%>/${pagedResult.pageNo+1}"> >> </a>
  </c:when>

 <c:when test="${pagedResult.pageNo !=1 && pagedResult.pageNo == pagedResult.pages}">
 <a class="page-number" href="<%=request.getContextPath()%>/${pagedResult.pageNo-1}"> << </a>
 <span class="page-number current">${pagedResult.pageNo}</span>
         </c:when>
       </c:choose> 
```

### 页面信息的注入
```
@RequestMapping("/{pageNo}")
public String getArticlePage(Model model,
                    @PathVariable("pageNo")Integer pageNo ){
 List<ArticleWithBLOBsDto> articleWithBLOBsDtoList = 
new ArrayList<ArticleWithBLOBsDto>();
 PagedResult<ArticleWithBLOBs> pagedResult =     
articleService.getArticlePage(pageNo,2);
 for (ArticleWithBLOBs articleWithBLOBs : pagedResult.getDataList())
{List<Category> categoryList = 
categoryService.getCategoryListByArticleId(articleWithBLOBs.getId());
     List<Tag> tagList = tagService.getTagListByArticleId(articleWithBLOBs.getId());
     ArticleWithBLOBsDto articleWithBLOBsDto = 
new ArticleWithBLOBsDto(articleWithBLOBs,tagList,categoryList);
     articleWithBLOBsDtoList.add(articleWithBLOBsDto);
    }

    model.addAttribute("pagedResult",pagedResult);
   	model.addAttribute("articleWithBLOBsDtoList",articleWithBLOBsDtoList);     		model.addAttribute("backgroundImage","/static/images/index/1555489887530.jpg");
    model.addAttribute("slogan","- 也许世界上也有五千朵和你一模一样的花，但只有你是我独一无二的玫瑰 -");
    return "home/index";
}
```
### 文章详情页标题的提取
```
public static List<Chapter> handleChapeter(String htmlContent){
    Document document = Jsoup.parse(htmlContent);
    Elements elements = document.getElementsByTag("h1");
    int i=1 ;
    List<Chapter> chapterList = new ArrayList<Chapter>();
    for (Element element : elements){
        String chapterName = element.text();
        String chapterId = element.attr("id");
        Chapter chapter = new Chapter(chapterId,i,chapterName);
        i++;
        chapterList.add(chapter);}
    return chapterList;}

```

### 关于分页用到了工具类
```
public static <T> PagedResult<T> toPagedResult(List<T> datas) {
    PagedResult<T> result = new PagedResult<T>();
    if (datas instanceof Page) {
        Page page = (Page) datas;
        result.setPageNo(page.getPageNum());
        result.setPageSize(page.getPageSize());
        result.setDataList(page.getResult());
        result.setTotal(page.getTotal());
        result.setPages(page.getPages());}
    else {
        result.setPageNo(1);
        result.setPageSize(datas.size());
        result.setDataList(datas);
        result.setTotal(datas.size());}
    return result;}

public class PagedResult<T> {
   private List<T> dataList;//数据
   private long pageNo;//当前页
   private long pageSize;//条数
   private long total;//总条数
   private long pages;//总页面数目
}
```

