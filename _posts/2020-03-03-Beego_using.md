---
layout: post
title: Beego In Use
tags: [go]
comments: true
---

## 时区问题

ORM 默认使用 time.Local 本地时区

作用于 ORM 自动创建的时间
从数据库中取回的时间转换成 ORM 本地时间
如果需要的话，你也可以进行更改

```
// 设置为 UTC 时间
orm.DefaultTimeLoc = time.UTC
```

ORM 在进行 RegisterDataBase 的同时，会获取数据库使用的时区，然后在 time.Time 类型存取时做相应转换，以匹配时间系统，从而保证时间不会出错。

注意:

鉴于 Sqlite3 的设计，存取默认都为 UTC 时间
使用 go-sql-driver 驱动时，请注意参数设置
从某一版本开始，驱动默认使用 UTC 时间，而非本地时间，所以请指定时区参数或者全部以 UTC 时间存取
例如：`root:root@/orm_test?charset=utf8&loc=Asia%2FShanghai`

注意设置相应时区时，要保证运行环境中存在指定的 时区文件。


参见 [loc](https://github.com/go-sql-driver/mysql#loc) / [parseTime](https://github.com/go-sql-driver/mysql#parsetime)


Refer to: https://beego.me/docs/mvc/model/orm.md#%E6%97%B6%E5%8C%BA%E8%AE%BE%E7%BD%AE
