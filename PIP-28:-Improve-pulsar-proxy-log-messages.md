* Status: **discussion**
* Author: [Samuel Sun](https://github.com/foreversunyao)
* Pull Request: 
* Mailing List Discussion: 
* Release: N/A

## Motivation

Pulsar Proxy is almost a gateway for all pulsar requests, it could be useful if it can record more details for the traffic, like source, target, session id, response time for each request, even for request message body. For request message body, we need a "switch" to enable/disable this, due to it could make proxy a bit slower.

Currently we have for proxy: 
 - pulsar_proxy_active_connections
 - pulsar_proxy_new_connections
 - pulsar_proxy_rejected_connections
 - pulsar_proxy_binary_ops
 - pulsar_proxy_binary_bytes
 - session_id,source_ip:port,target_ip:port for new connection created by ConnectionPool.java

What we will have in proxy log for each request to Proxy from client:
 - **client_ip:port**
 - **proxy_ip:port**
 - **broker_ip:port**
 - **session_id**
 - **response_time** from proxy sending request to broker to proxy received ack from broker, basically processing time cost, including producer/consumer
 - **topic name**
 - **msg body** could be plain text after decryption
 
And also a new config for enable/disable msg body

Log Example could be:

**producer**:
[pulsar-discovery-io-2-1] INFO org.apache.pulsar.proxy.server.ProxyConnection - [**C:10.194.243.2:55605**, id: 0x6a4fabee, L:/10.194.243.83:45603 - R:10.194.243.97:6650, **3ms**, **T:tenant/namespace/topic**] **This is a test msg**

**consumer**:
[pulsar-discovery-io-2-1] INFO  org.apache.pulsar.proxy.server.ProxyConnection - [**C:10.194.243.2:55605**, id: 0x6a4fabee, L:/10.194.243.83:45603 - R:10.194.243.97:6650, **3ms**, **T:tenant/namespace/topic**] **subscriptionA**

C: stands for Client
3ms: response time
T: stands for Topic

## code
Potential mainly change these codes:

**ProxyConnection.java and ConnectionPool.java**
main logic for getting client info and keep it proxy, record the response time and get request msg boday when proxy send request to broker.

**AdminProxyHandler.java**
enable/disable msg body output

**ProxyService.java**
new metrics process