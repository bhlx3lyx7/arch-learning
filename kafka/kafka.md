# Kafka Learning

## Arch

## Usage

### Producer

Configurations important for producer:
```
acks
retries
max.in.flight.requests.per.connection
delivery.timeout.ms
request.timeout.ms
linger.ms
batch.size
receive.buffer.bytes
send.buffer.bytes
enable.idempotence
```

### Consumer

Configurations important for consumer:
```
fetch.min.bytes
fetch.max.bytes
heartbeat.interval.ms
session.timeout.ms (depends on group.min.session.timeout.ms & group.max.session.timeout.ms in broker config)
auto.offset.reset
enable.auto.commit
auto.commit.interval.ms
max.poll.interval.ms
max.poll.records
receive.buffer.bytes
request.timeout.ms

```

- for long time process, consumer can `pause` the partitionSet and `resume` after process finish, to avoid the broker rebalance

### Data Loss

保证数据不丢，需要做到at least once，再加上处理数据的幂等性，就可以在生产环境中获得exact once的效果。

#### Broker
- min.insync.replicas (一般是N/2+1)
- no sync flush to disk, essentially async flush
- only provide flush config by msg or time, but not recommended

只要min.insync.replicas小于N，都可能在理论上存在丢数据的可能，因为partition选leader的策略是从insync list里随机挑一个，而insync list的更新频率默认是10s，所以在这个time gap里落后的follower有可能会被选成leader，这样就丢了一段数据。这种配置也是生产环境下会用的，所以这种丢数据的可能性是存在的。

如果把min.insync.replicas设置成等于N，理论上来说不会丢数据了，但是结合数据刷盘的实现方式，也是有可能丢数据的。  
leader先落盘，follower从leader的disk同步数据，但是follower因为异步刷盘，所以会在写入pagecache但是未刷盘的情况下ack，于是producer收到所有node的ack(如果配置ack=all)认为发送成功，但是这时leader node down，两个follower都没刷盘，同时断电，pagecache丢失。再重启之后，leader没起来，只有两个follower，会成功选出新leader，但是新写的数据就丢了。  
这个情况是非常极端的，只是理论场景，我们也不需要expect它会出现。从kafka的设计哲学来看，是expect从replica恢复数据来保证一致性，而不是从硬盘恢复数据，这样的设计哲学提供的可靠度也是够的，而且performance会更好。

从这一点来看，相对于kafka的落盘策略，raft就安全得多，每次集群sync都是刷盘操作，并且刷盘成功才会ack。对于业务层，只要做到response是在刷盘后，或者是通过读硬盘上的raft log来repsonse，就可以实现强一致性，response success的数据一定在多数node的硬盘上存在。

#### Producer
- ack = all
- sync send or wait for callback, fail then retry
- retries

前两点保证发送数据不丢，一般也会加上第三点来做自动重试。

#### Consumer
- no auto commit
- commit after process done

消费者不丢数据比较容易做到，前提是消费者处理数据需要幂等性，这样可以安全的重复消费。

## Reference
- https://kafka.apache.org/documentation/
