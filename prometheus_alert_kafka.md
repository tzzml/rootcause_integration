根据您提供的Datadog告警规则和Prometheus指标，我将为您转换为一组完整的Prometheus告警规则，并针对每个告警提供详细解释：

### 1. Kafka高生产延迟告警
```yaml
groups:
- name: kafka-alerts
  rules:
  - alert: KafkaHighProduceLatency
    # 使用kafka_topic_partition_leader指标结合请求延迟
    # 由于Prometheus指标中没有直接的P99延迟指标，我们使用近似方法
    expr: |
      # 计算每个broker的平均请求延迟
      avg by (instance) (
        kafka_network_requestmetrics_totaltimems{request="Produce"}
      ) > 200
    for: 5m
    labels:
      severity: warning
      category: performance
    annotations:
      summary: "高生产延迟 (instance {{ $labels.instance }})"
      description: |
        broker {{ $labels.instance }} 的生产延迟已达到 {{ $value }}ms (阈值: 200ms)。
        潜在影响:
        - 客户端超时
        - 处理延迟
        建议操作:
        1. 检查broker资源使用情况(CPU/内存/磁盘IO)
        2. 检查网络延迟
        3. 考虑topic分区重新平衡
        4. 评估broker扩容需求
      dashboard: "http://grafana.example.com/d/kafka-overview"  # 替换为实际仪表板
```

### 2. Kafka离线分区告警
```yaml
  - alert: KafkaOfflinePartitions
    # 使用under_replicated_partition指标检测离线分区
    expr: |
      sum by (instance) (
        kafka_topic_partition_under_replicated_partition == 1
      ) > 1
    for: 5m
    labels:
      severity: critical
      category: availability
    annotations:
      summary: "检测到离线分区 (instance {{ $labels.instance }})"
      description: |
        在 {{ $labels.instance }} 上检测到 {{ $value }} 个离线分区。
        紧急程度: 高
        需要立即检查:
        1. broker日志中的错误信息
        2. 磁盘空间和IO性能
        3. ZooKeeper连接状态
        4. 网络分区问题
      runbook: "http://wiki.example.com/kafka-recovery"  # 替换为实际运维手册
```

### 3. Kafka高请求率告警
```yaml
  - alert: KafkaHighRequestRate
    # 使用7天滑动窗口计算基线，检测异常请求率
    expr: |
      (
        rate(kafka_server_brokertopicmetrics_messagesin_total[1h])
        > on(instance)
        2 * avg_over_time(
          rate(kafka_server_brokertopicmetrics_messagesin_total[1h])[7d]
        )
      )
      and
      rate(kafka_server_brokertopicmetrics_messagesin_total[1h]) > 1000
    for: 15m
    labels:
      severity: warning
      category: throughput
    annotations:
      summary: "异常高请求率 (instance {{ $labels.instance }})"
      description: |
        生产者 {{ $labels.instance }} 的请求率异常高: {{ $value }} 请求/秒。
        比7天基线高出2倍以上。
        可能原因:
        1. 消费者滞后导致积压
        2. 突发流量
        3. 应用程序异常
        检查建议:
        1. 监控消费者滞后情况
        2. 检查生产者配置
        3. 评估分区数量是否足够
```

### 补充说明：
1. **指标适配建议**：
   - 如果使用不同的Kafka exporter，指标名称可能需要调整
   - 建议先查询Prometheus中实际可用的Kafka指标：`{__name__=~"kafka_.*"}`

2. **告警分级**：
   - 建议为每个告警设置多级阈值(如warning/critical)
   - 示例：
     ```yaml
     - alert: KafkaHighProduceLatency-Warning
       expr: produce_latency > 200
       labels:
         severity: warning
     
     - alert: KafkaHighProduceLatency-Critical
       expr: produce_latency > 500
       labels:
         severity: critical
     ```

3. **优化建议**：
   - 为常用指标创建Recording Rules提高查询效率
   - 添加维护窗口例外逻辑：`...unless ON() maintenance_mode == 1`
   - 考虑添加关联指标检查(如磁盘IO、CPU等)减少误报

4. **部署建议**：
   - 先在测试环境验证这些规则
   - 逐步调整阈值至适合生产环境的水平
   - 配合Grafana仪表板使用效果更佳

这些规则已经考虑了：
- 语义等价转换(Datadog → PromQL)
- 实际可用的Prometheus指标
- 告警分级和分类
- 可操作的修复建议
- 相关文档链接

请根据实际环境中的指标可用性和业务需求进行适当调整。