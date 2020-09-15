# Redis

## Overview
Redis holds the database completely on memory. Open source, and written in C.
Advances:
- Fast
- Rich types, compare to other key-value store
- Atomic Operation
- Multi-utility tool

## Commands
### Keys
- rename oldkey newkey -> redis supports renaming the key
- exists key -> check if a key exists
- tll key -> remaining time in key expiry
- randomkey -> return a random key from db (point?)
- type key -> type of key

### String
- set / get
- setex key seconds value -> set a key-value pair with time to expire
- setnx key value -> computeIfAbsent
- getrange key start end -> get a substring of the value
- getset key value -> set the new value and get the old one
- mset key value [key value ...] -> set a set of key-values
- msetnx key value [key value ...] -> set a set of key-values only if key does not exist
- mget key1 [key2 ...] -> get a batch of values from keys
- setrange key offset value -> set the value from offset bit
- strlen key -> get the length of the value
- incr / decr key
- incrby(float) / decrby(float) key offset
- append key value -> append the value

### Hash
map inside the big map 
10004 name xiaoming age 10
the commands are just like string, with an 'h' in the head
- hset / hget key field (value)
- hmset / hmget key field (value) [field (value) ...]
- hmsetnx / hmgetnx -> only if not existed
- hdel key field [field ...]
- hexists key field -> there is an 's'
- hgetall key -> get all fields and values as a list
- hkeys / hvals key
- hscan

### List
a list as the value. Most commands starts with an 'l', except commands that handle the last element
- lpush / rpush key value [value ...] -> push value(s) from head / tail
- lpushx / rpushx key value -> add if the list exists
- lpop / rpop key -> get a value
- (b)rpoplpush source destination (timeout) -> pop the last element and push it to another list
- lindex key index -> get the "index"th element
- llen key -> length of the list
- lrange key start stop -> get a sublist of the list
- ltrim key start stop -> trim a sublist of the list

### Sets
unordered set of unique strings
It feels like each type is written by separated groups with no common standards
- sadd key member [member ... ] -> add members
- scard key -> count
- sdiff key1 [key2, key3 ... ] -> find unique members in key1, subtracts common members in key2, key3...
- sinter key1 [key2, key3 ... ] -> find intersects among these keys
- sinterstore newkey [key1, key2 ... ] -> find intersects and store them into newkey
- sismember key member
- smembers
- smove source destination member
- spop key -> get a random member, and remove it
- srandmember key [count ] -> get random member(s) without removing it(them)
- srem key [member1, member2 ... ] -> remove (see? rm, rem)
- sunion key1 [key2 ] -> union
- sunionstore dest key1 [key2 ]
- sscan

### Sorted Sets
start with a 'z'
- zadd key score1 member1 [score2 member2 ...] -> insert members with scores. Smaller score is, prior it the member is
- zcount key min max -> count, only count, min max score
- zincrby key increment member -> increase its score
- zinterstore dest numofkeys key1 [key2 ... ] -> store intersects of 'numofkeys' keys, the common members will be stored by sum of scores
- zunionstore dest numofkeys key1 [key2 ... ] -> store, with sum of scores
- zrem key member [member ... ] -> rem again
- zremrangeby(lex)(rank)(score) key min max -> range by ...
- zrevrange key start stop
- zrevrank key member -> get the reversed rank

### Publish / Subscribe
publish and subscribe via the 'channel'
- psubscribe pattern [pattern ...] -> subscribe channels with regular expression, * ? []
- pubsub subcommand ...
- publish / subscribe 
- punsubscribe [pattern [pattern ...]] -> stop subscribing with pattern
- unsubscribe channel [channel ... ] -> stop subscribing 

### Transactions
a batch of commands to execute
- multi -> start storing multiple commands
- exec -> execute all commands
- watch key [key ... ] -> watch if the key-value is changed before the 'exec' command, if so, cancel this transaction and do it again later
- discard -> discard after multi
- unwatch -> unwatch all watched keys

### Scan
Different type supports different scan. Scan doesn't promise same results every time.
`hscan` `sscan` `zscan`
`scan cursor [match pattern] [count count] [type type]`
- cursor is the start of the iteration, from last iteration's return value. Each scan returns a new cursor, identifying the start in next iteration. 
- pattern a regular expression
- count number of results to scan this time
- type type, from version 6.0

## Locks
Redis is a single-threaded server, contrast to Memcache multi-threaded. One way to set a lock is to use "setnx", "expire" and "del" to identify, timeout and unlock a unique lock. When distributed, there may occur problems:
- **misunlock**: thread A gets lock, expire after 30s. After 30s, lock expires, B gets the lock. Then, A finish the execution, del the lock, B is still under operation, A will unlock B's lock.
  - add a identifier of current thread in the value. Check if the lock belongs to the current thread before deleting
- **Distribute by expire**: Thread A gets lock and expires but doesn't unlock, then B gets the lock, A and B get the lock concurrently. Set the expiration time long enough to finish the execution. Add daemon thread to hold the lock until the execution finished.
- **ReentrantLock**: If one thread allows multiple locks inside, this lock is reenterable. To make sure all locks are unlocked, use a countable key to identify the count. When count down to 0, the lock is done.
- **Use Subscribe & Publish**: It's time consuming to ask -> not expired -> ask -> ... So, if a thread finish its lock, it can publish a message to subscribers. Subscribers are threads waiting for locks. Then the ball kicks to how to let threads compete about the lock.

## Features 
- **Persistence**: Redis supports persistence, by storing the data into local disks. Each time start Redis, the data will be loaded into memory. Redis dump the snapshot of a period into dump.rdb. The other way is AOF(Append-only File), which adds write records one by one. RDB is fast, only one file, but will lose data if crash in the middle of a period. AOF is slow, but doesn't lose records. 
- **Sharding**: In a redis cluster, there are 16384 slots. Each node handles a part. When there comes a query, hash it and find its slot, then go to the right node. 

## Problem to Solve
- Cache Avalanche（缓存雪崩）
  - Lots of cached data expire at the same time, or the cache server crashes or is down. Then, all the queries flood into database, and the high load crashes the db or impacts the performance.
  - Solution: 
    - User clusters to ensure there is cache service at any time. 
    - Use lock or queue to make sure not too many threads make one-time read or write operation to the db. 
    - Arrange the expiration time of keys, let them not expire at the same time.
- Cache penetration（缓存穿透）
  - Data does not exist in db, so can't be in cache. When query this key, will ask cache, then db. Then, if someone query lots of nonexisting data, the db will have a large load.
  - Solution:
    - If a key doesn't exist both locations, store it in redis with an empty value. Set the expiration time short.
    - Bloom Filter. A huge map stores the keys that may exist on the database. Then, a key which definitely doesn't exist will be filtered off. 
- Cache breakdown
  - If a hot key in the cache expires, still lots of queries about this key come, db will overload.
  - Solution:
    - Lock. Use mutex key to make sure the value is added to the cache again if expires. Then, only one thread can access this object, and db won't have lots of queries.
    - Not expired. Do not set expire on the key.
- If full
  - Global space:
    - noneviction: throw error when add new key
    - allkeys-lru: remove least used keys
    - allkeys-random: remove random keys
  - Key space with expiration:
    - volatile-lru: remove least used keys with expiration
    - volatile-random: random with expiration
    - volatile-ttl: remove the key will die soon