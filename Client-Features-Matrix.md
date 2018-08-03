

# Pulsar client feature matrix

This matrix is updated with Pulsar 2.1 release.

| Feature                                   | Java | C++ | Go | Python | WebSocket |
|:------------------------------------------|:----:|:---:|:--:|:------:|:---------:|
| Partitioned topics                        |  ✅   |  ✅  | ✅  |   ✅    |     ✅     |
| Batching                                  |  ✅   |  ✅  | ✅  |   ✅    |     ✅     |
| Compression                               |  ✅   |  ✅  | ✅  |   ✅    |     ✅     |
| TLS                                       |  ✅   |  ✅  | ✅  |   ✅    |     ✅     |
| Authentication                            |  ✅   |  ✅  | ✅  |   ✅    |     ✅     |
| Reader API                                |  ✅   |  ✅  | ✅  |   ✅    |     ✅     |
| Proxy Support                             |  ✅   |  ✅  | ✅  |   ✅    |     ✅     |
| Effectively-Once                          |  ✅   |  ✅  | ✅  |   ✅    |     ❌     |
| Schema                                    |  ✅   |     |    |        |     ❌     |
| Consumer seek                             |  ✅   |  ✅  |    |   ✅    |     ❌     |
| Multi-topics consumer                     |  ✅   |     |    |        |     ❌     |
| Topics regex consumer                     |  ✅   |     |    |        |     ❌     |
| Compacted topics                          |  ✅   |  ✅  |    |   ✅    |      ❌      |
| User defined properties producer/consumer |  ✅   |     |    |        |     ❌     |
| Reader hasMessageAvailable                |  ✅   |  ✅   |    |   ✅     |     ❌     |
| Hostname verification                     |  ✅   |     |    |        |     ❌     |

## In progress

This matrix keeps the features that are already developed (or under developing) in master, but not yet released.


| Feature                                   | Java | C++ | Go | Python | WebSocket |
|:------------------------------------------|:----:|:---:|:--:|:------:|:---------:|
| Multi-topics consumer                     |  ✅   |  ✅   |    |    ✅    |     ❌     |
| Topics regex consumer                     |  ✅   |  [#2219](https://github.com/apache/incubator-pulsar/pull/2219)   |    |   [#2219](https://github.com/apache/incubator-pulsar/pull/2219)     |     ❌     |


