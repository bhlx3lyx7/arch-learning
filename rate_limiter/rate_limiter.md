# Rate limiter

## algorithms
- token bucket
- leaky bucket
- fixed window
- sliding log
- sliding window

## rate limiter design
- it is simple to design a single node rate limiter
- for 3rd-party rate limiter or Lb rate limiter, it is much harder, traditional solution is to leverage a centralized storage like Redis for fast counting
- in eBay, Shield is designed as a rate limiter, works like a fuse
	+ counting results are submitted to a central service, with kafka & storm pipeline to guarantee high availability and correctness (decision made by event driven)
	+ if counting policy exceeds, subject will be punished, for a configured period (like a fuse) or the rest of calculation period (like fixed window); this is an interesting mechanism of rate limiting
	+ limit subjects can be on levels of app, ip, user, etc.

## distributed rate limiter
each service node makes decision to serve or deny a request independently, to serve requests fast, and get rid of the Redis storage.

- an idea is to roughly limit the rate for imprecise scenarios
	+ assume each node serves even or uneven traffic, but basically keeps stable
	+ each node heartbeats to a centralized storage like etcd, to announce its aliveness and recent traffic
	+ each node reads from the centralized storage, to predict its traffic percentage among all nodes, and calculate its own threshold, to work as a single node rate limiter
- cloud bouncer from Yahoo, is a distributed rate limiter
	+ it uses token bucket algorithm
	+ each node polls policies from a centralized policy service
	+ each node spreads its recent (last 1 second) traffic count to other nodes via gossip protocal
	+ each node makes decision independently
- another idea is to reserve a bulk of tokens from a centralized storage, not to reserve a token for every request, but it might be not that precise and could still face latency challenge

## References
- rate limiting algorithms: https://www.quinbay.com/blog/understanding-rate-limiting-algorithms
- rate limiter design: https://www.enjoyalgorithms.com/blog/design-api-rate-limiter
- rate limiting with redis: https://blog.callr.tech/rate-limiting-for-distributed-systems-with-redis-and-lua/
- ebay Shield: https://wiki.corp.ebay.com/display/PLATSEC/Design+of+SHIELD
- cloud bouncer - distributed rate limiter: https://yahooeng.tumblr.com/post/111288877956/cloud-bouncer-distributed-rate-limiting-at-yahoo