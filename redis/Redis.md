##### Redis

##### Redis配置文件



##### Redis基础类型

> redis,是基于key-value的NoSql，既然是key-value的，那么它的数据类型也是key-value格式，在redis中，也可以通过help命令查看每一种类型的api

###### String类型

![1596082831512](.\img\1596082831512.png)

incr key : 自增  decr key ：自减

###### List类型

**在list中存储的键值对，可以添加重复元素，是有序的**

![1596083232796](.\img\1596083232796.png)

List类型，lpush从列表的头部添加元素，rpush从列表的尾部添加元素。

![1596083340180](.\img\1596083340180.png)

lpop 从列表的头部弹出第一个元素

###### HASH类型

**hash是一个string类型的field和value的映射表，hash特别适用于存储对象，存储的是不重复的，无序的**

![1596084034707](.\img\1596084034707.png)

###### SET类型

**Set是string类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。**

![1596084377307](.\img\1596084377307.png)

###### ZSET类型

**ZSET是一个有序集合，且不允许出现重复元素，自动有序，在添加元素的时候，通过设置的score来给元素排序**

![1596084771996](.\img\1596084771996.png)



![1596084872363](.\img\1596084872363.png)

###### Geospatial类型 地理位置

> 应用场景：附近的人
>
> 经纬度的规则：
>
> ​		有效经度为-180至180度
>
> ​		有效纬度为-85.05112878至85.05112878度

key  , 指定地理位置（经度，纬度） 成员

geoadd key 经度，维度  成员

```shell
#添加地理位置  geoadd 经度 维度 成员
#定位成员的经纬度  geopos key 成员
#计算两地距离  geodist  成员1 成员2 withdist(显示距离) withcoord(显示定位信息) count 数量
#计算某一位置范围内存在的数据（附近的人）  georadius key 经度 维度 范围 单位


127.0.0.1:6379> geoadd china 116.28 39.54 beijing  #添加北京的经纬度
(integer) 1
127.0.0.1:6379> geoadd china 117.10 39.10 tianjin
(integer) 1
127.0.0.1:6379> geoadd china 121.26 31.12 shanghai
(integer) 1
127.0.0.1:6379> geoadd china 91.02 29.39 lasa
(integer) 1
127.0.0.1:6379> geoadd china 113.18 23.10 guangzhou
(integer) 1
127.0.0.1:6379> geoadd china 121.31 25.02 taibei
(integer) 1
127.0.0.1:6379> geodist china beijing lasa   #计算北京到拉萨的距离
"2564794.3023"
127.0.0.1:6379> geodist china beijing lasa km  #计算北京到拉萨的距离 单位为km
"2564.7943"
127.0.0.1:6379>   
```

###### HyperLogLog 基数统计

>统计元素中不重复元素的个数

```shell
# 添加指定元素到key pfadd key 元素
# 计算key中基数个数 pfcount key
# 合并两个元素集合， pfmerge key  key1 key2
127.0.0.1:6379> pfadd key 1 
(integer) 1
127.0.0.1:6379> pfadd key 1
(integer) 0
127.0.0.1:6379> pfadd key 2
(integer) 1
127.0.0.1:6379> pfcount key
(integer) 2
```

###### Bitmaps 位图

位存储

setbit getbit

##### Springboot整合redis

引入依赖

```pom
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
```

添加配置

```xml
spring.redis.host=localhost
spring.redis.port=6379
```

编写配置类，RedisTemplate

```java
@Configuration
public class RedisConfig  {

    @Bean
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        //设置key的序列化方式
        template.setKeySerializer(RedisSerializer.string());
        // 设置value的序列化方式
        template.setValueSerializer(RedisSerializer.json());
        // 设置hash的key的序列化方式
        template.setHashKeySerializer(RedisSerializer.string());
        // 设置hash的value的序列化方式
        template.setHashValueSerializer(RedisSerializer.json());

        template.afterPropertiesSet();
        return template;
    }
}
```

利用redisTemplate的api操作redis

##### Redis的持久化操作

> redis是基于内存的数据库，既然是基于内存的，那必定会出现内存数据保存的情况，redis就提供了两种方式，对内存中的数据进行持久化。

###### RDB 

**快照，将内存中的数据，写入硬盘，那么redis既然是单线程的，这就意味着，不仅要响应用户的请求，还需要进行内存快照，不就影响了redis的效率了吗？要是进行持久化的时候，还没有保存完，就来了个请求，删除了数据，怎么办？**

**1.那看下RDB是怎么做的**

**redis在进行持久化的时候，使用操作系统的COW(Copy On Write)机制，调用fork()函数，复制一个当前的线程，也将父进程中内存的数据先复制下来，让子进程进行持久化**

**2.RDB又是什么时候被触发的呢？**

- **save命令**

- **bgsave命令**

- **redis配置自动触发**

  ![1596098159959](.\img\1596098159959.png)

3.**但是，如果子线程在进行持久化一个大的数据的时候，突然电脑关机了，那没有被持久化的数据，也会消失，这就出现了AOF**

###### AOF

**客户端每次对服务端进行写操作的时候，会将这些操作追加保存到.aof文件中，在redis服务器重启的时候，会加载这个aof文件中的命令，从而将数据持久化。**

**redis默认是不开启AOF的，可以通过以下配置开启AOF**

```xml
# 开启aof机制
appendonly yes

# aof文件名
appendfilename "appendonly.aof"

# 写入策略,always表示每个写操作都保存到aof文件中,
#everysec 过程与always相同，只是fsync的频率为1秒钟一次。这个是Redis默认配置，如果系统宕机，会丢失一秒
#左右的数据
#no 由操作系统决定什么时候从系统缓冲区刷新到硬盘
appendfsync always

# 默认不重写aof文件
no-appendfsync-on-rewrite no

# 保存目录
dir ./
```

**AOF开启后，会一直在aof文件中追加操作，如果不删除，aof文件会变得特变大，redis也提供了AOF重写**

**配置AOF重写的方式：**

​			  **1.通过配置redis的配置文件，将重写aof文件的命令开启：no-appendfsync-on-rewrite yes**

​				**2.客户端发送bgrewriteaof命令进行重写**

##### Redis事务

> 事务可以一次性执行多条命令,**redis的事务，不支持回滚，在执行多条命令的时候，期间有一条出现问题，是，其他执行的命令不受影响**

```shell
# 开启事务  multi 
# 执行命令
# 结束事务  exec
# 放弃事务  discrad
127.0.0.1:6379> set key1 1
OK
127.0.0.1:6379> set key2 a
OK
127.0.0.1:6379> set key3 2
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> incr key1
QUEUED
127.0.0.1:6379> incr key2
QUEUED
127.0.0.1:6379> incr key3
QUEUED
127.0.0.1:6379> exec
1) (integer) 2
2) (error) ERR value is not an integer or out of range
3) (integer) 3
127.0.0.1:6379> get key1
"2"
127.0.0.1:6379> get key2
"a"
127.0.0.1:6379> get key3
"3"
127.0.0.1:6379>   
```

##### Redis的锁

> **watch ,监视**

##### 缓存穿透

> 缓存穿透，查询一个数据库一定不存在的数据（比如通过一个一定不存在的id查询）,缓存中没有数据，就会向数据库中查询，但是查询后也是null,如果统一时间点，请求量巨大，所有的请求全不打在了数据库上，数据库就会宕机。**在解决的这个问题的时候，可以通过在缓存中设置一个null的值**

```java
		User user;
        if(isExist){
            logger.info("缓存中存在当前用户，从缓存中查找");
            user = (User) redisTemplate.opsForValue().get(key);
        }else {
            logger.info("缓存中没有存在当前用户，从数据库中查找，然后添加到缓存");
            user = userMapper.getUser(id);
            if(user!=null){
                redisTemplate.opsForValue().set(key,user);
            }else {
                //当数据库的数据为空时，添加一个null值在缓存中
                redisTemplate.opsForValue().set(key,null,60, TimeUnit.SECONDS);
            }
        }
```

**布隆过滤器**



##### 缓存雪崩

> 缓存中的数据，在同一时间段，失效，直至请求来的时候，直接打在数据库



##### 缓存击穿

> 热点key的问题，当这个热点key在失效时，瞬间请求打过来

##### Redis的发布订阅

##### Redis集群