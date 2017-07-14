---
title: Paginator分页器实例
date: 2017-05-30 09:33:35
categories: [Java]
tags: [分页器]
description:
permalink: paginator
---
# Java中实现一个分页器
<!-- more -->
```java
import java.io.Serializable;
import java.util.ArrayList;
import java.util.List;

/**
 * 分页器
 *
 * @author Create by YL on 2017/05/29.
 */
public class Paging<E> implements Serializable {
    private static final long serialVersionUID = -2429864663690465105L;
    private long page = 1; // 当前页
    private long size = 10; // 每页显示记录数
    private long total; // 总记录数
    private long pages; // 总页数

    private long start = 1; // 每页起始数（从1开始）
    private long end; // 每页结束数
    private long prev = 1; // 上一页（从1开始）
    private long next; // 下一页
    private boolean hasPrev; // 判断是否有上一页
    private boolean hasNext; // 判断是否有下一页

    private List<E> items = new ArrayList<E>(); // 查询结果集

    public Paging() {
    }

    public Paging(long page, long size) {
        this.page = page;
        this.size = size;
    }

    public long getSize() {
        return size;
    }

    public void setSize(long size) {
        this.size = size;
    }

    public long getPage() {
        if (page > this.getPages())
            page = this.getPages();
        if (page <= 0)
            page = 1;
        return page;
    }

    public void setPage(long page) {
        this.page = page;
    }

    public long getTotal() {
        return total;
    }

    public void setTotal(long total) {
        this.total = total;
    }

    public long getPages() {
        long total = this.getTotal();
        long size = this.getSize();
        if ((total % size) == 0) {
            pages = total / size;
        } else {
            pages = total / size + 1;
        }
        return pages == 0 ? 1 : pages;
    }

    public void setPages(long pages) {
        this.pages = pages;
    }

    /**
     * 开始行，可用于Oracle、MySQL等分页
     * <pre>
     * Oracle: rownum <= endRow and rownum > startRow
     * MySQL: limit startRow, pageSize
     * </pre>
     */
    public long getStart() {
        start = this.getPage() > 0 ? (this.getPage() - 1) * this.getSize() : 0;
        return start;
    }

    public void setStart(long start) {
        this.start = start;
    }

    /**
     * 结束行，可以用于oracle分页使用
     * Oracle: rownum <= endRow
     */
    public long getEnd() {
        end = this.getStart() + this.getSize();
        return end;
    }

    public void setEnd(long end) {
        this.end = end;
    }

    public long getPrev() {
        if (isHasPrev()) {
            return getPage() - 1;
        } else {
            return getPage();
        }
    }

    public void setPrev(long prev) {
        this.prev = prev;
    }

    public long getNext() {
        if (isHasNext()) {
            return getPage() + 1;
        } else {
            return getPage();
        }
    }

    public void setNext(long next) {
        this.next = next;
    }

    public boolean isHasPrev() {
        return this.getPage() - 1 >= 1;
    }

    public void setHasPrev(boolean hasPrev) {
        this.hasPrev = hasPrev;
    }

    public boolean isHasNext() {
        return getPage() + 1 <= getPages();
    }

    public void setHasNext(boolean hasNext) {
        this.hasNext = hasNext;
    }

    public List<E> getItems() {
        return items;
    }

    public void setItems(List<E> items) {
        this.items = items;
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder(100);
        sb.append(this.getClass().getSimpleName());
        sb.append(" [");
        sb.append("page=").append(this.getPage());
        sb.append(", size=").append(this.getSize());
        sb.append(", total=").append(this.getTotal());
        sb.append(", pages=").append(this.getPages());
        sb.append(", start=").append(this.getStart());
        sb.append(", end=").append(this.getEnd());
        sb.append(", prev=").append(this.getPrev());
        sb.append(", next=").append(this.getNext());
        sb.append(", hasPrev=").append(this.isHasPrev());
        sb.append(", hasNext=").append(this.isHasNext());
        sb.append(", items=").append(items);
        sb.append("]");
        return sb.toString();
    }
}

```
