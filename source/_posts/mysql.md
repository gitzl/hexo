---
title: mysql
---
> 备份

``` bash
mysqldump  -uroot  -p  --all-databases>backup.sql
mysqldump  -uroot  -p  --databases dbname1 dbname2>backup.sql
```
> 恢复

``` bash
mysql  -uroot  -p  <backup.sql
```

> 排查最大连接数

```
show global status like 'Max_used_connections'; 查询设置的最大连接数
show variables like '%max_connections%';  查询历史最大连接数
set GLOBAL max_connections=256; 更新最大连接数

```