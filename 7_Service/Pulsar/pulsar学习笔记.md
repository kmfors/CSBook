# Namespaces

Pulsar 命名空间是主题的逻辑分组，也是租户内的逻辑命名。租户通过管理 API 创建命名空间。例如，拥有不同应用程序的租户可以为每个应用程序创建一个单独的命名空间。命名空间允许应用程序创建和管理主题的层次结构。主题 my-tenant/app 1 是我的租户的应用程序 app 1 的命名空间。您可以在命名空间下创建任意数量的主题。

# Subscriptions

Pulsar subscription 是一个命名的配置规则，它决定了如何向消费者发送消息。它是由一组消费者建立的主题租约。Pulsar 有四种订阅类型：

- 独占 (exclusive) 订阅
- 共享 (shared) 订阅
- 故障转移 (failover) 订阅
- 键 (key_shared) 共享

![[pulsar学习笔记-20240523164533.png]]


## Key_shared

有三种映射算法规定了如何为给定的信息密钥（或排序密钥）选择消费者：

- Auto-split Hash Range（散列范围）
- Auto-split Consistent Hashing（一致散列）
- Sticky（粘性）

当消费者使用 Key_Shared 订阅类型时，需要为生产者禁用批处理（默认启用）或使用基于密钥的批处理。

对于 Key_Shared 订阅类型来说，基于密钥的批处理是必要的，原因有两个：

- 代理会根据消息的密钥分派消息，但默认的分批方法可能无法将具有相同密钥的消息打包到同一批次。
- 由于是消费者而不是代理从批次中分派消息，因此一个批次中第一条消息的密钥会被视为该批次中所有消息的密钥，从而导致上下文错误。

```C++
ProducerConfiguration producerConfig;
producerConfig.setBatchingType(ProducerConfiguration::BatchingType::KeyBasedBatching);
Producer producer;
client.createProducer("my-topic", producerConfig, producer);
```