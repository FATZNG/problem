1.为什么redis这么快

    因为是在内存中操作
    单线程执行，没有线程切换开销
    I/O多路复用机制
    高效的存储结构
2.为什么key过期了，还在内存中没有被删除

    （提问：为什么说它过期了？因为刚开始设置这个key的时候带了过期时间，但是没过期吗）
    2.1、在开始的时候，设置key带了过期时间，然后修改这个key的时候，没有带过期时间，则过期时间被覆盖成了 -1，即不过期
    2.2、定时删除还没有删除到这点来

3.redis删除key的机制

    1、惰性删除：在查这个key的时候，会比较key的过期时间，如果过期了，就删除
    2、定时删除：redis默认100ms删除一部分过期的冷数据
        2.1、为什么不全部删除：全部删除会对redis的性能造成影响
    3、主动清理机制：设置maxmemory到达上限时，根据以下三种策略进行删除
        三种策略：1、*删除过期*了的key
                    1.1、volatile-ttl:越早过期越先被删除
                    1.2、volatile-random：随机删除
                    1.3、volatile-lru：根据LRU算法进行淘汰
                    1.4、volatile-lru：根据LFUU算法进行淘汰
                2、对*所有key*进行操作
                    1.1、volatile-random：随机删除
                    1.2、volatile-lru：根据LRU算法进行淘汰
                    1.3、volatile-lru：根据LFUU算法进行淘汰
                3、*不操作*：返回拒绝写入的异常

4、LRU LFU

    LRU：(Least Recently used,最近最少次使用):淘汰很久没被访问的数据。
    LFU：(Least Frequently used,最不常使用)：淘汰最近一段时间被访问次数最少的数据。（exp：热点榜）


5、缓存穿透

    有请求先查redis，没有再查数据库，数据库有就写入到redis，但是有些数据在缓存和数据库中都没有，就导致请求跑到数据库了，就导致数据库压力过大。
    解决方案：
        数据库中返回为空值时，也把key缓存到redis里,并设置几分钟的过期时间
        访问白名单（bitmaps）
        布隆过滤器
        实时监控：缓存命中率低下时，排访问对象以及访问对象，设置IP黑名单

6、缓存击穿

    大量数据过期，导致请求直接全部大在数据库，导致数据库压力激增
    解决方案：
    监控热点数据，并根据策略调整过期时间。
    使用分布式锁（setnx），以抢占的方式去读取数据库，然后写入数据库，没有获取到锁的休眠一段时间。
    在原有的失效时间的基础上再加上一个随机缓存过期时间，让数据不会集体失效