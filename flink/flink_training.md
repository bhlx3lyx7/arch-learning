# Flink Training

## Concepts
- Job client
	+ generate dataflow graph from code
	+ submit job of the dataflow
- Job manager
	+ master node of job
	+ yarn scheduler (or k8s as scheduler)
	+ job master for each job instance
	+ manages task manager states and heartbeats
- Task manager
	+ worker node
	+ multiple task slots (task slot executes a sub-task)

## Deployment
- Session: long run cluster, schedule task slots when submit a job
	+ ebay Flink cluster only support this mode
	+ start job fast
	+ no isolation of different jobs
	+ daemon threads can not be recovered if task fails and need to reschedule to another task manager
- Job: start up cluster when submit a job
	+ start job slow
	+ isolation of different jobs
- Application: long run job master, start up task managers when submit a job
	+ alike job mode, but better than job mode
	+ job master long run, so start up faster than job mode
	+ isolation of different jobs

## General workflow when coding
- source
- operator
- sink

## Serialize & Deserialize
- internal data types (flink serializer)
	+ basic types
	+ row
	+ tuple
	+ pojo
	+ ...
- external data types (kryo as fall back serializer)
	+ avro
	+ ...

## Connector
- source
	+ kafka
- sink
	+ rolling policy
	+ files in buckets and subtasks: inprocess -> pending -> finish

## stream time processing
- streaming documents
	+ [streaming 101](https://www.oreilly.com/radar/the-world-beyond-batch-streaming-101/)
	+ [streaming 102](https://www.oreilly.com/radar/the-world-beyond-batch-streaming-102/)
- data time in stream
	+ **event time**
	+ storage time
	+ ingestion time
	+ **processing time**
- **watermark**: special element inserted by flink infra
	+ simply assumes an out-of-orderness of at N seconds, then set (timestamp - N seconds) as the inserted watermark
	+ always based on event time
	+ watermark triggers event time timer, then event time window can be triggered
	+ watermark is generated in the infra layer of flink
	+ watermark can be extract and set by user
- lateness
	+ data event time < current watermark
	+ bounded amount of lateness can be allowed
	+ event time exceeds the pre-defined lateness will be dropped, there's a metric to monitor the exceed lateness count, users can then re-configure the lateness bound
	+ downstream will receive updated result of closed window triggered by late event
- watermark styles
	+ periodic watermarks
	+ punctuated watermarks
- multi sources watermarks to downstream operator, use the smallest one
- kafka source watermark is better to assign watermark by partition and send the smallest one to downstream operator
- idle source can be detected and ignored
- side output: for really late event output, with a specialized output tag
- timers
	+ event-time timer
	+ processing-time timer
	+ processes of onTime() and processElement() are synchronized, so timer will block element process
	+ included in checkpoints
	+ recovery can recover event-time timer and processing-time timer with different strategies

## Window
- window types by key
	+ keyed window
		* keyed stream can be windowed in key group
	+ non-keyed window
		* windowAll: parallellism only 1
- window types by slide
	+ tumbling window: aligned, fixed-length, non-overlapping
	+ sliding window: aligned, fixed-length, overlapping
	+ session window: unaligned, variable-length
		* each event in a single session window with length, and merged if any overlap with others
		* late events can produce late merge, similar to the late event update closed window result
- customize window: based on global window, a long run window never triggers
	+ assigner: assign an event in which window
	+ trigger: trigger a window calculation
	+ evictor: evict data not needed in the window any more, like a filter
- count window: windowling by count rather than time
	+ count window result might be non-deterministic, because it depends on the ingestion order of upstream operators
	+ some windows are incomplete, then will need a custom trigger (eg. with timer) to discard or trigger the incomplete window
- window functions
	+ reduce, aggregate, fold: aggregate process
	+ process: non-aggregate process
- empty window is not exist in flink
	+ users can simulate empty window in workaround
- window alignment: flink will align the time window to 0 based unit, eg. align to 1:00 rather than 1:04

## Architecture

### Task slots and resources
- task manager: process
- task slot: thread, so multi slots in same TM (task manager) share the JVM resources
- slot sharing: one slot can run multi tasks, which means the slot is shared by multi tasks
	+ different partitions of the same operator can not be assigned to the same task slot
	+ api can manually assign operators in groups, to make them can share slot
- benefits of slot sharing
	+ necessary number of slots = max available parallelism, we can apply for a little more slots and task manager resource
	+ better resource utilization
- evenly spread-out slots: can be configured in cluster, to evenly spread tasks to TMs

### Graphs
- stream graph, in flink client
	+ stream node: source, operator, sink
	+ stream edge: edges added between nodes, different types: rebalance, hash, forward, etc.
- job graph: generated from stream graph, in flink client
	+ job vertex: job node
	+ job edge
	+ intermediate dataset: data queue to be send to downstream node
- execution graph: in flink job manager
	+ execution job vertex
	+ execution edge
	+ intermediate result
- physical execution graph: in flink job manager
	+ task: vertex
	+ input gate: edge
	+ result partition: data
	
### Memory usage of TM (task manager)
ordered by size, from small to large:
- flink framework etc: JVM heap
- operator state: JVM heap
- network buffer: off heap / native
- timer state: configured by backend type
- keyed state: configured by backend type

backend types:
- heap based: GC choice, scale out using multi task managers per node
- rocksdb based: fast storage, avoid network
- rocksdb w/timer in heap

performance considerations
- use efficient type serializers
- flattern pojos, avoid deep objects

### Fault tolerance
- exactly once
- at least once
- at most once

state persistence
- checkpoint / snapshot
- how to snapshot all the node states at the same point
	+ asynchornous barrier snapshotting (Chandy-Lamport)
	+ checkpoint barrier: initiated by coordinator (in job master) to the source, and propagate to the downstream operators and sinks; when all the nodes receives the barrier and take snapshots, coordinator can mark this barrier success
	+ the coordinator can work as the coordinator of distributed transaction, for multi sinks' barrier 2PL commit
	+ barrier alignment: for exactly once syntax, need to wait and align the barriers from multi upstreams
- snapshot persist to DFS is async
	+ state copy a backup, so the origin state can be used in following process
	+ upload the backup state
	+ then ack to coordinator as success

- checkpoint: managed by flink, intended for failure recovery
- savepoint: user triggered, retained indefinitely, intended for rescaling or upgrades, always full size

if change job topology, how to find the original state checkpoint
- uid of operator can be set to link state

unaligned checkpoint (need to enable this feature if needed)
- if some upstream input stream is slow, when a barrier comes, simply take snapshot, together with the input buffers (barrier not come) and output buffers