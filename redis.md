# 单线程架构

单线程架构和I/O多路复用模型来实现高性能的内存数据库服务。

## 单线程模型

redis使用单线程来处理消息，客户端的所有请求到服务端都会进入队列里边然后逐个被执行

## 快的原因

* 纯内存访问，所有数据放在内存。响应飞快
* 非阻塞I/O，使用epoll作为I/O多路复用技术的实现

# 常用api

## 字符串

### 内部编码

* int：8字节长整型
* embstr： <=39字节的字符串
* raw：>=39个字节的字符串

### 使用场景

* 缓存功能，降低数据库压力
* 计数
* 流量削锋
* 通过setnx实现锁
* 用户身份验证

## 哈希

### 内部编码

* ziplist压缩列表：当哈希类型元素个数小于hash-max-ziplist-entries 配置（默认512个）、同时所有值都小于hash-max-ziplist-value配置（默认64 字节）时，Redis会使用ziplist作为哈希的内部实现。比哈希表节省空间。
* hashtable哈希表：当哈希类型无法满足ziplist的条件时，Redis会使 用hashtable作为哈希的内部实现，因为此时ziplist的读写效率会下降，而 hashtable的读写时间复杂度为O（1）

### 使用场景

可用于缓存用户信息等这些变动小的数据。

必要时可以缓存其他基本固定的信息以减少数据库的联表查询。

## 列表

是一个存储数据的有序列表，可对两端进行push或者pop操作

以此能够实现栈或队列

### 内部编码

* ziplist（压缩列表）：当列表的元素个数小于list-max-ziplist-entries配置 （默认512个），同时列表中每个元素的值都小于list-max-ziplist-value配置时 （默认64字节）
* ·linkedlist（链表）：当列表类型无法满足ziplist的条件时，Redis会使用 linkedlist作为列表的内部实现

### 使用场景

* 消息队列：的lpush+brpop命令组合即可实现阻塞队列，生产 者客户端使用lrpush从列表左侧插入元素，多个消费者客户端使用brpop命令 阻塞式的“抢”列表尾部的元素，多个客户端保证了消费的负载均衡和高可用性
* 文章列表：

* lpush+lpop=Stack（栈）
* lpush+rpop=Queue（队列）
* lpsh+ltrim=Capped Collection（有限集合）
* lpush+brpop=Message Queue（消息队列）

## 集合

无序，不可重复

### 内部编码

* intset整数集合：当集合中的元素都是整数且元素个数小于set-max-intset-entries配置（默认512个）时，Redis会选用intset来作为集合的内部实 现，从而减少内存的使用
* hashtable（哈希表）：当集合类型无法满足intset的条件时，Redis会使 用hashtable作为集合的内部实现。**元素不为整数，也会变成hashtable。**

### 使用场景

* 可以将用户所喜爱的文章、标签、记录，通过集合能够得到同一个爱好的人
  * 给用户添加标签
  * 给标签添加用户
  * 两着操作需放在同一个事物里

## 有序集合

元素不可重复，但是有序(同给给每个元素的score**(score能重复)**作为排序依据)

### 内部编码

* ziplist（压缩列表）：当有序集合的元素个数小于zset-max-ziplistentries配置（默认128个），同时每个元素的值都小于zset-max-ziplist-value配 置（默认64字节）时，Redis会用ziplist来作为有序集合的内部实现，ziplist 可以有效减少内存的使用
* skiplist（跳跃表）：当ziplist条件不满足时，有序集合会使用skiplist作 为内部实现，因为此时ziplist的读写效率会下降

### 使用场景

排行榜系统： 	zadd添加 zincrby给热度+1

## 键值问题

* 使用keys 来遍历键，可能会导致redis客户端请求阻塞，可使用一个不对外提供服务的redis来遍历键，但会影响主从复制
* scan渐进式遍历键，cursor为0开始，每次使用返回的cursor， 知道返回的cursor为0结束。键的变化（增加、删除、修改）会导致遍历结果不准确

# 对象

# 持久化

## RDB

