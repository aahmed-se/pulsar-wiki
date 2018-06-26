

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
| Compacted topics                          |  ✅   |  ✅  |    |   ✅    |           |
| User defined properties producer/consumer |  ✅   |     |    |        |     ❌     |
| Reader hasMessageAvailable                |  ✅   |     |    |        |     ❌     |
| Hostname verification                     |  ✅   |     |    |        |     ❌     |
