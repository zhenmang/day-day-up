## string

命令 行为 返回值 使用示例
 set 设置存储在给定键中的值 OK set('key', 'value')
 get 获取存储在给定键中的值 value/null get('key')
 del 删除存储在给定键中的值(任意类型) 1/0 del('key')
 incrby 将键存储的值加上整数 increment incrby('key', increment)
 decrby 将键存储的值减去整数 increment decrby('key', increment)
 incrbyfloat 将键存储的值加上浮点数 increment incrbyfloat('key', increment)
 append 将值 value 追加到给定键当前存储值的末尾 append('key', 'new-value')
 getrange 获取指定键的 index 范围内的所有字符组成的子串 getrange('key', 'start-index', 'end-index')
 setrange 将指定键值从指定偏移量开始的子串设为指定值 setrange('key', 'offset', 'new-string')

## list

命令 行为 返回值 使用示例
 rpush 将给定值推入列表的右端 当前列表长度 rpush('key', 'value1' [,'value2']) (支持数组赋值)
 lrange 获取列表在给定范围上的所有值 array lrange('key', 0, -1) (返回所有值)
 lindex 获取列表在给定位置上的单个元素 lindex('key', 1)
 lpop 从列表左端弹出一个值，并返回被弹出的值 lpop('key')
 rpop 从列表右端弹出一个值，并返回被弹出的值 rpop('key')
 ltrim 将列表按指定的 index 范围裁减 ltrim('key', 'start', 'end')

## hash

命令 行为 返回值 使用示例
 hset 在散列里面关联起给定的键值对 1(新增)/0(更新) hset('hash-key', 'sub-key', 'value') (不支持数组、字符串)
 hget 获取指定散列键的值 hget('hash-key', 'sub-key')
 hgetall 获取散列包含的键值对 JSON hgetall('hash-key')
 hdel 如果给定键存在于散列里面，则移除这个键 hdel('hash-key', 'sub-key')
 hmset 为散列里面的一个或多个键设置值 OK hmset('hash-key', obj)
 hmget 从散列里面获取一个或多个键的值 array hmget('hash-key', array)
 hlen 返回散列包含的键值对数量 hlen('hash-key')
 hexists 检查给定键是否在散列中 1/0 hexists('hash-key', 'sub-key')
 hkeys 获取散列包含的所有键 array hkeys('hash-key')
 hvals 获取散列包含的所有值 array hvals('hash-key')
 hincrby 将存储的键值以指定增量增加 返回增长后的值 hincrby('hash-key', 'sub-key', increment) (注：假如当前 value 不为为字符串，则会无输出，程序停止在此处)
 hincrbyfloat 将存储的键值以指定浮点数增加

## zset

命令 行为 返回值 使用示例
 zadd 将一个带有给定分支的成员添加到有序集合中 zadd('zset-key', score, 'key') (score 为 int)
 zrange 根据元素在有序排列中的位置，从中取出元素
 zrangebyscore 获取有序集合在给定分值范围内的所有元素
 zrem 如果给定成员存在于有序集合，则移除
 zcard 获取一个有序集合中的成员数量 有序集的元素个数 zcard('key')

## keys 命令组

命令 行为 返回值 使用示例
 del 删除一个(或多个)keys 被删除的 keys 的数量 del('key1'[, 'key2', ...])
 exists 查询一个 key 是否存在 1/0 exists('key')
 expire 设置一个 key 的过期的秒数 1/0 expire('key', seconds)
 pexpire 设置一个 key 的过期的毫秒数 1/0 pexpire('key', milliseconds)
 expireat 设置一个 UNIX 时间戳的过期时间 1/0 expireat('key', timestamp)
 pexpireat 设置一个 UNIX 时间戳的过期时间(毫秒) 1/0 pexpireat('key', milliseconds-timestamp)
 persist 移除 key 的过期时间 1/0 persist('key')
 sort 对队列、集合、有序集合排序 排序完成的队列等 sort('key'[, pattern, limit offset count])
 flushdb 清空当前数据库