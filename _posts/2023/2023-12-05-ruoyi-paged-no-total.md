---
title: 若以分页失效，显示总数为当前查询集合数量错误
date: 2023-12-05 21:58:55 UTC+8
categories: [文章]
tags: [java, ruoyi]     # TAG names should always be lowercase
# author: 1
---

修改 Controller 代码如下所示:
```java
@GetMapping("/list")
public TableDataInfo pagedList(LzPatrolRecordPagedListRequestVO lzPatrolRecordPagedListRequestVO) {
    // startPage();
    PageDomain pageDomain = TableSupport.buildPageRequest();
    Page<Object> page = PageHelper.startPage(pageDomain.getPageNum(), pageDomain.getPageSize());
    List<LzPatrolRecordPagedListResponseVO> responseVOList = lzPatrolRecordService.pagedList(lzPatrolRecordPagedListRequestVO);
    TableDataInfo dataTable = getDataTable(responseVOList);
    dataTable.setTotal(page.getTotal());
    return dataTable;
}
```