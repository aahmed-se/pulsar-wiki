# Message dispatch throttling

 * **Status**: Implemented
 * **Pull Request**: [#634](https://github.com/apache/incubator-pulsar/pull/634)

## Motivation
Producers and consumers can interact with broker on high volumes of data. This can monopolize broker resources and cause network saturation, which can have adverse effects on other topics owned by that broker. Throttling based on resource consumption can protect against these issues and it can play important role for large multi-tenant clusters where a small set of clients using high volumes of data can easily degrade performance of other clients.
## Message throttling
### Consumer: 
Throttling limit can be configured at namespace level by defining message-rate threshold which will apply to each of the topic under that namespace. Sometimes, namespace with large message backlog can continuously drain messages with multiple connected consumers, which can easily over utilize broker’s bandwidth. Therefore, we can configure message-rate for that namespace which will enforce configured message-rate to all the topics under that namespace and each topic under that namespace can dispatch a maximum configured messages per second before it gets throttled by broker.

By default, broker does not throttle message dispatching for any namespace unless cluster is configured with default throttling-limit or namespace has been configured as a part of namespace policy in a cluster. We can configure throttling limit for a specific namespace using an admin-api.

### Producer: 
Broker already has capability to throttle number of publish messages by reading only fixed number of in flight messages per client connection, which can protect broker if client tries to publish significantly large number of messages.

## Broker changes:

Broker supports message-dispatch throttling for all types of subscriptions. Message-rate limit applies across all the subscriptions for a loaded topic. Therefore, broker initializes rate-limiter per topic and rate-limiter distributes permits across all the subscriptions while dispatching the message. So, subscription only reads available messages if rate-limiter has permits available else subscription backs off and schedule next read after few milliseconds. If subscription doesn’t have backlog and it has already caught up then it reads published messages from the broker’s cache and it doesn’t have to read from the bookie. Therefore, broker doesn’t apply message-rate limiting for the subscribers which are already caught up with producers and consuming messages without building a backlog.
 

## Cluster throttling configuration
By default, broker does not throttle message dispatching for any of the topics. However, if we want to uniformly distribute broker’s resources across all the topics then, we can configure default message-rate at cluster level and it will be effective immediately after configuring it . This default message-rate configured in a cluster will apply to all the topics of all the brokers serving in that cluster. However, namespace with already configured throttling will override cluster’s default limit while dispatching messages for all the topics under that namespace.

Following configuration force every broker in the cluster to set dispatch throttling-limit 1000 for each topic served by the broker. By default, value of this configuration is 0 which disables default throttling in all the brokers of that cluster.

```shell
brokers update-dynamic-config --config messageDispatchRatePerTopic --value 0
```


## Namespace throttling configuration
We can always override the default throttling-limit for namespace topics that need a higher or lower message-rate. We can configure throttling-limit for all the topics under a specific namespace and it will be immediately effective after configuring it. Also, throttling-limit for a namespace topics will be configured per cluster that gives flexibility to configure specific message-rate for the namespace in each cluster.

Following configuration sets dispatching throttling-limit for all the topics under that namespace.
```
pulsar-admin namespaces <property/cluster/namespace> set-msg-rate <message-rate-threshold>
```

 
## Alternate approach:

### Dispatch throttling per subscriber:
In above approach, broker does message dispatching throttling per namespace. However, there could be a high possibility that specific subscriber of the topic has a large backlog and over consuming bandwidth of the broker. Therefore, other topics or subscribers under the same namespace can be impacted and suffer due to namespace level throttling. In that case, we can provide an option to configure subscriber level throttling by storing subscriber configuration (under `/managed-ledger/property/cluster/ns/persistent/topic/subscriber/configuration`) into zookeeper. However, it can create an administrative complexity to manage configurations for every topic and subscriber.

### Dispatch throttling per topic:
As described in [#402](https://github.com/apache/incubator-pulsar/issues/402#issuecomment-302434483), we can define rate limiting policy that maps to the list of regex which can match to the topics for which we want to define throttling. This approach can give more granular control by throttling on topic level. However, on a long run it might be complex or difficult to manage rate limiting policies for large number of topics.


### Throttling threshold: Message-rate Vs Bytes-rate
Broker reads data from bookkeeper and dispatches it to consumer in form of message entity. Therefore, it makes more sense to define threshold as message-rate over bytes-rate.
