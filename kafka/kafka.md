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

## Reference
- https://kafka.apache.org/documentation/
