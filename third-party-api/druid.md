### Druid

Druid 数据库连接池，[详情链接](https://github.com/alibaba/druid/wiki/Druid连接池介绍)。

| 配置参数                                  | 缺省值  | 参数说明                                                     |
| ----------------------------------------- | ------- | ------------------------------------------------------------ |
| initialSize                               | 0       | 初始化连接数量                                               |
| minIdle                                   | 0       | 最小空闲连接数                                               |
| maxActive                                 | 8       | 最大并发连接数                                               |
| maxWait                                   | -1L     | 获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。 |
| timeBetweenEvictionRunsMillis             | 60000   | 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒Destroy线程会检测连接的间隔时间 |
| minEvictableIdleTimeMillis                | 1800000 | 配置一个连接在池中最小生存的时间，单位是毫秒                 |
| validationQuery                           | null    | 用来检测连接是否有效的sql，要求是一个查询语句                |
| testOnBorrow                              | FALSE   | 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。 |
| testOnReturn                              | FALSE   | 归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能 |
| testWhileIdle                             | TRUE    | 建议配置为true，不影响性能，并且保证安全性。 申请连接的时候检测，如果空闲时间大于 timeBetweenEvictionRunsMillis， 执行validationQuery检测连接是否有效。 |
| poolPreparedStatements                    | FALSE   | false 是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql5.5以下的版本中没有PSCache功能，建议关闭掉。5.5及以上版本有PSCache，建议开启。 |
| maxPoolPreparedStatementPerConnectionSize | 10      | 要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。单个connnection独享一个statement cache，也就是说maxOpenPreparedStatements是针对单个connection链接的 |

运行机制:

数据库连接池在初始化的时候会创建 `initialSize` 个连接，当有数据库操作时，会从池中取出一个连接。如果当前池中正在使用的连接数等于 `maxActive`，则会等待一段时间，等待其他操作释放掉某一个连接，如果这个等待时间超过了 `maxWait`，则会报错；如果当前正在使用的连接数没有达到 `maxActive`，则判断当前是否空闲连接，如果有则直接使用空闲连接，如果没有则新建立一个连接。在连接使用完毕后，不会将其物理连接关闭，而是将其放入池中等待其他操作复用。 同时连接池内部有机制判断，如果当前的总的连接数少于 `miniIdle`，则会建立新的空闲连接，以保证空闲连接数为 `miniIdle`。如果当前连接池中某个连接在空闲了 `timeBetweenEvictionRunsMillis` 时间后仍然没有使用，则被物理性的关闭掉。有些数据库连接的时候有超时限制（mysql连接在8小时后断开），或者由于网络中断等原因，连接池的连接会出现失效的情况，这时候设置一个 `testWhileIdle` 参数为 `true`，可以保证连接池内部定时检测连接的可用性，不可用的连接会被抛弃或者重建，尽可能保证从连接池中得到的 `Connection` 对象是可用的。当然，为了保证绝对的可用性，也可以使用`testOnBorrow` 为 `true`（即在获取 `Connection` 对象时检测其可用性），不过这样会影响性能。