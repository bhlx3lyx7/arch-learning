# Rocket MQ Learning

## Features

**consume way**
- pull consume: consumer pull
- push consume: broker push

**consume group**
- clustering consume
- broadcasting consume

**msg order**
- global ordered
- partition ordered

**msg filtering**
- tag filtering in broker side

**delivery**
- at least once

**msg recall**
- re-consume historical msg by time in millisecond precision

**transaction msg**
- transaction msg including application local process & msg sending
- X/Open XA distributed transaction, eventual consistency

**delay msg**
- messageDelayLevel
- delay msg in broker level, not in topic level
- delay msg stored in broker special topic firstly, broker schedule to consume and push into real msg topic

**msg re-consume**
- retry in consumer group level, not in topic level
- retry topic for each consumer group

**msg re-send**
- RocketMQ naturally have duplicate msg, some other things will also produce duplicate msg like producer resend and consumer rebalance

**traffic control**
- producer traffic control: broker side trigger check condition, reject produce request to control traffic
- consumer traffic control: consumer side trigger check condition, reduce consume frequency

**dead-letter msg**
- consumer group level, with special topic to save the msg after max retry num
- can be monitored and resend to consume eventually

## References
- official site: https://rocketmq.apache.org/
- official document: https://github.com/apache/rocketmq/tree/master/docs/cn