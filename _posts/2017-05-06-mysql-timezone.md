---
layout: post
title: MySQL Time Zone
date: 2017-05-06 22:00:00 +0800
tags:
- mysql
---

最近遇到 MySQL 显示 timestamp 的问题，往一个表里插入条目或更新条目的时候都需要记录时间，记录的过程往往没有什么问题，但取数据进行展示的时候就会有问题了。

像 Facebook 这样的大公司，为了容灾，数据会存储在多个地区，身处 🇬🇧 的 Alice 发布了一条状态，会在欧洲和北美的数据中心都存储一份。为了用户体验，Alice 🇺🇸 的朋友 Bob 在阅读这条状态的时候，应该将时间显示为美国时区。

**考虑两种情况：**

1️⃣ Facebook 欧洲和北美数据中心的机器都将时区设置为 UTC，这样当使用 MySQL 的 select 语句进行查询的时候显示的字符串都为 UTC 时间，不同地区的用户在显示的时候可以将这个时间字符串转换为当地的时间。

2️⃣ Facebook 在欧洲的机器和北美的机器时区不同，都为当地时区，那么当使用 select 语句进行查询的时候显示的都是当地时间。那么，假设北美的数据中心挂了，那么身处 🇺🇸 的 Bob 获取到的是欧洲数据中心的数据，这时候如果不对数据进行处理就会显示错误的时间。

这种情况可以使用 MySQL 的 `CONVERT_TZ(dt, from_tz, to_tz)` 进行处理：

```
mysql> select * from app;
+----+----------+---------------+----------+---------------------+---------------------+
| id | name     | display_name  | platform | create_date         | update_date         |
+----+----------+---------------+----------+---------------------+---------------------+
|  1 | testApp1 | I am test 1   | ISO      | 2017-05-10 05:49:42 | 2017-05-10 05:49:42 |
|  2 | testApp2 | I am test 2   | ANDROID  | 2017-05-10 05:49:42 | 2017-05-10 05:49:42 |
|  3 | testApp3 | I am test 3   | STB      | 2017-05-10 05:49:42 | 2017-05-10 05:49:42 |
+----+----------+---------------+----------+---------------------+---------------------+
4 rows in set (0.00 sec)
mysql> select id, name, display_name, platform, CONVERT_TZ(create_date, @@global.time_zone, '+1:00') as create_date from app;
+----+----------+---------------+----------+---------------------+
| id | name     | display_name  | platform | create_date         |
+----+----------+---------------+----------+---------------------+
|  1 | testApp1 | I am test 1   | ISO      | 2017-05-10 13:49:42 |
|  2 | testApp2 | I am test 2   | ANDROID  | 2017-05-10 13:49:42 |
|  3 | testApp3 | I am test 3   | STB      | 2017-05-10 13:49:42 |
+----+----------+---------------+----------+---------------------+
4 rows in set (0.01 sec)
```

这就需要后台程序根据用户的时区创建包含 CONVERT_TZ 的 SQL 语句进行查询，然后将结果返回给前端。

当然 Facebook 肯定不会使用我描述的方法，这里只是举例进行时区的说明。

**Tips:**

MySQL 存储在数据库中的数据是时间戳（自格林尼治时间1970-01-01零点至今的秒数），只不过 select 时显示是根据时区进行转化的。

设置 MySQL 的时区有两种方式：

设置机器时区:

```shell
timedatectl list-timezones
timedatectl set-timezone UTC
```

设置 MySQL 时间戳:

```
mysql> SET GLOBAL time_zone = '+00:00';
```

<br>
<span class="post-meta">
Reference:
</span>
<br>
<span class="post-meta">
1 [CONVERT_TZ][ref1]<br>
2 [MySQL Server Time Zone Support][ref2]<br>
3 [How do I set the time zone of MySQL?][ref3]
</span>

[ref1]: https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_convert-tz
[ref2]: https://dev.mysql.com/doc/refman/5.7/en//time-zone-support.html
[ref3]: http://stackoverflow.com/questions/930900/how-do-i-set-the-time-zone-of-mysql
