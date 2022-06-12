# Redis 基础数据类型

Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset（有序集合）。

## string（字符串）

### 介绍

字符串类型是Redis的最基本数据结构。 字符串类型的值实际可以为字符串，数字，二进制，但是值最大不能超过512M。

| key     | value                              |
| ------- | ---------------------------------- |
| hello   | world                              |
| counter | 1                                  |
| bits    | 10000100                           |
| json    | {"id":1,"name":"xiaocai","age":18} |

### 命令

- 设置键值：`set key value [EX seconds] [PX milliseconds] [NX|XX]`

- 通过键获取值：`get key`

- 设置过期时间：`set key value ex 秒数`或`setex key 秒数 value`或`set key value px 毫秒数`

- 不存在key才能设置成功：`set key value nx`

- 存在key才能设置成功：`set key value xx`

- 批量设置或批量获取：`mset key1 value1 [key2 value2 ......]` `mget key1 key2`

- 计数：`incr key` `incrby key increment`

- 删除：`del key1 [key2 ......]`

> 内部编码：
>
> 1. int 8个字节的长整型
> 2. embstr 小于等于39个字节的字符串
> 3. raw  大于39个字节的字符串

## list（列表）

### 介绍

list类型是用来存储多个有序的字符串。每列字符串称之为元素。一个list的最大存储为2^32-1个元素。可以对列表进行双端插入和弹出，也可以指定索引下标获取元素。

### 命令

- 头部添加元素：`lpush key value [value ...]`

- 尾部添加元素：`rpush key value [value ...]`

- 头部和尾部删除元素：`lpop key` `rpop key`

- 索引操作:索引需要对全部list进行遍历，性能会随着元素个数的增大而变差

  - 根据key检索list一定范围内的value：lrange key start stop

    > start与end不是索引位置，而是偏移量。其中 0 表示列表的第一个元素， 1 表示列表的第二个元素，以此类推。 你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。

  - 通过索引获取列表中的元素：`LINDEX KEY_NAME INDEX_POSITION `

    > 可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。

  - 对一个列表进行修剪：`ltrim key start stop`

    > 列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。下标规则与索引规则一致。

  - 列表长度（大小）：`len key`

- 在列表的元素前或者后插入元素：`LINSERT key BEFORE|AFTER pivot value`

  > 将值 value 插入到列表 key 当中，位于值 pivot 之前或之后。
  >
  > 当指定元素不存在于列表中时，不执行任何操作。
  >
  > 当列表不存在时，被视为空列表，不执行任何操作。
  >
  > 如果 key 不是列表类型，返回一个错误。

- 通过索引来设置元素的值：LSET KEY_NAME INDEX VALUE

  > 当索引参数超出范围，或对一个空列表进行 LSET 时，返回一个错误。

> 内部编码：
>
> 1. ziplist（压缩列表）：小于3.2版本，当元素个数小于list-max-ziplist-entries配置（默认512个），同时每个元素的值长度都小于list-max-ziplist-value配置（默认64字节）
> 2. linkedlist（链表）：小于3.2版本，不满足ziplist的条件
> 3. quicklist：Redis 3.2版本，以一个ziplist为节点的linkedlist

## hash（哈希）

### 介绍

hash是一个string类型的field和value的映射表。 它适合用于存储对象，它是无序的，不能使用索引操作。

###  命令

- 设置：`HSET KEY_NAME FIELD VALUE`
- 哈希表中指定字段的值：`HGET KEY_NAME FIELD_NAME`
- 获取在哈希表中指定 key 的所有字段和值：`HGETALL KEY_NAME`
- 删除一个或多个哈希表字段：`HDEL KEY_NAME FIELD1.. FIELDN`
- 获取哈希长度：`hlen key`
- 同时将多个 field-value (域-值)对设置到哈希表 key 中：`HMSET key field1 value1 [field2 value2 \]`
- 查看哈希表 key 中，指定的字段是否存在：`HEXISTS key field`
- 获取哈希表中所有值：`HKEYS key`
- 获取哈希表中所有字段：`HVALS key`
- 只有在字段 field 不存在时，设置哈希表字段的值：`HSETNX key field value`

> 内部编码：
>
> 1. ziplist（压缩列表）：当元素个数小于hash-max-ziplist-entries配置（默认512个）和每个元素大小小于hash-max-ziplist-value配置（默认64字节）时
> 2. hashtable（哈希表）：不满足ziplist条件时

## set（集合）

### 介绍

- Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。

- 集合对象的编码可以是 intset 或者 hashtable。

- Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。

-  集合中最大的成员数为 232  - 1 (4294967295, 每个集合可存储40多亿个成员)。 

### 命令

- 添加一个或多个成员到命令中：` SADD KEY_NAME VALUE1..VALUEN`

  > 假如集合 key 不存在，则创建一个只包含添加的元素作成员的集合。
  >
  > 当集合 key 不是集合类型时，返回一个错误。
  >
  > *注意：在 Redis2.4 版本以前， SADD 只接受单个成员值。*

- 返回集合中元素的数量：`SCARD KEY_NAME `

- 返回第一个集合与其他集合之间的差异：`SDIFF FIRST_KEY OTHER_KEY1..OTHER_KEYN`

  > 不存在的集合 key 将视为空集。
  >
  > 差集的结果来自前面的 FIRST_KEY ,而不是后面的 OTHER_KEY1，也不是整个 FIRST_KEY OTHER_KEY1..OTHER_KEYN  的差集。

- 将给定集合之间的差集存储在指定的集合中：`SDIFFSTORE DESTINATION_KEY KEY1..KEYN`

  > 如果指定的集合 key 已存在，则会被覆盖。

- 返回给定所有集合的交集：`SINTER KEY KEY1..KEYN`

  > 不存在的集合 key 被视为空集。 当给定集合当中有一个空集时，结果也为空集(根据集合运算定律)。

- 返回给定所有集合的交集并存储在DESTINATION_KEY中：`SINTERSTORE DESTINATION_KEY KEY KEY1..KEYN`

- 判断 member 元素是否是集合 key 的成员：`SISMEMBER KEY VALUE`

- 返回集合中的所有成员：`SMEMBERS key`

- 将 member 元素从 source 集合移动到 destination 集合：`SMOVE source destination member`

  > MOVE 是原子性操作。
  >
  > 如果 source 集合不存在或不包含指定的 member 元素，则 SMOVE 命令不执行任何操作，仅返回 0 。否则， member 元素从 source 集合中被移除，并添加到 destination 集合中去。
  >
  > 当 destination 集合已经包含 member 元素时， SMOVE 命令只是简单地将 source 集合中的 member 元素删除。
  >
  > 当 source 或 destination 不是集合类型时，返回一个错误。

- 移除并返回集合中的一个随机元素：`SPOP key`

  > 该命令类似 [Srandmember](https://www.runoob.com/redis/sets-srandmember.html) 命令，但 SPOP 将随机元素从集合中移除并返回，而 Srandmember 则仅仅返回随机元素，而不对集合进行任何改动。

- 返回集合中一个或多个随机数：`SRANDMEMBER key [count]`

  > 从 Redis 2.6 版本开始， Srandmember 命令接受可选的 count 参数：
  >
  > - 如果 count 为正数，且小于集合基数，那么命令返回一个包含 count 个元素的数组，数组中的元素各不相同。如果 count 大于等于集合基数，那么返回整个集合。
  > - 如果 count 为负数，那么命令返回一个数组，数组中的元素可能会重复出现多次，而数组的长度为 count 的绝对值。
  >
  > 该操作和 SPOP 相似，但 SPOP 将随机元素从集合中移除并返回，而 Srandmember 则仅仅返回随机元素，而不对集合进行任何改动。

- 移除集合中一个或多个成员：`SREM key member1 [member2]`

  > 当 key 不是集合类型，返回一个错误。
  >
  > 在 Redis 2.4 版本以前， SREM 只接受单个成员值

- 返回所有给定集合的并集：`SUNION key1 [key2]`

- 所有给定集合的并集存储在 destination 集合中：`SUNIONSTORE destination key1 [key2]`

- 迭代集合中的元素：`SSCAN key cursor [MATCH pattern] [COUNT count]`

  - cursor - 游标。
  - pattern - 匹配的模式。
  - count - 指定从数据集里返回多少元素，默认值为 10 。

## zset（有序集合）

### 介绍

- Redis 有序集合和集合一样也是 string 类型元素的集合,且不允许重复的成员。
- 不同的是每个元素都会关联一个 double 类型的分数。redis 正是通过分数来为集合中的成员进行从小到大的排序。
- 有序集合的成员是唯一的,但分数(score)却可以重复。
- 集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。 集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。 

### 命令

| 序号 | 命令及描述                                                   |
| ---- | ------------------------------------------------------------ |
| 1    | [ZADD key score1 member1 (score2 member2)](https://www.runoob.com/redis/sorted-sets-zadd.html)  <br>向有序集合添加一个或多个成员，或者更新已存在成员的分数 |
| 2    | [ZCARD key](https://www.runoob.com/redis/sorted-sets-zcard.html)  <br/>获取有序集合的成员数 |
| 3    | [ZCOUNT key min max](https://www.runoob.com/redis/sorted-sets-zcount.html)  <br/>计算在有序集合中指定区间分数的成员数 |
| 4    | [ZINCRBY key increment member](https://www.runoob.com/redis/sorted-sets-zincrby.html)  <br/>有序集合中对指定成员的分数加上增量 increment |
| 5    | [ZINTERSTORE destination numkeys key (key ...)](https://www.runoob.com/redis/sorted-sets-zinterstore.html)  <br/>计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 destination 中 |
| 6    | [ZLEXCOUNT key min max](https://www.runoob.com/redis/sorted-sets-zlexcount.html)  <br/>在有序集合中计算指定字典区间内成员数量 |
| 7    | [ZRANGE key start stop (WITHSCORES)](https://www.runoob.com/redis/sorted-sets-zrange.html)  <br/>通过索引区间返回有序集合指定区间内的成员 |
| 8    | [ZRANGEBYLEX key min max (LIMIT offset count)](https://www.runoob.com/redis/sorted-sets-zrangebylex.html)  <br/>通过字典区间返回有序集合的成员 |
| 9    | [ZRANGEBYSCORE key min max (WITHSCORES) (LIMIT)](https://www.runoob.com/redis/sorted-sets-zrangebyscore.html)  <br/>通过分数返回有序集合指定区间内的成员 |
| 10   | [ZRANK key member](https://www.runoob.com/redis/sorted-sets-zrank.html)  <br/>返回有序集合中指定成员的索引 |
| 11   | [ZREM key member (member ...)](https://www.runoob.com/redis/sorted-sets-zrem.html)  <br/>移除有序集合中的一个或多个成员 |
| 12   | [ZREMRANGEBYLEX key min max](https://www.runoob.com/redis/sorted-sets-zremrangebylex.html)  <br/>移除有序集合中给定的字典区间的所有成员 |
| 13   | [ZREMRANGEBYRANK key start stop](https://www.runoob.com/redis/sorted-sets-zremrangebyrank.html)  <br/>移除有序集合中给定的排名区间的所有成员 |
| 14   | [ZREMRANGEBYSCORE key min max](https://www.runoob.com/redis/sorted-sets-zremrangebyscore.html)  <br/>移除有序集合中给定的分数区间的所有成员 |
| 15   | [ZREVRANGE key start stop (WITHSCORES)](https://www.runoob.com/redis/sorted-sets-zrevrange.html)  <br/>返回有序集中指定区间内的成员，通过索引，分数从高到低 |
| 16   | [ZREVRANGEBYSCORE key max min (WITHSCORES)](https://www.runoob.com/redis/sorted-sets-zrevrangebyscore.html)  <br/>返回有序集中指定分数区间内的成员，分数从高到低排序 |
| 17   | [ZREVRANK key member](https://www.runoob.com/redis/sorted-sets-zrevrank.html)  <br/>返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序 |
| 18   | [ZSCORE key member](https://www.runoob.com/redis/sorted-sets-zscore.html)  <br/>返回有序集中，成员的分数值 |
| 19   | [ZUNIONSTORE destination numkeys key (key ...)](https://www.runoob.com/redis/sorted-sets-zunionstore.html)  <br/>计算给定的一个或多个有序集的并集，并存储在新的 key 中 |
| 20   | [ZSCAN key cursor (MATCH pattern) (COUNT count)](https://www.runoob.com/redis/sorted-sets-zscan.html)  <br/>迭代有序集合中的元素（包括元素成员和元素分值） |
