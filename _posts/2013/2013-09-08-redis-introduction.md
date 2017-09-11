---
layout: post
comments: true
title: "redis简报"
categories: ["redis", "nosql"]
---

**RE**mote **DI**ctionary **S**erver(Redis) 是一个由Salvatore Sanfilippo写的key-value存储系统，
与memcached相比，redis支持更丰富的数据结构，特点是高性能、持久存储，适应高并发的应用场景。它起步较晚，发展迅速，
目前已被许多大型机构采用，比如Twitter、Github、新浪微博等。

## Redis安装

redis的安装没有其他依赖，非常简单。操作如下：

```bash
wget http://download.redis.io/releases/redis-2.6.16.tar.gz
tar zxvf redis-2.6.16.tar.gz
cd redis-2.6.16/
make
```

这样就会在src目录下生成以下几个可执行文件：redis-benchmark、redis-check-aof、redis-check-dump、redis-cli、redis-server。
这几个文件加上redis.conf就是redis的最终可用包了。可以考虑把这几个文件拷贝到你希望的地方。例如:

```bash
mkdir -p /usr/local/redis/bin
mkdir -p /usr/local/redis/etc
cp redis.conf /usr/local/redis/etc
cd src
cp redis-benchmark redis-check-aof redis-check-dump redis-cli redis-server /usr/local/redis/bin
```

现在就可以启动redis了。 

```bash
cd /usr/local/redis
bin/redis-server etc/redis.conf
```

注意，默认复制过去的redis.conf文件的daemonize参数为no，所以redis不会在后台运行，可以修改为yes则为后台运行redis。
另外配置文件中规定了pid文件，log文件和数据文件的地址，如果有需要先修改，默认log信息定向到stdout.
这时候就可以打开终端进行测试了，默认的监听端口是6379，所以用telnet进行连接如下：

```bash
# telnet localhost 6379
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
SET hello world
+OK
GET hello
$5
world
quit
+OK
Connection closed by foreign host.
```

或者使用redis-cli客户端:

```bash
# redis-cli
redis 127.0.0.1:6379> SET hello world
OK
redis 127.0.0.1:6379> GET hello
"world"
```

## redis数据结构

redis相对memcached来说，支持更加丰富的数据结构，正如作者所说的，redis是一个数据结构服务器（data structures server），
redis的所有功能就是将数据以其固有的几种结构保存，并提供给用户操作这几种结构的接口。redis目前支持以下几种数据类型：

* String类型

字符串是最简单的类型，和memcached支持的类型是一样的，但是在功能上更加丰富。

```bash
redis 127.0.0.1:6379> SET name "Redis 2.6.16"
OK
redis 127.0.0.1:6379> GET name
"Redis 2.6.16"
```

另外，它还支持批量读写:

```bash
redis 127.0.0.1:6379> MSET age 30 sex Male
OK
redis 127.0.0.1:6379> MGET age sex
1) "30"
2) "Male"
```

还可以当成数字来使用，并支持对数字的加减操作:

```bash
redis 127.0.0.1:6379> INCR age
(integer) 31
redis 127.0.0.1:6379> INCRBY age 2
(integer) 33
redis 127.0.0.1:6379> GET age
"33"
redis 127.0.0.1:6379> DECR age
(integer) 32
redis 127.0.0.1:6379> DECRBY age 2
(integer) 30
redis 127.0.0.1:6379> GET age
"30"
```

还支持对字符串进行部分修改或获取操作

```bash
redis 127.0.0.1:6379> STRLEN name
(integer) 12
redis 127.0.0.1:6379> GETRANGE name 0 4
"Redis"
redis 127.0.0.1:6379> APPEND name ", NoSQL"
(integer) 19
redis 127.0.0.1:6379> GET name
"Redis 2.6.16, NoSQL"
```

* List类型

Redis能够把数据存储成一个链表，并能对这个链表进行操作:

```bash
redis 127.0.0.1:6379> LPUSH language Java
(integer) 1
redis 127.0.0.1:6379> LPUSH language C++
(integer) 2
redis 127.0.0.1:6379> RPUSH language C
(integer) 3
redis 127.0.0.1:6379> LLEN language
(integer) 3
redis 127.0.0.1:6379> LRANGE language 0 2
1) "C++"
2) "Java"
3) "C"
redis 127.0.0.1:6379> LPOP language
"C++"
redis 127.0.0.1:6379> LLEN language
(integer) 2
redis 127.0.0.1:6379> LREM language 1 Java
(integer) 1
redis 127.0.0.1:6379> LLEN language
(integer) 1
```

Redis也支持很多修改操作

```bash
redis 127.0.0.1:6379> LRANGE language 0 2
1) "C"
redis 127.0.0.1:6379> LINSERT language BEFORE C C++
(integer) 2
redis 127.0.0.1:6379> LINSERT language BEFORE C Java
(integer) 3
redis 127.0.0.1:6379> LLEN language
(integer) 3
redis 127.0.0.1:6379> LRANGE language 0 2
1) "C++"
2) "Java"
3) "C"
redis 127.0.0.1:6379> LTRIM language 2 -1
OK
redis 127.0.0.1:6379> LLEN language
(integer) 1
redis 127.0.0.1:6379> LRANGE language 0 2
1) "C"
```

* Sets类型

Redis能够将一系列不重复的值存储成一个集合，并支持修改和集合关系操作。

```bash
redis 127.0.0.1:6379> SADD system Win
(integer) 1
redis 127.0.0.1:6379> SADD system Linux
(integer) 1
redis 127.0.0.1:6379> SADD system Mac
(integer) 1
redis 127.0.0.1:6379> SADD system Linux
(integer) 0
redis 127.0.0.1:6379> SMEMBERS system
1) "Win"
2) "Mac"
3) "Linux"
```

Sets结构也支持相应的修改操作

```bash
redis 127.0.0.1:6379> SREM system Win
(integer) 1
redis 127.0.0.1:6379> SMEMBERS system
1) "Mac"
2) "Linux"
redis 127.0.0.1:6379> SADD system Win
(integer) 1
redis 127.0.0.1:6379> SMEMBERS system
1) "Mac"
2) "Win"
3) "Linux"
```

Redis还支持对集合的子交并补等操作

```bash
redis 127.0.0.1:6379> SADD phone Android
(integer) 1
redis 127.0.0.1:6379> SADD phone Iphone
(integer) 1
redis 127.0.0.1:6379> SADD phone Win
(integer) 1
redis 127.0.0.1:6379> SMEMBERS phone
1) "Win"
2) "Iphone"
3) "Android"
redis 127.0.0.1:6379> SINTER system phone
1) "Win"
redis 127.0.0.1:6379> SUNION system phone
1) "Win"
2) "Iphone"
3) "Mac"
4) "Linux"
5) "Android"
redis 127.0.0.1:6379> SDIFF system phone
1) "Mac"
2) "Linux"
```

* Sorted Sets类型

Sorted Sets和Sets结构非常相似，不同的是Sorted Sets中的数据会有一个score属性，并会在写入时就按这个score排好序。

```bash
redis 127.0.0.1:6379> ZADD days 0 mon
(integer) 1
redis 127.0.0.1:6379> ZADD days 1 tue
(integer) 1
redis 127.0.0.1:6379> ZADD days 2 wed
(integer) 1
redis 127.0.0.1:6379> ZADD days 3 thu
(integer) 1
redis 127.0.0.1:6379> ZADD days 4 fri
(integer) 1
redis 127.0.0.1:6379> ZADD days 5 sat
(integer) 1
redis 127.0.0.1:6379> ZADD days 6 sun
(integer) 1
redis 127.0.0.1:6379> ZCARD days
(integer) 7
redis 127.0.0.1:6379> ZRANGE days 0 6
1) "mon"
2) "tue"
3) "wed"
4) "thu"
5) "fri"
6) "sat"
7) "sun"
redis 127.0.0.1:6379> ZSCORE days sat
"5"
redis 127.0.0.1:6379> ZCOUNT days 3 6
(integer) 4
redis 127.0.0.1:6379> ZRANGEBYSCORE days 3 6
1) "thu"
2) "fri"
3) "sat"
4) "sun"
```

* Hash类型

Redis能够存储多个键值对的数据

```bash
redis 127.0.0.1:6379> HMSET student name Tom age 12 sex Male
OK
redis 127.0.0.1:6379> HKEYS student
1) "name"
2) "age"
3) "sex"
redis 127.0.0.1:6379> HVALS student
1) "Tom"
2) "12"
3) "Male"
redis 127.0.0.1:6379> HGETALL student
1) "name"
2) "Tom"
3) "age"
4) "12"
5) "sex"
6) "Male"
redis 127.0.0.1:6379> HDEL student sex
(integer) 1
redis 127.0.0.1:6379> HGETALL student
1) "name"
2) "Tom"
3) "age"
4) "12"
```

Redis能够支持Hash的批量修改和获取

```bash
redis 127.0.0.1:6379> HMSET kid name Akshi age 2 sex Female
OK
redis 127.0.0.1:6379> HMGET kid name age sex
1) "Akshi"
2) "2"
3) "Female"
```

在这些数据结构的基础上，跟memcached一样，Redis也支持设置数据过期时间，并支持一些简单的组合型的命令。

设置数据过期时间

```bash
redis 127.0.0.1:6379> SET name "John Doe"
OK
redis 127.0.0.1:6379> EXISTS name
(integer) 1
redis 127.0.0.1:6379> EXPIRE name 5
(integer) 1
```

5秒后再查看

```
redis 127.0.0.1:6379> EXISTS name
(integer) 0
redis 127.0.0.1:6379> GET name
(nil)
```

简单的组合型的命令。通过MULTI和EXEC，将几个命令组合起来执行

```bash
redis 127.0.0.1:6379> SET counter 0
OK
redis 127.0.0.1:6379> MULTI
OK
redis 127.0.0.1:6379> INCR counter
QUEUED
redis 127.0.0.1:6379> INCR counter
QUEUED
redis 127.0.0.1:6379> INCR counter
QUEUED
redis 127.0.0.1:6379> EXEC
1) (integer) 1
2) (integer) 2
3) (integer) 3
redis 127.0.0.1:6379> GET counter
"3"
```

你还可以用DICARD命令来中断执行中的命令序列

```bash
redis 127.0.0.1:6379> SET newcounter 0
OK
redis 127.0.0.1:6379> MULTI
OK
redis 127.0.0.1:6379> INCR newcounter
QUEUED
redis 127.0.0.1:6379> INCR newcounter
QUEUED
redis 127.0.0.1:6379> INCR newcounter
QUEUED
redis 127.0.0.1:6379> DISCARD
OK
redis 127.0.0.1:6379> GET newcounter
"0"
```