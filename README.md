# elk_stack_monitor

Kibana Stack Monitoring assets for the production Elasticsearch cluster.

Used together with the OS-patch rolling restart:
https://github.com/nwlterry/elk_cluster_post_os_patching_restart

## Cluster status rules (source of truth)

| File | Kibana rule name | When it fires |
|------|------------------|---------------|
| `elk_cluster_status-informational.json` | `CC \| Elasticsearch \| Cluster Status ( Informational )` | ES query count on `.ds-metrics-elasticsearch.stack_monitoring.cluster_stats-default*` |
| `elk_cluster_status-warning.json` | `DevOps \| Elasticsearch \| Cluster Status ( Warning )` | `elasticsearch.cluster.stats.status` is **red** in the last 5m |

Both rules are tagged `Elasticsearch`, `Monitoring`, `Infrastructure`, and **`maintenance-mute`**.

The rolling-restart playbook disables these two by **exact name** (and anything tagged `maintenance-mute`) before the first host reboot, then enables them after the cluster is green with 16 ES nodes.

Index: `.ds-metrics-elasticsearch.stack_monitoring.cluster_stats-default*`

## Other assets

| File | Purpose |
|------|--------|
| `elasticsearch_monitoring_alert_rule.txt` | Disk-usage rule + Slack connector |
| `elasticsearch_node_down_alert_rule.txt` | Node-count < 16 |
| `elasticsearch_monitoring_dashboard.txt` | Node / disk / uptime dashboard |
| `test_in_dev_tools` | Disk-usage aggregation for Console |

## Import

Kibana → Stack Management → Rules → import or recreate from the JSON. After import, confirm **enabled** and tag `maintenance-mute`.

If a rolling restart is aborted:

```bash
ansible-playbook playbooks/rolling_restart.yml --tags unmute --ask-vault-pass
```
