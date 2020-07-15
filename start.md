### 1.Shiro-Redis

[View on GitHub](https://github.com/alexxiyang/shiro-redis)

###### 1.请使用GitHub进行访问

###### 2.shiro-redis简介



​	shiro only provide the support of ehcache and  concurrentHashMap. Here is an implement of redis cache can be used by  	shiro. Hope it will help you!  

​	shiro仅仅提供一个ehcache以及一个高速缓存(高并发的一个内存)的支持,这里由一个有redis缓存的实现能够让你使用Shiro，希望能够帮到你.   

3. ###### 下载地址

​           [*download* .ZIP](https://github.com/alexxiyang/shiro-redis/zipball/master)[*download* .TGZ](https://github.com/alexxiyang/shiro-redis/tarball/master)                  



### 2.shiro-redis

[![Build Status](https://travis-ci.org/alexxiyang/shiro-redis.svg?branch=master)](https://travis-ci.org/alexxiyang/shiro-redis) [![Maven Central](https://maven-badges.herokuapp.com/maven-central/org.crazycake/shiro-redis/badge.svg)](https://maven-badges.herokuapp.com/maven-central/org.crazycake/shiro-redis)

shiro only provide the support of ehcache and concurrentHashMap. Here is an implement of redis cache can be used by shiro. Hope it will help  you!

当前maven尝鲜版3.2.3

### 3.Download

You can choose these 2 ways to include shiro-redis into your project

- use “git clone https://github.com/alexxiyang/shiro-redis.git” to  clone project to your local workspace and build jar file by your self
- 你可以选择两种方式,一种是git,一种是当地的jar依赖
- 或者maven
- add maven dependency

```
<dependency>
    <groupId>org.crazycake</groupId>
    <artifactId>shiro-redis</artifactId>
    <version>3.2.3</version>
</dependency>
```

> **Note:**\ Do not use version < 3.1.0\ **注意**：\ 请不要使用3.1.0以下版本

### 4.Before use

使用之前干什么？

Here is the first thing you need to know. `Shiro-redis` needs an id  field to identify your authorization object in `Redis`. So please make  sure your principal class has a field which you can get unique id of  this object. Please setting this id field name by `cacheManager.principalIdFieldName = <your id field name of principal object>`

在这里你需要知道的第一件事情,`Shiro-redis`需要一个id字段在`Redis`中对你的授权对象进行身份标识,所以请确保你的principal 类具有你能唯一标识这个对象的一个字段,通过`cacheManager.principalIdFieldName` 设置这个id字段名 为你的principal对象的id字段名

For example:

If you create `SimpleAuthenticationInfo` like the following:

```
@Override
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
    UsernamePasswordToken usernamePasswordToken = (UsernamePasswordToken)token;
    UserInfo userInfo = new UserInfo();
    userInfo.setUsername(usernamePasswordToken.getUsername());
    return new SimpleAuthenticationInfo(userInfo, "123456", getName());
}
```

Then the userInfo object is your principal object. You need to make sure `UserInfo` has an unique field to identify it in Redis. Take userId as an example:

那么这个userInfo对象是你的首选对象(能够最精确标识你用户信息的一个对象),你需要确保UserInfo有一个唯一的字段去在Redis中标识它,那么这个举个例子:

```
public class UserInfo implements Serializable{
//注意  需要序列化

    private Integer userId

    private String username;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public Integer getUserId() {
        return this.userId;
    }
}
```

Put `userId` as the value of `cacheManager.principalIdFieldName`, like this:

给`cachemanage.principalIdFieldName`赋值

```java
cacheManager.principalIdFieldName = userId
```

If you’re using Spring, the configuration should be

如果你需要使用spring,那这个配置就应该是Bean定义的方式：作为一个手动注入的属性

```
<property name="principalIdFieldName" value="userId" />
```

Then `shiro-redis` will call `userInfo.getUserId()` to get the id for storing `Redis` object.

这个框架将会调用这个前面提到的`AuthenticationInfo` 获取这个id在`Redis`(进行存储)

### 5.How to configure ?

You can configure `shiro-redis` either in `shiro.ini` or in `spring-*.xml`

你能够很简单的配置它

`shiro.ini`

Here is the configuration for `shiro.ini`.

`Redis` Standalone

`Redis` 单机应用

```java
[main]
#====================================
# shiro-redis configuration [start]
#====================================

#===================================
# Redis Manager [start]
#===================================

# Create redisManager
redisManager = org.crazycake.shiro.RedisManager

//可以是远程主机
# Redis host. If you don't specify host the default value is 127.0.0.1:6379
redisManager.host = 127.0.0.1:6379

#===================================
# Redis Manager [end]
#===================================

Redis session 数据访问层
#=========================================
# Redis session DAO [start]
#=========================================

# Create redisSessionDAO
redisSessionDAO = org.crazycake.shiro.RedisSessionDAO

# Use redisManager as cache manager
redisSessionDAO.redisManager = $redisManager

#shiro的web应用必配
sessionManager = org.apache.shiro.web.session.mgt.DefaultWebSessionManager

sessionManager.sessionDAO = $redisSessionDAO

securityManager.sessionManager = $sessionManager

#=========================================
# Redis session DAO [end]
#=========================================

#==========================================
# Redis cache manager [start]
#==========================================

# Create cacheManager
cacheManager = org.crazycake.shiro.RedisCacheManager

# Principal id field name. The field which you can get unique id to identify this principal.
# For example, if you use UserInfo as Principal class, the id field maybe `id`, `userId`, `email`, etc.
# Remember to add getter to this id field. For example, `getId()`, `getUserId()`, `getEmail()`, etc.
# Default value is id, that means your principal object must has a method called `getId()`
#

首选对象的ID字段名,  能够唯一表示这个对象的身份.
举个例子,如果你使用UserINFO作为 Principal class,这个字段可能是ID,USERID,email,等等.
记得需要为这个字段添加一个getter方法
默认值是id,这就意味着你的首选对象必须要有一个getter方法 能够被调用,名称是getId()
cacheManager.principalIdFieldName = id

# Use redisManager as cache manager
cacheManager.redisManager = $redisManager

#要将缓存管理器交给安全管理器
securityManager.cacheManager = $cacheManager

#==========================================
# Redis cache manager [end]
#==========================================

#=================================
# shiro-redis configuration [end]
#=================================
```

For complete configurable options list, check [Configurable Options](http://alexxiyang.github.io/shiro-redis/#configurable-options).

对于完整信息配置,请点击这个链接.

Here is a [tutorial project](https://github.com/alexxiyang/shiro-redis-tutorial) for you to understand how to configure `shiro-redis` in `shiro.ini`.

这有一个指南项目能够帮你理解如果在Shiro-redis中配置shiro.ini

Redis Sentinel

redis的哨兵

if you’re using Redis Sentinel, please change the redisManager configuration into the following:

如果你使用了Redis的哨兵机制,请改变这个redisManager的配置,如下所述:

```
#===================================
# Redis Manager [start]
#===================================

# Create redisManager
redisManager = org.crazycake.shiro.RedisSentinelManager

#填充哨兵所在主机位置 IP加端口
# Sentinel host. If you don't specify host the default value is 127.0.0.1:26379,127.0.0.1:26380,127.0.0.1:26381
#如果不指定就会有一个默认值,依次是......
redisManager.host = 127.0.0.1:26379,127.0.0.1:26380,127.0.0.1:26381

#哨兵服务器的名称
# Sentinel master name
redisManager.masterName = mymaster

#===================================
# Redis Manager [end]
#===================================
```

For complete configurable options list, check [Configurable Options](http://alexxiyang.github.io/shiro-redis/#configurable-options).

Redis Cluster

redis的分布式配置

If you’re using redis cluster, here is an example of configuration :

如果你使用了redis分布式配置,这有一个例子:

```
#===================================
# Redis Manager [start]
#===================================

#必须改变管理为分布式的管理器
# Create redisManager
redisManager = org.crazycake.shiro.RedisClusterManager

#Redis集群配置
# Redis host and port list
redisManager.host = 192.168.21.3:7000,192.168.21.3:7001,192.168.21.3:7002,192.168.21.3:7003,192.168.21.3:7004,192.168.21.3:7005

#===================================
# Redis Manager [end]
#===================================
```

For complete configurable options list, check [Configurable Options](http://alexxiyang.github.io/shiro-redis/#configurable-options).



对于spring,这是我们希望所看到的:

​		如何配置:

### 6.Spring

Redis Standalone

redis单机版:

spring.xml:

```
<!-- shiro-redis configuration [start] -->

<!-- Redis Manager [start] -->
<bean id="redisManager" class="org.crazycake.shiro.RedisManager">
    <property name="host" value="127.0.0.1:6379"/>
</bean>
<!-- Redis Manager [end] -->

<!-- Redis session DAO [start] -->
<bean id="redisSessionDAO" class="org.crazycake.shiro.RedisSessionDAO">
    <property name="redisManager" ref="redisManager" />
</bean>
<bean id="sessionManager" class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager">
    <property name="sessionDAO" ref="redisSessionDAO" />
</bean>
<!-- Redis session DAO [end] -->

<!-- Redis cache manager [start] -->
<bean id="cacheManager" class="org.crazycake.shiro.RedisCacheManager">
    <property name="redisManager" ref="redisManager" />
</bean>
<!-- Redis cache manager [end] -->

<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <property name="sessionManager" ref="sessionManager" />
    <property name="cacheManager" ref="cacheManager" />

    <!-- other configurations -->
    <property name="realm" ref="exampleRealm"/>
    <!--管理器密钥-->
    <property name="rememberMeManager.cipherKey" value="kPH+bIxk5D2deZiIxcaaaA==" />
</bean>

<!-- shiro-redis configuration [end] -->
```

For complete configurable options list, check [Configurable Options](http://alexxiyang.github.io/shiro-redis/#configurable-options).

Here is a [tutorial project](https://github.com/alexxiyang/shiro-redis-spring-tutorial) for you to understand how to configure `shiro-redis` in spring configuration file.

Redis Sentinel

Redis哨兵

If you use redis sentinel, here is an example of configuration :

```
<!-- shiro-redis configuration [start] -->
<!-- shiro redisManager -->
<bean id="redisManager" class="org.crazycake.shiro.RedisSentinelManager">
    <property name="host" value="127.0.0.1:26379,127.0.0.1:26380,127.0.0.1:26381"/>
    <property name="masterName" value="mymaster"/>
</bean>
```

For complete configurable options list, check [Configurable Options](http://alexxiyang.github.io/shiro-redis/#configurable-options).

Redis Cluster

If you use redis cluster, here is an example of configuration :

```java
<!-- shiro-redis configuration [start] -->
<!-- shiro redisManager -->
<bean id="redisManager" class="org.crazycake.shiro.RedisClusterManager">
    <property name="host" value="192.168.21.3:7000,192.168.21.3:7001,192.168.21.3:7002,192.168.21.3:7003,192.168.21.3:7004,192.168.21.3:7005"/>
</bean>
```

For complete configurable options list, check [Configurable Options](http://alexxiyang.github.io/shiro-redis/#configurable-options).

### 7.Serializer



#序列化

Since redis only accept `byte[]`, there comes to a serializer problem. Shiro-redis is using StringSerializer as key serializer and ObjectSerializer as value serializer. You can use your own custom serializer, as long as this custom serializer implemens `org.crazycake.shiro.serializer.RedisSerializer`



自从redis只接受一个byte数组的开始,这就产生了一个序列化问题,Shiro-redis使用StringSerializer作为key的序列化以及使用ObjectSerializer作为值的序列化,你能够使用你自己的序列化器,同时这个自定义的序列化器需要实现RedisSerializer

​	我们一般常用的是JACKSON2JSON...Serializer



For example, let’s change the charset of keySerializer.(改变这个序列化的字符集编码,是为了支持中文)

```
# If you want change charset of keySerializer or use your own custom serializer, you need to define serializer first
#
# cacheManagerKeySerializer = org.crazycake.shiro.serializer.StringSerializer

# Supported encodings refer to https://docs.oracle.com/javase/8/docs/technotes/guides/intl/encoding.doc.html
# UTF-8, UTF-16, UTF-32, ISO-8859-1, GBK, Big5, etc
#
# cacheManagerKeySerializer.charset = UTF-8

# cacheManager.keySerializer = $cacheManagerKeySerializer
```

These 4 Serializers are replaceable:

- cacheManager.keySerializer
- cacheManager.valueSerializer
- redisSessionDAO.keySerializer
- redisSessionDAO.valueSerializer

### 8.Configurable Options

配置选项:

### 9.RedisManager(redis的管理器)

| Title           | Default                                     | Description                                                  |
| :-------------- | :------------------------------------------ | :----------------------------------------------------------- |
| host            | `127.0.0.1:6379`                            | Redis host. If you don’t specify host the default value is `127.0.0.1:6379`. If you run redis in sentinel mode or cluster mode, separate host names with comma, like `127.0.0.1:26379,127.0.0.1:26380,127.0.0.1:26381` |
| masterName      | `mymaster`                                  | **Only used for sentinel mode** The master node of Redis sentinel mode 只有在哨兵模式下才需要配置 |
| timeout         | `2000`                                      | Redis connect timeout. Timeout for jedis try to connect to redis server(In milliseconds) 单位毫秒 |
| soTimeout       | `2000`                                      | **Only used for sentinel mode or cluster mode** The timeout for jedis try to read data from redis server  仅仅在哨兵模式或者分布式模式下,这个jedis会尝试从redis服务器中读取数据的一个超时时间. |
| maxAttempts     | `3`                                         | **Only used for cluster mode** Max attempts to connect to server  尝试连接redis的次数 |
| password        |                                             | Redis password                                               |
| database        | `0`                                         | Redis database. Default value is 0                           |
| jedisPoolConfig | `new redis.clients.jedis.JedisPoolConfig()` | JedisPoolConfig. You can create your own JedisPoolConfig instance and set attributes as you wish Most of time, you don’t need to set jedisPoolConfig Here is an example. `jedisPoolConfig = redis.clients.jedis.JedisPoolConfig` `jedisPoolConfig.testWhileIdle = false` `redisManager.jedisPoolConfig = jedisPoolConfig` |
| count           | `100`                                       | Scan count. Shiro-redis use Scan to get keys, so you can define the number of elements returned at every iteration. 你能够限制返回数据的个数 |
| jedisPool       | `null`                                      | **Only used for sentinel mode or single mode** You can create your own JedisPool instance and set attributes as you wish |



### 10.RedisSessionDAO(redis的数据访问层)

| Title                  | Default                                           | Description                                                  |
| :--------------------- | :------------------------------------------------ | :----------------------------------------------------------- |
| redisManager           |                                                   | RedisManager which you just configured above (Required)      |
| expire                 | `-2`                                              | Redis cache key/value expire time. The expire time is in second. Special values: `-1`: no expire `-2`: the same timeout with session Default value: `-2` **Note**: Make sure expire time is longer than session timeout.  设置存储值的过期时间 |
| keyPrefix              | `shiro:session:`                                  | Custom your redis key prefix for session management **Note**: Remember to add colon at the end of prefix.  在存储值之前需要加入的key前缀 |
| sessionInMemoryTimeout | `1000`                                            | When we do signin, `doReadSession(sessionId)` will be called by shiro about 10 times. So shiro-redis save Session in  ThreadLocal to remit(缓解) this problem. sessionInMemoryTimeout is expiration  of Session in ThreadLocal.  Most of time, you don’t need to change it. 默认是不需要改变的. |
| sessionInMemoryEnabled | `true`                                            | Whether or not enable temporary save session in ThreadLocal 是否需要支持在ThreadLocal临时保存session |
| keySerializer          | `org.crazycake.shiro.serializer.StringSerializer` | The key serializer of cache manager You can change the implement of key serializer or the encoding of StringSerializer. Supported encodings refer to [Supported Encodings](https://docs.oracle.com/javase/8/docs/technotes/guides/intl/encoding.doc.html). Such as `UTF-8`, `UTF-16`, `UTF-32`, `ISO-8859-1`, `GBK`, `Big5`, etc For more detail, check [Serializer](http://alexxiyang.github.io/shiro-redis/#serializer) |
| valueSerializer        | `org.crazycake.shiro.serializer.ObjectSerializer` | The value serializer of cache manager You can change the implement of value serializer For more detail, check [Serializer](http://alexxiyang.github.io/shiro-redis/#serializer) |

### 11.CacheManager(缓存管理器)

| Title                | Default                                           | Description                                                  |
| :------------------- | :------------------------------------------------ | :----------------------------------------------------------- |
| redisManager         |                                                   | RedisManager which you just configured above (Required)      |
| principalIdFieldName | `id`                                              | Principal id field name. The field which you can get unique id to identify this principal. For example, if you use UserInfo as Principal class, the id field maybe `id`, `userId`, `email`, etc. Remember to add getter to this id field. For example, `getId()`, `getUserId(`), `getEmail()`, etc. Default value is `id`, that means your principal object must has a method called `getId()` |
| expire               | `1800`                                            | Redis cache key/value expire time.  The expire time is in second. |
| keyPrefix            | `shiro:cache:`                                    | Custom your redis key prefix for cache management **Note**: Remember to add colon at the end of prefix. |
| keySerializer        | `org.crazycake.shiro.serializer.StringSerializer` | The key serializer of cache manager You can change the implement of key serializer or the encoding of StringSerializer. Supported encodings refer to [Supported Encodings](https://docs.oracle.com/javase/8/docs/technotes/guides/intl/encoding.doc.html). Such as `UTF-8`, `UTF-16`, `UTF-32`, `ISO-8859-1`, `GBK`, `Big5`, etc For more detail, check [Serializer](http://alexxiyang.github.io/shiro-redis/#serializer) |
| valueSerializer      | `org.crazycake.shiro.serializer.ObjectSerializer` | The value serializer of cache manager You can change the implement of value serializer For more detail, check [Serializer](http://alexxiyang.github.io/shiro-redis/#serializer) |

对于支持Spring boot 那是最想要的吧?

### 12.Spring boot starter

Shiro-redis’s Spring-Boot integration is the easiest way to integrate Shiro-redis into a Spring-base application.

> Note: `shiro-redis-spring-boot-starter` version `3.2.1` is based on `shiro-spring-boot-web-starter` version `1.4.0-RC2`

对于和springboot集成最简单的方式是集成Shiro-redis作为一个spring的基础程序(应用)

First include the Shiro-redis Spring boot starter dependency in you application classpath

导入依赖

```
<dependency>
    <groupId>org.crazycake</groupId>
    <artifactId>shiro-redis-spring-boot-starter</artifactId>
    <version>3.2.1</version>
</dependency>
```

The next step depends on whether you’ve created your own `SessionManager` or `SessionsSecurityManager`. Because `shiro-redis-spring-boot-starter` will create `RedisSessionDAO` and `RedisCacheManager` for you. Then inject them into `SessionManager` and `SessionsSecurityManager` automatically.

第二步依赖于你是否需要创建你自己的SessionManager或者一个SessionSecurityManager.因为Shiro-redis-spring-boot-starter 将会创建一个RedisSessionDao和一个RedisCacheManager给你,它们会被自动注入SessionManager以及SessionsSecurityManager 

But if you’ve created your own `SessionManager` or `SessionsSecurityManager` as below:

如果你创建了....

```
@Bean
public SessionsSecurityManager securityManager(List<Realm> realms) {
    DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager(realms);
    // other stuff
    return securityManager;
}
```

You will have to inject them by yourself. for more deail, see below

你需要自行注入,更多信息看下面:

### 13.If you haven’t created your own `SessionManager` or `SessionsSecurityManager`

You are all set. Enjoy it!

如果没有创建,那么所有配置已经完整,好好享受.

### 14.If you have created your own `SessionManager` or `SessionsSecurityManager`

Inject `redisSessionDAO` and `redisCacheManager` which created by `shiro-redis-spring-boot-starter` already

```
@Autowired
RedisSessionDAO redisSessionDAO;

@Autowired
RedisCacheManager redisCacheManager;
```

Inject them into `SessionManager` and `SessionsSecurityManager`

```
@Bean
public SessionManager sessionManager() {
    DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();

    // inject redisSessionDAO
    sessionManager.setSessionDAO(redisSessionDAO);
    return sessionManager;
}

@Bean
public SessionsSecurityManager securityManager(List<Realm> realms, SessionManager sessionManager) {
    DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager(realms);

    //inject sessionManager
    securityManager.setSessionManager(sessionManager);

    // inject redisCacheManager
    securityManager.setCacheManager(redisCacheManager);
    return securityManager;
}
```

For full example, see [shiro-redis-spring-boot-tutorial](https://github.com/alexxiyang/shiro-redis-spring-boot-tutorial)

对于完整信息,查看这个链接.

### 15.Configuration Properties(配置属性)

| Title                                             | Default          | Description                                                  |
| :------------------------------------------------ | :--------------- | :----------------------------------------------------------- |
| shiro-redis.enabled                               | `true`           | Enables shiro-redis’s Spring module                          |
| shiro-redis.redis-manager.deploy-mode             | `standalone`     | Redis deploy mode. Options: `standalone`, `sentinel`, ‘cluster’ |
| shiro-redis.redis-manager.host                    | `127.0.0.1:6379` | Redis host. If you don’t specify host the default value is `127.0.0.1:6379`. If you run redis in sentinel mode or cluster mode, separate host names with comma, like `127.0.0.1:26379,127.0.0.1:26380,127.0.0.1:26381` |
| shiro-redis.redis-manager.master-name             | `mymaster`       | **Only used for sentinel mode** The master node of Redis sentinel mode |
| shiro-redis.redis-manager.timeout                 | `2000`           | Redis connect timeout. Timeout for jedis try to connect to redis server(In milliseconds) |
| shiro-redis.redis-manager.so-timeout              | `2000`           | **Only used for sentinel mode or cluster mode** The timeout for jedis try to read data from redis server |
| shiro-redis.redis-manager.max-attempts            | `3`              | **Only used for cluster mode** Max attempts to connect to server  在分布式模式下,尝试连接服务器Redis的次数 |
| shiro-redis.redis-manager.password                |                  | Redis password                                               |
| shiro-redis.redis-manager.database                | `0`              | Redis database. Default value is 0                           |
| shiro-redis.redis-manager.count                   | `100`            | Scan count. Shiro-redis use Scan to get keys, so you can define the number of elements returned at every iteration.   限制返回个数 |
| shiro-redis.session-dao.expire                    | `-2`             | Redis cache key/value expire time. The expire time is in second. Special values: `-1`: no expire `-2`: the same timeout with session Default value: `-2` **Note**: Make sure expire time is longer than session timeout. |
| shiro-redis.session-dao.key-prefix                | `shiro:session:` | Custom your redis key prefix for session management **Note**: Remember to add colon at the end of prefix. |
| shiro-redis.session-dao.session-in-memory-timeout | `1000`           | When we do signin, `doReadSession(sessionId)` will be called by shiro about 10 times. So shiro-redis save Session in  ThreadLocal to remit this problem. sessionInMemoryTimeout is expiration  of Session in ThreadLocal.  Most of time, you don’t need to change it. |
| shiro-redis.session-dao.session-in-memory-enabled | `true`           | Whether or not enable temporary(临时) save session in ThreadLocal |
| shiro-redis.cache-manager.principal-id-field-name | `id`             | Principal id field name. The field which you can get unique id to identify this principal. For example, if you use UserInfo as Principal class, the id field maybe `id`, `userId`, `email`, etc. Remember to add getter to this id field. For example, `getId()`, `getUserId(`), `getEmail()`, etc. Default value is `id`, that means your principal object must has a method called `getId()`  唯一标识存储数据的依据 |
| shiro-redis.cache-manager.expire                  | `1800`           | Redis cache key/value expire time.  The expire time is in second. 过期时间 |
| shiro-redis.cache-manager.key-prefix              | `shiro:cache:`   | Custom your redis key prefix for cache management **Note**: Remember to add colon at the end of prefix. |

​								缓存的过期时间(在前缀末尾添加冒号)



If you are using `shiro-redis` with `spring-boot-devtools`. Please add this line to `resources/META-INF/spring-devtools.properties` (Create it if there is no this file):

```java
restart.include.shiro-redis=/shiro-[\\w-\\.]+jar
```