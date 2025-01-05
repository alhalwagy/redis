# Environment setup

## Redis

```bash
docker network create some-network
docker run -p 6379:6379 --network some-network --name some-redis -d redis
```

https://redis.io/download/

## Redis CLI

```bash
docker run -it --network some-network --rm redis redis-cli -h some-redis
```

https://redis.io/docs/ui/cli/

## Redis Insight

![Redis Insight](redis-insight.png "Redis Insight")  
https://redis.com/redis-enterprise/redis-insight/

# Usage

## Connection

Check connection

```bash
> ping
"PONG"
```

Select database

```bash
> select 7
"OK"
```

Redis has 16 databases (0 to 15)

## Cache basics

```bash
> set k "the value"
"OK"

> get k
"the value"
```

Removing all

```bash
flushall
```

Getting all

````bash
> keys '*'
1) "k"


### Scanning
```bash
> mset users:1 alhalwagy users:2 ahmed users:3 mohammed users:4 amr users:5 ali
"OK"

> scan 0 count 2
1) "1"
2) 1) "users:5"
   2) "users:2"

> scan 1 count 2
1) "5"
2) 1) "users:1"
   2) "users:3"

> scan 5 count 2
1) "0"
2) 1) "users:4"
````

## key design

colon separated descriptive values like `user:1000:followers`  
a multiword in a key may be separated with dashes or dots like `comment:4321:reply.to` or `comment:4321:reply-to`  
The maximum allowed key size is 512 MB.

# Data types

## string

### Check key existence

```bash
> exists score
(integer) 0

> set score 1000
"OK"

> exists score
(integer) 1

> del score
(integer) 1

> exists score
(integer) 0
```

### `set` based on key existence

```bash
> set score 111 nx
"OK"
> set score 111 nx
(nil)

> del score
(integer) 1

> set score 222 xx
(nil)

> set score 222
"OK"

> set score 222 xx
"OK"
```

### `getset`

```bash
> set color red
"OK"

> getset color green
"red"

> get color
"green"
```

### `mset, mget`

```bash
> mset color red font arial
"OK"

> mget color font
1) "red"
2) "arial"
```

### key expiration

```bash
> set color red
"OK"

> expire color 10
(integer) 1

> exists color
(integer) 1

> exists color
(integer) 0
```

```bash
> set color red ex 10
"OK"

> ttl color
(integer) 6

> ttl color
(integer) 2

> ttl color
(integer) -2 // not exist

> get color
(nil)
```

### counter

```
> incr counter
(integer) 1

> incr counter
(integer) 2

> incr counter
(integer) 3

> incrby counter 5
(integer) 8

> set counter 10
"OK"

> decr counter
(integer) 9

> decrby counter 2
(integer) 7
```

## list (Linked List)

the operation of adding a new element in the head or in the tail of the list is performed in constant time

```
> lpush colors red green
(integer) 2

> lpush colors blue
(integer) 3

> rpush colors yellow
(integer) 4

> lrange colors 0 -1
1) "blue"
2) "green"
3) "red"
4) "yellow"
```

0 is the first element  
-1 is the last element

```
> rpop colors
"yellow"

> lpop colors 2
1) "blue"
2) "green"

> lpush colors red
(integer) 1

> llen colors
(integer) 1

> lpop colors
"red"

> exists colors
(integer) 0
```

removing all elements removes the key

## Hash

```bash
> hset user:1 name alhalwagy role admin active 1
(integer) 3

> hget user:1 name
"alhalwagy"

> hgetall user:1
1) "name"
2) "alhalwagy"
3) "role"
4) "admin"
5) "active"
6) "1"

> hmget user:1 name active
1) "alhalwagy"
2) "1"
```

## Set

unordered collections of strings

```bash
> sadd colors yellow red green blue
(integer) 4

> smembers colors
1) "yellow"
2) "blue"
3) "green"
4) "red"


> sismember colors blue
(integer) 1

> sismember colors black
(integer) 0


> sadd colors yellow orange
(integer) 1

> spop colors
"orange"

> spop colors
"red"
```

The SPOP command removes a random element

```bash
> scard colors
(integer) 3
```

## Sorted sets

set + hash

every element in a sorted set is associated with a floating point value, called the score  
elements are sorted based on the score

```bash
> zadd products 5000 mobile 200 charger 9000 tv 500 keyboard 1500 speaker
(integer) 5
```

insertion: O(log(N))

```bash
> zrange products 0 -1
1) "charger"
2) "keyboard"
3) "speaker"
4) "mobile"
5) "tv"


> zrevrange products 0 -1
1) "tv"
2) "mobile"
3) "speaker"
4) "keyboard"
5) "charger"


> zrange products 0 -1 withscores
1) "charger"
2) "200"
3) "keyboard"
4) "500"
5) "speaker"
6) "1500"
7) "mobile"
8) "5000"
9) "tv"
10) "9000"


> zrank products mobile
(integer) 3

> zremrangebyscore products 1000 10000000
(integer) 3

> zrevrange products 0 -1
1) "keyboard"
2) "charger"
```

# Transactions

All the commands in a transaction are serialized and executed sequentially.  
A request sent by another client will never be served in the middle of the execution of a Redis Transaction.  
This guarantees that the commands are executed as a single isolated operation.

multi then exec or discard  
Redis does not support rollbacks

```bash
> multi
"OK"

> set key1 value1
"QUEUED"

> incr counter
"QUEUED"

> incr counter
"QUEUED"

> exec
1) "OK"
2) "1"
3) "2"
```

# lua scripting

```bash
set name alhalwagy.ahmed

EVAL "local a = redis.call('get', KEYS[1]) .. ARGV[1] return redis.call('SET', KEYS[2], a)" 2 name mail @fawry.com

get mail
```

# Messaging

```bash
> PSUBSCRIBE 'inbox-*'
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "inbox-*"
3) (integer) 1

> PUBLISH inbox-alhalwagy hello1
(integer) 1

1) "pmessage"
2) "inbox-*"
3) "inbox-alhalwagy"
4) "hello1"
```

refrences:  
https://redis.io/docs/data-types/tutorial/
