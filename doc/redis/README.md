Redis
===================

插件信息
--------------
###### 最新版本：1.3.3
###### 下载地址：[http://grails.org/plugin/redis](http://grails.org/plugin/redis)
###### 官方文档：[https://github.com/grails-plugins/grails-redis](https://github.com/grails-plugins/grails-redis)
###### 作者：Ted Naleid, Burt Beckwith, Brian Coles, Michael Cameron, Christian Oestreich, John Engelman, David Seiler
###### 协议：[Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0.txt)

功能介绍
--------------
Redis 插件提供整合Redis数据存储的功能。Redis 是一个轻快的数据结构服务器。该插件使用一些内存技术在 Redis 中缓存复杂操作的结果。
该插件为 grails应用 提供 Redis连接池，并提供了一些辅助方法和缓存/记忆方法，可极大提升应用的性能。

安装
--------------
    grails install-plugin redis

该插件默认认为 Redis 运行在 `localhost:6379`，您可以如其他池相关配置项一样在 `grails-app/conf/Config.groovy` 中修改它：

    grails {
        redis {
            poolConfig {
                // jedis pool specific tweaks here, see jedis docs & src
                // ex: testWhileIdle = true
            }
            port = 6379
            host = "localhost"
            timeout = 2000 //default in milliseconds
            password = "somepassword" //defaults to no password
        }
    }
在 poolConfig 节您可以修改任何 [JedisPoolConfig 中可用的设置项][jedispoolconfig]。它实现了Apache Commons [GenericObjectPool][genericobjectpool]。

使用说明
--------------
RedisService Bean ###

    def redisService

`redisService` bean 封装了一个连接池连接。它有很多缓存/记忆辅助方法, 模板方法, 和基础的 Redis 命令，是使用 Redis 的主要接口。
（未完待续）

[jedispoolconfig]:https://github.com/xetorthio/jedis/blob/master/src/main/java/redis/clients/jedis/JedisPoolConfig.java
[genericobjectpool]:http://commons.apache.org/pool/apidocs/org/apache/commons/pool/impl/GenericObjectPool.html

