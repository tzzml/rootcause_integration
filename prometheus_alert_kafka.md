##下面是prometheus的指标说明
| 指标名称 | 说明 | 
|---------|---------| 
| kafka_brokers | Kafka集群中的代理数量 |
| kafka_broker_info | Kafka代理信息 |
| kafka_consumergroup_current_offset_sum | 消费者组在主题所有分 区的当前偏移量 |
| kafka_consumergroup_lag_sum | 消费者组在主题所有分区的当前近似延迟 |
| kafka_consumergroup_current_offset | 消费者组在主题/分区的当前偏移量 |
| kafka_consumergroup_lag | 消费者组在主题/分区的当前近似延迟 |
| kafka_consumergroup_members | 消费者组中的成员数量 |
| kafka_topic_partition_current_offset | 代理在主题/分区的当前偏移量 |
| kafka_topic_partition_in_sync_replica | 此主题/分区的同步副本 数 |
| kafka_topic_partition_leader | 此主题/分区的领导者代理ID |
| kafka_topic_partition_leader_is_preferred | 如果主题/分区使用 首选代理则为1 |
| kafka_topic_partition_oldest_offset | 代理在主题/分区的最早偏 移量 |
| kafka_topic_partition_replicas | 此主题/分区的副本数 |
| kafka_topic_partition_under_replicated_partition | 如果主题/分区是副本则为1 |
