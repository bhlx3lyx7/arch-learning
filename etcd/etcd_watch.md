# Watch in etcd

Client watches a key or a prefix, when key changes, etcd notifies client.

## Notify

1. when a key changes, including set and del, watcher hub notifies the event
2. for each event, split its key by `/`, get several prefixes and the key itself, notify the all these paths this event
3. when registered watcher received an event, it writes response to client

## Watch registry

1. watch a key or prefix, with a long-live RPC
2. watch is registered to listen to the watcher hub event channel, the channel could even be dedicated to the watched key/prefix

*an alternative could be to register the watches into a key-value map, with watched key as key; when notifying, just find the watcher by key*

- etcd can even watch history events, because it can create the channels with a given size (1000), the events can be stored in the channels, for future registered watcher to listen
- watcher can watch from a given index, then only the events with largger index can be returned

## TTL key

1. etcd has a ttl key heap (min heap) of TTL keys, ordering by expire time
2. when create a ttl key, push it into the ttl key heap
	- there might be another map of ttl keys, maintained in memory, together with the heap
	- first push (or update) the key into the map
	- then the updated node (also in the heap) triggers re-ordering of the heap

etcd leader node monitor thread triggers ttl key expiration every 500ms
1. get the world lock first, which means no read/write now
2. loop to remove the top key if it is expired
3. release the world lock, read/write recover

## References
- Watch API and TTL implementation in etcd: https://hustcat.github.io/watch_in_etcd/