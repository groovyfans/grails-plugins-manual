Grails Redis Plugin
===================

{hidden}
For integration between [Redis][redis] and Grails GORM layer, see the [Redis GORM plugin][redisgorm]. 
{hidden}
要整合 [Redis][redis] 到 Grails GORM 层, 请参见 [Redis GORM plugin][redisgorm]. 

{hidden}
That plugin was originally called "redis" (the name of this plugin), but it has since been refactored to "redis-gorm" and now relies on this plugin for connectivity.
{hidden}
该插件原名 "redis" , 现已改为 "redis-gorm" 并依赖该插件的连通性.

{hidden}
What is Redis?
{hidden}
什么是 Redis?
--------------
{hidden}
The best definition of Redis that I've heard is that it is a "collection of data structures exposed over the network".   
{hidden}
我听过的对Redis的最佳定义是：它是一个可发布于网络的数据结构集合

{hidden}
Redis is an [insanely fast][redisfast] key/value store, in some ways similar to [memcached][memcached], but the values it stores aren't just dumb blobs of data.  Redis values are data structures like [strings][redisstring], [lists][redislist], [hash maps][redishash], [sets][redisset], and [sorted sets][redissortedset].  Redis also can act as a lightweight pub/sub or message queueing system.
{hidden}
Redis 是个 [相当快][redisfast]的 键/值 存储器，某些地方类似于 [memcached][memcached]，只是它保存的值不只是数据团。Redis 值可以是一种数据结构如 [strings][redisstring], [lists][redislist], [hash maps][redishash], [sets][redisset], 和 [sorted sets][redissortedset]。同时 Redis 还可以作为轻量级的 发布/订阅 系统 或者 消息队列 系统。

{hidden}
Redis is used in production today by a [number of very popular][redisusing] websites including Craigslist, StackOverflow, GitHub, The Guardian, and Digg.
{hidden}
今天 Redis 已被用在 [多个流行的][redisusing] 网站，包括 Craigslist, StackOverflow, GitHub, The Guardian,  和 Digg。

{hidden}
It's commonly lumped in with other NoSQL technologies and is commonly used as a caching layerhas some similarities to Memcached or Tokyo Tyrant.  Because Redis provides network-available data structures, it's very flexible and it's able to solve all kinds of problems.  The creator of Redis, Salvatore Sanfilippo, has a nice post on his blog showing [how to take advantage of Redis by just adding it to your stack][addredisstack].  With the Grails Redis plugin, adding Redis to your grails app is very easy.
{hidden}
Redis 经常与其他 NoSQL 技术相提并论，并经常被用于缓存层如 Memcached 或 Tokyo Tyrant 那般。由于 Redis 提供了网络可用的数据结构，它非常灵活，能够解决各种问题。Redis 的缔造者， Salvatore Sanfilippo，在他的 Blog 上有篇很精彩的文章 [how to take advantage of Redis by just adding it to your stack][addredisstack]。使用 Grails Redis Plugin，将 Redis 加入你的应用非常简单。

{hidden}
I've created an [introduction to Redis using groovy][redisgroovy] that shows you how to install redis and use some basic groovy commands.  There is also a [presentation that I gave at gr8conf 2011][slideshareggr].
{hidden}
我曾写过一篇 [introduction to Redis using groovy][redisgroovy]，展示如何安装 redis 并使用一些基础的 groovy 命令。这还有另一篇有关的文章 [presentation that I gave at gr8conf 2011][slideshareggr]。

{hidden}
The official [Redis documentation][redis] is fantastic and includes a comprensive [list of Redis commands][rediscommands], each command web page also has an embedded REPL that lets you test out the command against a live Redis server.
{hidden}
官方的 [Redis 文档][redis] 很精彩，包含了精简的 [Redis 命令列表][rediscommands]，每个命令页面包含了一个嵌入式命令行工具，使你可以立刻在 Redis 服务器上试验该命令。


What is Jedis?
==============

[Jedis][jedis] is the Java Redis connection library that the Grails Redis plugin uses.  It's actively maintained, very fast, and doesn't try to do anything too clever.  One of the nice things about it is that it doesn't try to munge around with the Redis command names, but follows them as closely as possible.  This means that for almost all commands, the [Redis command][rediscommands] documentation can also be used to understand how to use the Jedis connection objects.  You don't need to worry about translating the Redis documentation into Jedis commands.

Installation
------------

    grails install-plugin redis


Out of the box, the plugin expects that Redis is running on `localhost:6379`.  You can modify this (as well as any other pool config options) by adding a stanza like this to your `grails-app/conf/Config.groovy` file:

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

The poolConfig section will let you tweak any of the [setter values made available by the JedisPoolConfig][jedispoolconfig].  It implements the Apache Commons [GenericObjectPool][genericobjectpool].

Plugin Usage
------------

### RedisService Bean ###

    def redisService

The `redisService` bean wraps the pool connection.   It has a number of caching/memoization helper functions, template methods, and basic Redis commands, it will be your primary interface to Redis.

The service overrides `propertyMissing` and `methodMissing` to delegate any missing requests to a Redis connection object.  This means that any method that you'd normally call on a Redis connection object can be called directly on `redisService`.

    // overrides propertyMissing and methodMissing to delegate to redis
    def redisService

    redisService.foo = "bar"   
    assert "bar" == redisService.foo   

    redisService.sadd("months", "february")
    assert true == redisService.sismember("months", "february")
        
It also provides a template method called `withRedis` that takes a closure as a parameter.  It passes a Jedis connection object to Redis into the closure.  The template method automatically gets an object out of the pool and ensures that it gets returned to the pool after the closure finishes (even if there's an error).

    redisService.withRedis { Jedis redis ->
        redis.set("foo", "bar")
    }

The advantage to calling `withRedis` rather than just calling methods directly on `redisService` is that multiple commands will only use a single connection instance, rather than one per command.

Redis also allows you to pipeline commands.  Pipelining allows you to quickly send commands to Redis without waiting for a response.  When the pipeline is executed, it returns a Result object, which works like a `Future` to give you the results of the pipeline.  See the [Jedis][jedis] documentation on pipelining for more details.  It works pretty much like the `withRedis` template does:

    redisService.withPipeline { Pipeline pipeline ->
        pipeline.set("foo", "bar")
    }

Redis has the notion of transactions, but it's not exactly the same as a database transaction.  Redis transactions guarantee that all of the commands in the transaction will be executed as an atomic unit.  Because Redis is single threaded, you're guaranteed to execute atomically and have a known state throughout the transaction.  Redis does not support rolling back modifications that happen during a transaction.

The `withTransaction` template method automatically opens and closes the transaction for you.  If the closure doesn't throw and exception, it will tell Redis to execute the transaction

    redisService.withTransaction { Transaction transaction ->
        transaction.set("foo", "bar")
    }

### Memoization ###

Memoization is a write-through caching technique.  The plugin gives a number of methods that take a key name, and a closure as parameters.  These methods first check Redis to see if the key exists.  If it does, it returns the value of the key and does not execute the closure.  If it does not exist in Redis, it executes the closure and saves the result in Redis under the key.  Subsequent calls will then be served the cached value from Redis rather than recalculating.

This technique is very useful for caching values that are frequently requested but expensive to calculate.

As of version 1.2 you may also use the new memoize annotations. See the Memoization Annotation section for usage and examples.

There are methods for the basic Redis data types:

### String Memoization ###

    redisService.memoize("user:$userId:helloMessage") { 
        // expensive to calculate method that returns a String
        "Hello ${security.currentLoggedInUser().firstName}"
    }

By default, the key/value will be cached forever in Redis, you can ensure that the key is refreshed either by deleting the key from Redis, making the key include a date or timestamp, or by using the optional `expire` parameter, the value is the number of seconds before Redis should expire the key:

    def ONE_HOUR = 3600
    redisService.memoize("user:$userId:helloMessage", [expire: ONE_HOUR]) { 
        """
        Hello ${security.currentLoggedInUser().firstName. 
        The temperature this hour is ${currentTemperature()}
        """
    }

### Domain Object Memoization ###

You can memoize a single domain object with redis.  It will cache the ID of the domain object returned from the closure and on subsequent cache hits will return a proxy domain object using grails <code>DomainObject.load(cachedId)</code>.


    String key = "user:42:favorite:author"
    Author author = redisService.memoizeDomainObject(Author, key) { 
        Author author = ... // expensive method to calculate user 42's favorite author...
        return author
    }

Now that you have the proxy object for the Author, you can do queries with it without actually having to hydrate the object (and anything it eagerly loads):

    def recommendedBooks = Book.findByAuthor(author) 

The object has the id field populated, but the remaining fields are lazily loaded only if their values are requested, so you can still do:

    println author.name

To actually print out the name of the author.

### Domain List Memoization ###

You can also memoize a list of domain object identifiers.  It doesn't cache the entire domain object, just the database IDs of the domain objects in a returned list.

This allows you to still grab the freshest objects from the database, but not repeatedly create an expensive list.  This could be a big database query that joins a bunch of tables.  Or some other process that does additional filtering based on selections the user has made in the UI.  Something ephemeral for that session or user, that you don't want to persist, but need to be able to react to.

    def key = "user:$id:friends-books-user-does-not-own"

    redisService.memoizeDomainList(Book, key, ONE_HOUR) { redis ->
        // expensive process to calculate all friend’s books and filter out books
        // that the user already owns, this stores the list of determined Book IDs
        // in Redis, but hydrates the Book objects from the DB
    }
        
### Other Memoization Methods ###

There are other memoization methods that the plugin provides, check out the [RedisService.groovy][redisservicecode] and the plugin tests for the exhaustive list.

    // Redis Hash memoize methods
    
    redisService.memoizeHash("saved-hash") { return [foo: "bar"] }

    redisService.memoizeHashField("saved-hash", "foo") { return "bar" }

    // Redis List memoize method
    redisService.memoizeList("saved-list") { return ["foo", "bar", "baz"] }

    // Redis Set memoize method
    redisService.memoizeSet("saved-set") { return ["foo", "bar", "baz"] as Set } 

    // Redis Sorted Set memoize method
    redisService.memoizeScore("saved-sorted-set", "set-item") { return score }

### Other Methods ###

The plugin also provides a few utility methods such as:

    redisService.flushDB() // dangerous!!!! should probably only be used for test cleanup

    // deletes all keys in the database matching a pattern, this is fairly expensive
    // as it uses the <code>keys</code> operation.  If you're doing this a lot and
    // have many keys in redis, you should be aggregating your own set of keys that
    // you'll later want to delete
    redisService.deleteKeysWithPattern("key:pattern:*")

### Redis Pool Bean ###

You can have direct access to the pool of Redis connection objects by injecting `redisPool` into your code.   Normally, you won't want to directly work with the pool, and instead interact with the `redisService` bean, but you have the option to manually work with the pool if desired.

    def redisPool


### Redis Taglib ###

The `redis:memoize` TagLib lets you leverage memoization within your GSP files.  Wrap it around any expensive to generate text and it will cache it and quickly serve it from Redis.

    <redis:memoize key="mykey" expire="3600">
        <!--
            insert expensive to generate GSP content here

            taglib body will be executed once, subsequent calls
            will pull from redis till the key expires
        -->
        <div id='header'>
            ... expensive header stuff here that can be cached ...
        </div>
    </redis:memoize>

### Multiple Redis Servers ###

If you are using multiple redis servers in your environment which are NOT clustered and would like to perform discrete operations on them seperately from a single application, you can accomplish that by by adding some configuration to your application.

The configuration block for redis accepts the following connections block parameters.

Config.groovy
``` groovy
grails {
    redis {
        poolConfig {
            // pool specific tweaks here
            // for parms see https://github.com/xetorthio/jedis/blob/master/src/main/java/redis/clients/jedis/JedisPoolConfig.java
            // numTestsPerEvictionRun = 4
        }
        port = 6379
        host = "localhost"

        connections {
            cache {
                poolConfig {
                    // pool specific tweaks here
                    // for parms see https://github.com/xetorthio/jedis/blob/master/src/main/java/redis/clients/jedis/JedisPoolConfig.java
                    // numTestsPerEvictionRun = 4
                }
                port = 6380
                host = "localhost"
            }
            search {
                poolConfig {
                    // pool specific tweaks here
                    // for parms see https://github.com/xetorthio/jedis/blob/master/src/main/java/redis/clients/jedis/JedisPoolConfig.java
                    // numTestsPerEvictionRun = 4
                }
                port = 6381
                host = "localhost"
            }
        }
    }
}
```

The standard config block for the default connection has not changed.  The new configuration is under the `connections` block.  You will need to name your connections ('cache' and 'search' in the above block).  The names must be unique.

A new service bean will be wired in addition to the default `redisService` bean with the capitalized connection name appended to it.  For example the above two connections would create a `redisServiceCache` and `redisServiceSearch` bean you can reference from your application code.  If desired, you can also access the connection-specific `redisPool` beans that are created using the same naming convention, in this example they would be `redisPoolCache` and `redisPoolSearch`.

In addition to the newly wired beans, you may also choose to continue using the standard `redisService` bean and simply refer to the connections by name when invoking targets on the service via `redisService.withConnection('cache').withRedis{...}` or `redisService.withConnection('search').memoize(key){...}`.

_Note: It is up to you if you prefer using the main `redisService` bean and the `withConnection` method or if you want to inject the additional service beans.  The end result is the same and the withConnection is simply a pass through to the newly created beans._

``` groovy
class FooService {

    def redisService
    // custom created beans for each connection, you can use these or just use the `withConnection` method
    // both methods are demonstrated below for example purposes, but you'd like choose one method or the other
    def redisServiceCache
    def redisServiceSearch  

    def doWork(){
        redisService.withRedis { Jedis redis ->
            redis.set("foo", "bar")
        }

        redisService.withConnection('cache').withTransaction { Jedis redis ->
            redis.set("foo", "bar")
        }

        redisServiceSearch.withPipeline { Jedis redis ->
            redis.set("foo", "bar")
        }

        redisServiceCache.memoize("somecachekey") {Jedis redis ->
            return cacheData
        }

        redisService.withConnection('search').memoizeDomainList(Book, "domainkey"){
            return Book.findAllByTitleInList(["book1", "book3"])
        }

        redisService.memoizeDomainIdList(Book, "domainkey"){
            return Book.findAllByTitleInList(["book1", "book3"])
        }

        redisServiceCache.memoizeDomainObject(Book, "domainkey"){
            return Book.get(book1.id)
        }

        redisServiceSearch.memoizeHash("domainkey"){
             return [foo: "bar"]
        }
    }
}
```

Memoization Annotations
------------

### Memoization Annotations ###

In addition to using the concrete and finite redisService.memoize* methods, as of version 1.2 you may now also annotate a method with an appropriate @Memoize* annotation.  This will perform an AST transformation at compile time and wrap the entire body of the method with the corresponding memoization method.  The parameters such as key and expire are passed into the annotation and used in the redisService memoize method calls.

The following are available as annotations:

<table width="100%">
    <tr><td><b>Annotation</b></td><td><b>Description</b></td></tr>
    <tr><td>@Memoize</td><td>Used to memoize methods that return a "string" - redisService.memoize</td></tr>
    <tr><td>@MemoizeDomainObject</td><td>Used to memoize methods that return a domain object - redisService.memoizeDomain</td></tr>
    <tr><td>@MemoizeDomainList</td><td>Used to memoize methods that return a domain object list - redisService.memoizeDomainList</td></tr>
    <tr><td>@MemoizeHash</td><td>Used to memoize methods that return a hash - redisService.memoizeHash</td></tr>
    <tr><td>@MemoizeHashField</td><td>Used to memoize methods that return a hash field - redisService.memoizeHashField</td></tr>
    <tr><td>@MemoizeList</td><td>Used to memoize methods that return a list - redisService.memoizeList</td></tr>
    <tr><td>@MemoizeSet</td><td>Used to memoize methods that return a set - redisService.memoizeSet</td></tr>
    <tr><td>@MemoizeScore</td><td>Used to memoize methods that returns a score from a hash - redisService.memoizeScore</td></tr>
</table>

There are integration usage tests written in spock for services at [RedisMemoizeServiceSpec.groovy][redisannotationservicespeccode] and for domains at [RedisMemoizeDomainSpec.groovy][redisannotationdomainspeccode]

### Memoization Annotation Keys ###

Since the value of the key must be passed in but will also be transformed by AST, we can not use the `$` style gstring values in the keys.  Instead you will use the `#` sign to represent a gstring value such as `@Memoize(key = "#{book.title}:#{book.id}")`.

During the AST tranformation these will be replaced with the `$` character and will evaluate correctly during runtime as `redisService.memoize("${book.title}:${book.id}"){...}`.

Anything that is not in the format `key='#text'` or `key="${text}"` will be treated as a string literal.  Meaning that `key="text"` would be the same as using the literal string `"text"` as the memoize key `redisService.memoize("text"){...}` instead of the variable `$text`.

Any variable that you use in the key property of the annotation will need to be in scope for this to work correctly.  You will only get a RUNTIME error if you use a variable reference that is out of scope.

### Memoization Annotation Notes ###

You are not required to import the `import grails.plugin.redis.RedisService` namespace or declare the service `def redisService` on any objects you wish to use this annotation with as the AST transform will detect whether this field is on your object and add it for you.  You may certainly have either the import or def statements if you would like, but they are not required if you use the @Memoize* annotations.

The user should be aware that any annotated method will be completely wrapped in the redis service call so any calculations that are contained within will also be wrapped and not executed of the key is in scope and not expired.

If the compile succeeds but runtime fails or throws an exception, make sure the following are valid:
    * Your key OR value is configured correctly.
    * The key uses a #{} for all variables you want referenced.

If the compile does NOT succeed make sure check the stack trace as some validation is done on the AST transform for each annotation type:
    * Required annotation properties are provided.
    * When using `expire` it is a valid Integer type variable.
    * When using `value` it is a valid closure.
    * When using `key` it is a valid String.

### @Memoize ###

The @Memoize annotation is to be used when dealing with objects that are stored in Redis as strings.  This annotation takes the following parameters:

    value   - A closure in the following format. (key OR value required)
    key     - A unique key for the data cache. (key OR value required)
    expire  - Expire time in ms.  Will default to never so only pass a value like 3600 if you want value to expire.

*You can either specify a closure OR a key and expire.  When using the closure style key `@Memoize({"#{text}"})` you may not pass a key or expire to the annotation as the closure will be evaluated directly and used as the key value.  This is due to a limitation on how Java deals with closure annotation parameters.*

Here is an example of usage:

    @Memoize({"#{text}"})
    def getAnnotatedTextUsingClosure(String text, Date date) {
        println 'cache miss getAnnotatedTextUsingClosure'
        return "$text $date"
    }

    @Memoize(key = '#{text}')
    def getAnnotatedTextUsingKey(String text, Date date) {
        println 'cache miss getAnnotatedTextUsingKey'
        return "$text $date"
    }

    //expire this extremely fast
    @Memoize(key = '#{text}', expire = '1')
    def getAnnotatedTextUsingKeyAndExpire(String text, Date date) {
        println 'cache miss getAnnotatedTextUsingKeyAndExpire'
        return "$text $date"
    }

    @Memoize(key = "#{book.title}:#{book.id}")
    def getAnnotatedBook(Book book) {
        println 'cache miss getAnnotatedBook'
        return book.toString()
    }

### @MemoizeDomainObject ###

The @MemoizeDomainObject annotation is to be used when dealing with domain objects that are to have their id's stored in Redis.  See the documentation on Domain Object Memoization above for more details.  This annotation takes the following parameters:

    key     - A unique key for the data cache. (required)
    expire  - Expire time in ms.  Will default to never so only pass a value like 3600 if you want value to expire.
    clazz   - The class of the object to be memoizing. (required)

Here is an example of usage:

    @MemoizeDomainObject(key = "#{title}", clazz = Book.class)
    def createDomainObject(String title, Date date) {
        println 'cache miss createDomainObject'
        Book.build(title: title, createDate: date)
    }

### @MemoizeDomainList ###

The @MemoizeDomainList annotation is to be used when dealing with lists of domain objects that are to have their id's stored in Redis.  See the documentation on Domain List Memoization above for more details.  This annotation takes the following parameters:

    key     - A unique key for the data cache. (required)
    expire  - Expire time in ms.  Will default to never so only pass a value like 3600 if you want value to expire.
    clazz   - The class of the object to be memizing. (required)

Here is an example of usage:

    @MemoizeDomainList(key = "getDomainListWithKeyClass:#{title}", clazz = Book.class)
    def getDomainListWithKeyClass(String title, Date date) {
        println 'cache miss getDomainListWithKeyClass'
        Book.findAllByTitle(title)
    }

### @MemoizeList ###

The @MemoizeList annotation is to be used when dealing with list type objects.  This annotation takes the following parameters:

    value   - A closure in the following format. (key OR value required)
    key     - A unique key for the data cache. (key OR value required)
    expire  - Expire time in ms.  Will default to never so only pass a value like 3600 if you want value to expire.

*You can either specify a closure OR a key and expire.  When using the closure style key `@Memoize({"#{text}"})` you may not pass a key or expire to the annotation as the closure will be evaluated directly and used as the key value.  This is due to a limitation on how Java deals with closure annotation parameters.*

Here is an example of usage:

    @MemoizeList(key = "#{list[0]}")
    def getAnnotatedList(List list) {
        println 'cache miss getAnnotatedList'
        return list
    }


### @MemoizeScore ###

The @MemoizeScore annotation is to be used when dealing with scores in hashes.  This annotation takes the following parameters:

    key     - A unique key for the data cache. (required)
    expire  - Expire time in ms.  Will default to never so only pass a value like 3600 if you want value to expire.
    member  - The hash property to store. (required)

Here is an example of usage:

    @MemoizeScore(key = "#{map.key}", member="foo")
    def getAnnotatedScore(Map map) {
        println 'cache miss getAnnotatedScore'
        return map.foo
    }

### @MemoizeHash ###

The @MemoizeHash annotation is to be used when dealing with maps/hash type objects.  This annotation takes the following parameters:

    value   - A closure in the following format. (key OR value required)
    key     - A unique key for the data cache. (key OR value required)
    expire  - Expire time in seconds.  Will default to never so only pass a value like 3600 (one hour) if you want value to expire.

*You can either specify a closure OR a key and expire.  When using the closure style key `@Memoize({"#{text}"})` you may not pass a key or expire to the annotation as the closure will be evaluated directly and used as the key value.  This is due to a limitation on how Java deals with closure annotation parameters.*

Here is an example of usage:

    @MemoizeHash(key = "#{map.foo}")
    def getAnnotatedHash(Map map) {
        println 'cache miss getAnnotatedHash'
        return map
    }

Release Notes
=============

* 1.0.0.M7 - released 8/5/2011 - this is actually the first released revision of the plugin. As it's replacing the old "redis" plugin (now "redis-gorm"), we needed to start with a number higher than the last released revision of that.  If you want the old redis-gorm plugin (which hasn't been released as of 8/5/2011), you can use "grails install-plugin 1.0.0.M6"
* 1.0.0.M8 - released 8/15/2011 - bugfix release mostly around the JedisTemplate that was ported from redis-gorm
* 1.0.0.M9 - released 8/16/2011 - removal of the Jedis/RedisTemplate stuff from redis-gorm as it's needed by things that can't rely on grails plugins, minor bugfixes for tests.
* 1.1 - released 12/10/2011 - removed hibernate & tomcat plugin dependency, added memoizeSet, memoizeList, memoizeDomainObject, and deleteKeysWithPattern methods, significantly reduced amount of time redis connections were used by plugin during memoization, BREAKING CHANGE: memoize methods no longer pass a Jedis connection object into the closure, they must be created on demand within the closure code.
* 1.2 - released 2/1/2012 - added memoize annotations to support spring-cache like support on domain, service, and controller classes.
* 1.3 - released 4/28/2012 - added support for additional redis server endpoint wiring via config block.
* 1.3.1 - released 5/20/2012 - adds support for Jedis 2.1 and a `database` parameter in the config to pick a redis database
* 1.3.2 - released 7/6/2012 - marks erroring pool connections as invalid so the pool knows to discard and recreate them
* 1.3.3 - released 2/27/2013 - bugfix for transactions

[redisgorm]: http://grails.github.com/inconsequential/redis/
[redis]: http://redis.io
[redisgroovy]: http://naleid.com/blog/2010/12/28/intro-to-using-redis-with-groovy/
[slideshareggr]: http://naleid.com/blog/2011/06/27/redis-groovy-and-grails-presentation-at-gr8conf-2011-and-gum/
[rediscommands]: http://redis.io/commands
[redisstring]:http://redis.io/commands#string
[redislist]:http://redis.io/commands#list
[redishash]:http://redis.io/commands#hash
[redisset]:http://redis.io/commands#set
[redissortedset]:http://redis.io/commands#sorted_set
[redisfast]:http://redis.io/topics/benchmarks
[memcached]:http://memcached.org/
[redisusing]:http://redis.io/topics/whos-using-redis
[addredisstack]:http://antirez.com/post/take-advantage-of-redis-adding-it-to-your-stack.html
[jedis]:https://github.com/xetorthio/jedis/wiki
[jedispoolconfig]:https://github.com/xetorthio/jedis/blob/master/src/main/java/redis/clients/jedis/JedisPoolConfig.java
[genericobjectpool]:http://commons.apache.org/pool/apidocs/org/apache/commons/pool/impl/GenericObjectPool.html
[redisservicecode]:https://github.com/grails-plugins/grails-redis/blob/master/grails-app/services/grails/plugin/redis/RedisService.groovy
[redisannotationservicespeccode]:https://github.com/grails-plugins/grails-redis/blob/master/test/projects/default/test/integration/grails/plugin/redis/RedisMemoizeServiceSpec.groovy
[redisannotationdomainspeccode]:https://github.com/grails-plugins/grails-redis/blob/master/test/projects/default/test/integration/grails/plugin/redis/RedisMemoizeDomainSpec.groovy