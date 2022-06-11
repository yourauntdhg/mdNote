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

### 命令

## zset（有序集合）

### 介绍

### 命令
