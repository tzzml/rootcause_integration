##下面是datadog的一组告警说明
## [Kafka] High produce latency: {{value}} reqs/s on broker {{instance.name}}
- **查询语句**: avg(last_5m):avg:kafka.request.produce.time.99percentile{*} by {instance} > 200


- **告警内容**: {{#is_alert}}

ALERT: The p99 produce latency on broker {{instance.name}} reached: {{value}}.

{{/is_alert}} 

{{#is_warning}}

WARNING: The p99 produce latency on broker {{instance.name}} reached: {{value}}.

{{/is_warning}} 


**Potential Impacts**

  - Client timeouts
  - Delays in the ability of clients to process their workload
  - Could be a leading indicator that the broker is falling behind
    due to lack of capacity or a performance-impacting incident.

**Recommended Actions**

  - Investigate the state of the broker
  - Consider topic rebalancing if the traffic on a given topic has
    outstripped the resources available to it
  - Consider expanding capacity by adding additional brokers
  - Broker restart or replacement can help in some situations.
    If TCP memory is high, and increasing in correlation with the
    load, this could mean that the disk is struggling to keep up.
    Restarting kafka has shown some immediate benefits when it comes
    to reducing the load.
## [Kafka] Offline partition on {{host.name}}
- **查询语句**: avg(last_5m):avg:kafka.replication.offline_partitions_count{*} > 1


- **告警内容**: Partition without an active leader detected
## [Kafka] High request rate on producer {{host.name}}
- **查询语句**: avg(last_1h):anomalies(avg:kafka.producer.request_rate{*}, 'basic', 2, direction='above', interval=20, alert_window='last_5m', count_default_zero='true') >= 1


- **告警内容**: The request rate on a producer is abnormally high: {{value}} request/s.

