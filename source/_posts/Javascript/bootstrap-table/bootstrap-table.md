---
title: bootstrap-table 的使用问题
categories:
  - Javascript
tags:
  - Javascript
  - bootstrap
  - bootstrap-table
abbrlink: b2990e54
date: 2017-08-29 11:55:07
---
# 自定义分页、排序等参数
```javascript
// bootstrap table初始化
$table.bootstrapTable({
    url: '${ctx}/manage/log/list',
    height: getHeight(),
    striped: true,
    search: true,
    showRefresh: true,
    showColumns: true,
    minimumCountColumns: 2,
    clickToSelect: true,
    // --------------------------------------------------------
    // 自定义分页、排序等参数
    // 如果配置'limit'，则 queryParams 默认参数是 offset、limit、order、search、sort
    /*
    queryParamsType: 'limit',
    queryParams: function (params) {
        return {
            page: params.pageNumber,
            size: params.pageSize,
            search: params.search,
            sort: params.sort,
            order: params.order
        };
    },
    */
    // 这里配置成''，则 queryParams 参数是 pageNumber、pageSize、searchText、sortName、sortOrder
    queryParamsType: '',
    queryParams: function (params) {
        return {
            page: params.pageNumber,
            size: params.pageSize,
            search: params.searchText,
            sort: params.sortName,
            order: params.sortOrder
        };
    },
    // --------------------------------------------------------
    detailView: true,
    detailFormatter: 'detailFormatter',
    pagination: true,
    paginationLoop: false,
    sidePagination: 'server',
    silentSort: false,
    smartDisplay: false,
    escape: true,
    searchOnEnterKey: true,
    idField: 'id',
    maintainSelected: true,
    toolbar: '#toolbar',
    columns: [
        {field: 'ck', checkbox: true},
        {field: 'id', title: '编号', sortable: true, align: 'right'},
        {field: 'description', title: '操作'},
        {field: 'username', title: '操作用户', align: 'right'}
    ]
});
```
