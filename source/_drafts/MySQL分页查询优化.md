---
title: MySQL分页查询优化
date: 2024-07-31 23:00:30
tags:
  - MySQL
  - 数据库
---
同事仙贝提醒在使用 MySQL 查询时，可以对写法进行一点优化。

# 常规写法

通常在分页查询时，我们会使用两个参数来获取结果：`pageNo` 和 `pageSize`，分别表示页数和每页记录数。

在拼接成 SQL 后类似：

```sql
SELECT * FROM users WHERE OFFSET ${pageNo} LIMIT ${pageSize}
```

