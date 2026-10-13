---
layout: post
title:  "clickhouse"
date:   2026-10-12 20:35:35 +0800
categories: command
---

访问查询clickhouse
```
clickhouse-client --user default --password
```

数据库操作
```
SHOW DATABASES;

# 使用数据库
USE market_data;

# 查看当前数据库
SELECT currentDatabase();
```

表操作
```
-- 查看所有表
SHOW TABLES;

-- 查看表结构
DESCRIBE TABLE table_name;

-- 或
DESC table_name;

-- 查看建表语句
SHOW CREATE TABLE table_name;

-- 判断表是否存在
EXISTS TABLE table_name;

-- 删除表
DROP TABLE IF EXISTS table_name;

-- 清空表数据
TRUNCATE TABLE table_name;

-- 创建表
CREATE TABLE IF NOT EXISTS users
(
    id UInt64,
    name String,
    age UInt8,
    amount Decimal(18, 2),
    created_at DateTime
)
ENGINE = MergeTree
ORDER BY id;
```

查询数据
```
-- 查询全部数据
SELECT *
FROM users;

-- 限制返回数量
SELECT *
FROM users
LIMIT 10;

-- 条件查询
SELECT *
FROM users
WHERE age >= 18;

-- 排序
SELECT *
FROM users
ORDER BY created_at DESC
LIMIT 10;

-- 分页查询
SELECT *
FROM users
ORDER BY id
LIMIT 20 OFFSET 40;
```







