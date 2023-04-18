### 1、Redis

redis 是单线程的。大的 redis 集群中 `keys` 操作会导致多路复用的 IO 主线程阻塞一段时间，系上服务会停顿。可以考虑使用 `scan` 类操作替代 `keys`，`scan` 可以无阻塞的提取出指定模式的 key 列表，但是可能重复。

与 `keys` 类似，`smembers` 命令也应慎用。

