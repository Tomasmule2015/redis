# redis
# 1 Reids 一般都有哪些使用场景？
1 缓存： 减轻数据库的查询压力  
2. 排行榜，利用redis的sort set 实现  
3. 计数器、限速器， 利用reis中原子性自增操作，可以统计用户点赞数，用户访问数等。  
4. 消息队列： redis的发布订阅模式， list队列机制。  
5. session共享： Session保存在服务器的文件中，如果是集群服务， 用户过来可能落在不同的机器上，就会导致用户频繁登录。 采用redis保存session， 用户可以获取到对于的session信息。  
6. 好友关系， 利用集合命令。 比如求交集，并集，差集等。 解决共同好友，共同爱好。  
# 2 Redis有哪些常见的功能？
1. 数据缓存功能  
2. 支持数据持久化  
3. 支持事务  
4. 支持消息队列  
5. 分布式锁功能  
# 3 Redis支持的数据类型有哪些？
string ,hash , list, set ,sort set  

# 4 Redis为什么快？
1. 完全基于内存  
2. 数据结构简单，对数据操作简单  
3. 采用单线程，避免上下文切换，不需要使用锁  
4. 使用多路I/O服用模型，非阻塞IO  

# 5 Redis可能有哪些缓存问题，怎么解决？
1. 缓存雪崩  
2. 缓存穿透  
3. 缓存击穿
4. 双写不一致（缓存数据一致性）  

# 6 怎么保证缓存和数据库的数据一致性？
https://juejin.cn/post/6964531365643550751  
删除缓存还是更新缓存？  
应该删除缓存，使用更新缓存会出现脏数据问题  
更新缓存如果缓存是经过复杂计算得到的，那么会浪费性能  

双写的情况下，先操作数据库还是操作缓存？  
如果先操作缓存，还是可能数据不一致。 因为先删除掉的缓存，可以被另外一个事务进行读操作给重新加回来。  

1. 缓存延时双删  
   删除缓存-》更新数据库-》sleep 一会 -》删除缓存  
2. 删除缓存重试机制  
  将删除失败的key放到消息队列里面，然后从队列里面拿出来重新做删除操作  
3. 读取binlog 异步删除缓存    
   将binlog发送到MQ队列里，然后通过ACK机制确认处理这条更新消息，删除缓存  

# 7  Redis持久化有哪些方式  
RDB （redis data base） 把内存数据以快照的形式保存到硬盘的二进制文件中    
AOF （append-only file）. Reids会将每一个收到的命令都通过write函数追加到文件最后。    

# 8 Redis内存淘汰策略有哪些？
volatile-lru: 从已设置过期时间的数据集中挑选最近最少使用的数据淘汰  
volatile-ttl: 从已设置过期时间的数据集中挑选将要过期的数据淘汰  
volatile-random: 从已设置过期时间的数据集中随机挑选数据淘汰  
allkey-lru : 从所有数据中挑选最近最少使用的数据淘汰  
allkeys-random: 从所有数据中随机挑选数据淘汰  
no-enviction(驱逐）： 禁止驱逐数据  
# 9 Redis 常见性能问题和解决策略？
主从复制慢 ， master最好不要做持久化工作。 master 和 slave最好一个局域网内。 主从复制不要用图状结构，单向链表更稳定。  

# 10 Redis的过期键的删除策略？ 
定时过期， key 有过期时间，到期即清除， 占用cpu.  
惰性过期， 只有当访问key时，才判断是否过期， （不友好）  
定期清除, 每隔一段时间扫描一定数量的key， 清除其中已过期的key.  
reis 同时使用了惰性过期和定期过期两种过期策略。  

# 12 Hash 冲突怎么办？
redis 通过链式哈希解决冲突， 同一个桶里面的元素使用链表报错，但是链表过长会导致性能变差。  
redis 有个rehash机制， 是为了让哈希表的负载因子维持在一个合理范围内。 保留新旧两个hash表（新的是扩容或者缩容），旧的数据慢慢往新的表迁移。  

# 13 什么是RDB内存快照？
redis内存中的数据在某一刻的状态数据， fork一个子进程进行内存快照的保存。  

# 14 在生成RDB期间，Redis可以同时处理写请求吗？  
可以，主进程可以处理客户请求， 从进程进行生成RDB操作  

# 15 Redis如何做内存优化？
1. 控制key的数量  对于存储相同的数据内容利用Redis的数据结构降低外层键的数量，也可以节省大量内存。  
2. 缩减key value的长度， key 业务设计时优化， value 可以考虑序列化成二进制  
3. 编码优化，内部编码优化  

# 16 Redis事务相关面试题  
Redis事务支持隔离吗？  
Redis单进程程序，且执行过程中事务队列中的命令不变。支持隔离。  
Redis事务保证原子性吗？ 支持回滚吗？  Redis单条命令是原子性的，但是事务不保证原子性。没有回滚。 事务中任意命令执行失败，其余的命令也会被执行。  

# 17 Redis线程模型

# 18 为什么需要Redis分区？ 有哪些实现方案？有什么优缺点？

# 19 如何解决Redis并发竞争key的问题

# 20 Redis 相比memcached 有哪些优势？

# 21 什么是缓存预热？

# 22 什么是缓存降级

# 23 Redis 6.0 为何映入多线程?

# 24 redis 哨兵机制是怎样的？

# 25 由于主从延迟导致读取到过期数据怎么处理？

# 26 主从复制的过程中如果因为网络原因停止复制了会怎样？

# 27 选取master的机制是怎样的？

# 28 Redis 集群是如何实现数据分布的，这种方式有什么优缺点？

# 29 RedLock解决了什么问题？ 原理是怎样的？



