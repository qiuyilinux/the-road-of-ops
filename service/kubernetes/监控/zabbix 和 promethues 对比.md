| 维度                       | zabbix                                                       | promethues                                                   |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 集群                       | 双主                                                         | 依赖于 kubernetes deployment service                         |
| 监控客户端                 | 需要手动部署控制版本                                         | 基于 kubernetes 可以自动部署                                 |
| 监控手段                   | 图形界面+配置文件                                            | 配置文件                                                     |
| 监控原理                   | node: zabbix_agent<br>pod: metris（cAdvisor）<br>kuberneters: metris | node: node_exporter<br/>pod: metris（cAdvisor）<br/>kuberneters: metris |
| 监控方式                   | 需要根据 metris 得到的信息手动配置(非常麻烦)                 | 原生支持监控 kubernetes                                      |
| 是否可以的独立于kubernetes | 可以                                                         | 可以但没必要                                                 |
| 自动发现                   | 配置支持                                                     | 原生支持                                                     |
| 自动化- 接口获取数据       | 支持                                                         | 支持                                                         |
| 自动化- 接口权限           | 支持                                                         | 原生不支持，可以考虑封装                                     |
| 权限                       | 支持                                                         | grafana 支持                                                 |
| 报警媒介                   | 主流报警媒介都支持                                           | 主流报警媒介都支持                                           |
| 报警策略                   | 基于键值的表达式                                             | 基于 promql 的表达式                                         |
| 数据存储方式               | 支持多种数据库                                               | 时序数据库                                                   |
| 图形展示                   | 原生图形界面                                                 | grafana（很完善，有很多官方提供的展示模板，导入也很放方便）  |

```
TOKEN=$(kubectl -n kube-system get secrets monitoring-secret-token -ojsonpath='{.data.token}' | base64 -d)
curl -k --header "Authorization: Bearer $TOKEN"  https://10.0.9.112:10250/metrics/cadvisor
```

