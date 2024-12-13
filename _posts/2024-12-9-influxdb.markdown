---
layout: post
title:  "influxdb"
date:   2024-12-9 20:35:35 +0800
categories: command
---

# 查询数据
```
select * from loadTable("dfs://<db_name>", "table_name") where timestamp between 2024.08.09T00:00:00 : 2020.08.09T23:59:59
```
