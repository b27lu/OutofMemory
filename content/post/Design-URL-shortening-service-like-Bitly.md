---
title: "设计类似Bit.ly的短网址服务 - Design URL Shortening Service Like Bitly"
date: 2018-11-04T21:10:13-05:00
draft: true
---

### Use Cases

1. 用户输入原网址，系统返回短网址（长度越短越好，比如6位字符）
2. 用户访问短网址，系统重定向至原网址

### Requirements

#### Functional Requirements:

1. 给定任意URL，系统生成唯一的短网址
2. 用户访问短网址，系统重定向至原网址
3. （optional）用户可以自定义短网址
4. （optional）短网址有默认的expiration, 用户也可以自定义expiration

#### Non-Functional Requirements:

1. high availability 高可用
2. realtime redirect
3. 生成的短网址是随机的 not predictable
4. (optional) statistics

### Assumption and Capacity Estimation

- read  heavy (read: 访问短网址；write：生成短网址)    read:write ratio = 100 : 1
- Assume 500M write per month, then 100 * 500M = 50B read per month
- QPS 读 50B / (30*24*3600) ~= 20K per sec 写 500M / (30*24*3600) ~= 200 per sec
- 存储 storage （假设5年，每条记录500 byte）500M * 12 * 5 * 500byte = 15TB 
- 内存 memory 根据80-20 原则，假设每天缓存20% URL需要 20% ＊ 20K per sec * 3600 * 24 * 500 byte ~= 172GB

### Database Design

#### SQL or NOSQL?

Answering questions below will help you make decison.

Question | Answer | Note
--- | --- | ---
Does it need to support transactions? | **No** | NoSQL does not support transaction.
Do we need rich SQL query? | **No** | NoSQL does not support as many queries as SQL, eg. join
Do we need to use AUTO_INCREMENT ID? | **Yes** | NoSQL couldn’t do this. It only has a global unique Object_id.
Does the system has a high requirement for QPS? | **No** | NoSQL has high performance, eg. Memcached’s QPS could reach million level, MondoDB does 10K level, MySQL only supports K level.
How high is the system’s scalability? | **No** | SQL requires developers write their codes to scale, while NoSQL comes with them (sharding, replica).

facts: billions of records, read heavy, not relational data, each record is less than 1KB

选择NOSQL database

    URL table
    hash: varchar (Primary Key)
    originalURL: varchar
    creationDate: datetime
    expirationDate: datetime
    UserID: int

    User table
    userID: int (primary Key)
    Name: varchar
    Email: varchar
    CreationDate: datetime
    LastLogin: datetime

### High Level Design and Algorithm

How to generate short and unique key for a given URL?

#### 方法1: using hash function to calculate unique key 

We can use hash function like `md5` or `sha1` to calculate the hash value of given URLs and take the first 6 characters as the short URL.

For example, using `md5` to get the hash of wikipedia

>`https://www.wikipedia.org -> 8d1d8a3956ab5668e1da4e3e7f856f08`

and then we take the first 6 characters `8d1d8a` as the unique key.

**Pros:** simple and straightforward 

**Cons:** hash collisions. Any hash algorithm could have inevitable conflicts. In our case, chances of hash collision are even higher since we only take first 6 characters.

Is there any way to mitigate it? 

Yes and no. Because the total valid long URLs are much more than total hash value we can generate from any hash functions, hash collision is inevitable. 

However, considering our service only shorten URLs that are requested by customers and it's a much smaller subset of total valid URLs, we can append time stamp to the original URL and re-calculate hash value until we find a unique hash value. (Time stamp will get changed when we re-calculate, so we will have different hash value)


#### 方法2：给每个URL分配一个唯一的key

创建一个key generation service 离线生成大量unique key，对于每个给定的URL分配一个未使用的key，并把这个key标记为已使用。

如何生成大量唯一的key？可以用relational database的auto increment id,然后把id编码为key。
concurrency 的问题怎么解决？预先生成key，分配key时加锁。application server缓存若干key，这些key可以在实际使用之前就标记为已使用。

### REST API 

Only one service: URLService

Core (Business Logic) Layer:
Class: URLService
Interface:
URLService.encode(String long_url)
URLService.decode(String short_url)

GET: /{short_URL}
return http redirect response 301

POST : 
goo.gl做法： POST: https://goo.gl/api/shorten 
bit.ly做法： POST: https://bitly.com/data/shorten
Request Body: {url=longUrl} e.g. {“longUrl”: “http://www.google.com/”}
Return OK(200), short_url is included in the data

-----

K: Data Access

K数据存取
第一步：select 选存储结构 -> 内存 or 文件系统 or 数据库 -> SQL or NoSQL?
第二步：schema 细化数据表


1. base62
Take short_url as a 62 base notation. 6 bits could represent 62^6=57 billion.
Each short_url represent a decimal digit. It could be the auto_increment_id in SQL database.

not thread safe
```java
public class URLService {
    HashMap<String, Integer> ltos;
    HashMap<Integer, String> stol;
    static int COUNTER;
    String elements;
    URLService() {
        ltos = new HashMap<String, Integer>();
        stol = new HashMap<Integer, String>();
        COUNTER = 1;
        elements = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
    }
    public String longToShort(String url) {
        String shorturl = base10ToBase62(COUNTER);
        ltos.put(url, COUNTER);
        stol.put(COUNTER, url);
        COUNTER++;
        return "http://tiny.url/" + shorturl;
    }
    public String shortToLong(String url) {
        url = url.substring("http://tiny.url/".length());
        int n = base62ToBase10(url);
        return stol.get(n);
    }

    public int base62ToBase10(String s) {
        int n = 0;
        for (int i = 0; i < s.length(); i++) {
            n = n * 62 + convert(s.charAt(i));
        }
        return n;

    }
    public int convert(char c) {
        if (c >= '0' && c <= '9')
            return c - '0';
        if (c >= 'a' && c <= 'z') {
            return c - 'a' + 10;
        }
        if (c >= 'A' && c <= 'Z') {
            return c - 'A' + 36;
        }
        return -1;
    }
    public String base10ToBase62(int n) {
        StringBuilder sb = new StringBuilder();
        while (n != 0) {
            sb.insert(0, elements.charAt(n % 62));
            n /= 62;
        }
        while (sb.length() != 6) {
            sb.insert(0, '0');
        }
        return sb.toString();
    }
}
```

第二步：schema 细化数据表
一个表:两列(id,long_url),其中id为主键(自带index)，long_url将其index,这样一张表可以双向查！

#### Database table schema

We need one table `url_mapping` which has two columns `(id, long_url)`. The id the primary key and is auto increased integer. The `long_url` is of type varchar and we create an index on it so that we can search long_url quickly as well. (The primary key column will have index automatically created by database). The table is like example below.

id | long_url
--|--
123456 | www.google.com
123457 | www.youtube.com

基本的系统架构是:
浏览器 <-> Web <-> Core <-> DB

O: optimize
> How to improve the response speed?
> Improve the response speed between web server and database
Use Memcached to improve response speed. When getting long_url, search in the cache first, then database. We could put 90% read request on the cache.

> Improve the response speed between web server and user’s browser
Different locations use different web server and cache server. All the areas share a DB used to match the users to the closest web server (through DNS) when they have a miss on the cache.

> What if we need one more MySQL machine?
> Issues:
running out of cache
More and more write requests
More and more cache misses
> Solutions:
Database Partitioning

1. Vertical sharding 
2. Horizontal sharding
The best way is horizontal sharding.

Currently table structure is (id, long_url). So, which column should be sharding key?

An easy way is id modulo sharding.

Here comes another question: How could multiple machines share a global auto_increment_id?

Two ways: 1. use one more machine to maintain the id. 2. use zookeeper. Both suck.

So, we do not use global auto_increment_id.

The pro way is put the sharding key as the first byte of the short_url.

Another way is to use consistent hashing to break the cycle into 62 pieces. It doesn’t matter how many pieces because there probably would not be over 62 machines (it could be 360 or whatever). Each machine is responsible for the service in the part of the cycle.

write long_url -> hash(long_url)%62 -> put long_url to the specific machine according to hash value -> generate short_url on this machine -> return short_url

short_url request -> get the sharding key (first byte of the short_url) -> search in the corresponding machine based on sharding key -> return long_url

Each time we add a new machine, put half of the range of the most used machine to the new machine.

假如一台MySQL 存不下/忙不过 了怎么办？
面临问题：
Cache资源不够
写操作越来越多
越来越多的cache miss率
怎么做：
拆数据库。
拆数据库有两种，一种是把不同的表放到不同的机器(vertical sharding)，另一种是把数据散列到不同的机器(horizontal)。
最好用的是horizontal sharding。
当前的表结构是:(id, long_url)，既需要用id查long_url，也需要用long_url查id，如何分，把哪列作为sharding key呢？
一个简单可行的办法是，按id取模sharding，因为读(短到长)的需求是主要的；写的时候就广播给所有机器，由于机器不会太多，也是可行的。
此时一个新的问题来了，n台机器如何共享一个全局自增id?
两个办法：开一台新的机器专门维护这个全局自增id，或者用zookeeper。都不好。
所以我们不用全局自增id。
业内的做法是，把sharding key作为第一位直接放到short_url里。这样就不需要全局自增id,每台机器自增就好了。
用consistent hashing将环分为62份(这个无所谓，因为预估机器不会超过这个数目，也可以设成360或者别的数，每次新加一个机器可以把区间最大的分一半)每个机器在环上负责一段区间。
具体做法：
新来一个long_url -> hash(long_url)%62 -> 把long_url放到hash value对应的机器里 -> 在这台机器上生成short_url -> 返回short_url
来一个short_url请求 -> 提取short_url的第一位得到sharding key -> 到sharding key对应的机器里找 -> 返回long_url
新增一台机器 -> 找原来机器里负责range(0-61)最大的机器 -> 将其range减半 -> 把一半放到新增机器上

> More Optimization
Put Chinese DB in China, American DB in the United States. Use geographical information as the sharding key, e.g. 0 for Chinese websites, 1 for American websites.

加一个新功能custom url怎么做？
单独建一张表，存custom_url <--> long_url
当查询时，先从custom表里查，再从url表里查。
注意，千万不要在url表里插一列custom，这样这列大部分的值为空。