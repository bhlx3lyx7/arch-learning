# AMQP Learning

AMQP，即Advanced Message Queuing Protocol，一个提供统一消息服务的应用层标准高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同的开发语言等条件的限制。Erlang中的实现有RabbitMQ等。

## Concepcts
AMQ Model的设计是由以下需求驱动的：
- 确保符合标准的实现之间的互操作性。
- 提供清晰且直接的方式控制QoS
- 保持一致和明确的命名
- 通过协议能够修改服务端的各种配置
- 使用可以轻松映射到应用程序级API的命令符号
- 清晰，每个操作只能做一件事。

AMQP传输层是由以下需求驱动的
- 紧凑。能够快速封包和解包
- 可以携带任意大小的消息，没有明显的限制
- 同一个连接可以承载多个通道（Channel）
- 长时间存活，没有显著的限制
- 允许异步命令流水线
- 容易扩展。易于处理新需求、或者变更需求
- 向前兼容
- 使用强大的断言模型，可修复
- 对编程语言保持中立
- 适合代码生成过程

在设计过程中，希望能够支持不同的消息架构：
- 先存后发模型。有多个Writer，只有一个Reader
- 分散工作负载。有多个Writer和多个Reader
- 发布订阅模型，多个Writer和多个reader
- 基于消息内容的路由，多个Writer，多个Reader
- 队列文件传输，多个Writer，多个Reader
- 两个节点之间点对点连接
- 市场数据（Market data）分发。多个数据源，多个Reader

## AMQ Model
主要包含了三个主要的组件：
- exchange（交换器）：从Publisher程序中收取消息，并把这些消息根据一些规则路由到消息队列（Message Queue）中
- message queue（消息队列）：存储消息。直到消息被安全的投递给了消费者。
- binding ：定义了 message queue 和 exchange 之间的关系，提供了消息路由的规则。

## Implementation
- RabbitMQ

## References
- https://zhuanlan.zhihu.com/p/147675691
- https://baike.baidu.com/item/AMQP