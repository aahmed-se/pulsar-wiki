# Message dispatch throttling

 * **Status**: Under discussion
 * **Issue**: [[402](https://github.com/apache/incubator-pulsar/issues/402)]

## Motivation
Producers and consumers can interact with broker on high volumes of data. This can monopolize broker resources and cause network saturation, which can have adverse effects on other topics owned by that broker. Throttling based on resource consumption can protect against these issues and it can play important role for large multi-tenant clusters where a small set of clients using high volumes of data can easily degrade performance of other clients.
## Message throttling
### Consumer: 
Throttling can be configured per namespace basis by defining message-rate threshold. Sometimes, namespace with large message backlog can continuously drain messages with multiple connected consumers, which can easily over utilize broker’s bandwidth. Each broker owns a set of bundles under a namespace and with throttling enabled, broker can control message dispatching across all the bundles under that namespace. This throttling-limit is defined on a per-broker basis so, each namespace can fetch a maximum configured messages per second per broker before it gets throttled.

By default, broker does not throttle message dispatching for any namespace unless broker is configured with default throttling-limit or namespace has been configured as a part of namespace policy in a cluster. We can configure throttling limit for a specific namespace using an admin-api.
### Producer: 
Broker already has capability to throttle number of publish messages by reading only fixed number of in flight messages per client connection, which can protect broker if client tries to publish significantly large number of messages.

## Broker throttling configuration
By default, broker does not throttle message dispatching for any of the topics. However, if we want to uniformly distribute broker’s resources across all the namespaces then, we can configure `dispatchingMessageRatePerNamespace` and start broker with configured throttling-limit, which will apply to all broker owned namespaces. However, namespace with already configured throttling will override broker’s default limit while dispatching the message.

Following [broker-configuration](https://github.com/apache/incubator-pulsar/blob/master/conf/broker.conf) forces broker to apply throttling-limit 1000 to each namespace. By default, value of this configuration is -1 which disables throttling in broker.
```java
dispatchingMessageRatePerNamespace = 1000
```

## Namespace throttling configuration
We can override broker’s default throttling-limit for a namespace if that requires setting value higher or lower than default limit. Throttling-limit for a namespace can be configured per cluster and will be immediately effective after configuring it.

Following configuration sets dispatching throttling-limit of a namespace
```
pulsar-admin namespaces <property/cluster/namespace> set-dispatch-throttling <message-rate-threshold>
```
 
## Alternate approach:

### Dispatch throttling per subscriber:
In above approach, broker does message dispatching throttling per namespace. However, there could be a high possibility that specific subscriber of the topic has a large backlog and over consuming bandwidth of the broker. Therefore, other topics or subscribers under the same namespace can be impacted and suffer due to namespace level throttling. In that case, we can provide an option to configure subscriber level throttling by storing subscriber configuration (under `/managed-ledger/property/cluster/ns/persistent/topic/subscriber/configuration`) into zookeeper. However, it can create an administrative complexity to manage configurations for every topic and subscriber.

### Dispatch throttling per topic:
As described in [#402](https://github.com/apache/incubator-pulsar/issues/402#issuecomment-302434483), we can define rate limiting policy that maps to the list of regex which can match to the topics for which we want to define throttling. This approach can give more granular control by throttling on topic level. However, on a long run it might be complex or difficult to manage rate limiting policies for large number of topics.


### Throttling threshold: Message-rate Vs Bytes-rate
Broker reads data from bookkeeper and dispatches it to consumer in form of message entity. Therefore, it makes more sense to define threshold as message-rate over bytes-rate.





