告警规则样例：

```
groups:
- name: example
  rules:
  - alert: HighNodeCpuUsage
    expr: node_cpu_usage > 0.5
    for: 10m
    labels:
      severity: page
    annotations:
      summary: High cpu usage
      sulotion: Waiting for self-healing
```

在prometheus 启动文件中加入配置项：
```
rule_files:
  - /etc/prometheus/*.rules # 自定义文件路径， *为通配符
```

当满足告警触发条件时即可触发告警。

prometheus只负责生成告警：
告警的处理与发送交给alertmanager。
```
route: # 告警路由
  receiver: 'default-receiver' # 告警接收者
  group_wait: 30s              # 告警触发之前等待聚合时间
  group_interval: 5m           # 分组间隔
  repeat_interval: 4h          # 重复告警发送间隔
  group_by: [cluster, alertname]  # 以label 中的 cluster和alertname 
  routes:                         #
  - receiver: 'database-pager'    
    group_wait: 10s
    matchers:
    - service=~"mysql|cassandra"
  - receiver: 'frontend-pager'
    group_by: [product, environment]
    matchers:
    - team="frontend"
  - receiver: 'dev-pager'
    matchers:
      - service="inhouse-service"
    mute_time_intervals:
      - offhours
      - holidays
    continue: true # continue 参数表示是否在匹配上此路由后继续下一个路由匹配··
  - receiver: 'on-call-pager'
    matchers:
      - service="inhouse-service"
    active_time_intervals:
      - offhours
      - holidays
```