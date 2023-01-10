---
title: how to use PostgreSQL 
tags: ["postgresql"]
categories: []
date: 2022-05-26
lastmod: 2022-05-26
---

## 概述


## 架构


## 常用命令


### 连接

```sh 
# connection
export PGPASSWORD=examplepassword ;  psql -h 127.0.0.1 -p 5432 -U admin -d database-name
```

### 创建数据库

```sh
## create database, use binary file `createdb`
createdb example001
```

```
# create database example001;
```



### show databases


* Use `\l` or `\l+` in psql to show all databases in the current PostgreSQL server.
* Use the `SELECT` statement to query data from the `pg_database` to get all databases.

use `\l` 或是 `\l+` 获取到更多的信息；

```sql
postgres-# \l
                                  List of databases
    Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
------------+----------+----------+-------------+-------------+-----------------------
 example001 | postgres | UTF8     | en_US.utf-8 | en_US.utf-8 | 
 pgdb       | postgres | UTF8     | en_US.utf-8 | en_US.utf-8 | =Tc/postgres         +
            |          |          |             |             | postgres=CTc/postgres+
            |          |          |             |             | pguser=CTc/postgres
 postgres   | postgres | UTF8     | en_US.utf-8 | en_US.utf-8 | 
(3 rows)

postgres-# 
```


```sh
## show database
SELECT datname FROM pg_database;
```

### delete database

```
drop database databae-name;
```

### switch database
```sh
## 切换数据库
pgdb=# \c pgdb
You are now connected to database "pgdb" as user "postgres".
```

### tables

* Use the `\dt` or `\dt+` command in psql to show tables in a specific database.
* Use the `SELECT` statement to query table information from the `pg_catalog.pg_tables` catalog.

```sql
postgres=# \dt
          List of relations
 Schema |  Name   | Type  |  Owner   
--------+---------+-------+----------
 public | weather | table | postgres
(1 row)

postgres=# \dt+
                                     List of relations
 Schema |  Name   | Type  |  Owner   | Persistence | Access method |  Size   | Description 
--------+---------+-------+----------+-------------+---------------+---------+-------------
 public | weather | table | postgres | permanent   | heap          | 0 bytes | 
(1 row)
```

### privileges

```sql
# 查看用户的表权限
select * from information_schema.table_privileges 

# 查看usage权限表
select * from information_schema.usage_privileges where grantee='user_name';

#查看存储过程函数相关权限表
select * from information_schema.routine_privileges where grantee='user_name';

# 建用户授权
create user user_name;
alter user user_namewith password '';
alter user user_namewith CONNECTION LIMIT  20;#连接数限制


# 
select * from INFORMATION_SCHEMA.role_table_grants
# or
select * from information_schema.table_privileges
```

## 参考
* [postgresql sql commands](https://www.postgresql.org/docs/current/sql-commands.html)



## operator

POSTGRES_USER=$(kubectl -n component get secrets pg-db-component-pguser-secret -o jsonpath="{.data.username}" | base64 --decode)

POSTGRES_PASSWORD=$(kubectl -n component get secrets pg-db-component-pguser-secret -o jsonpath="{.data.password}" | base64 --decode)
