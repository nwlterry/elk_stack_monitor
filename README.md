# elk_stack_monitor

Kibana Stack Monitoring assets for the production Elasticsearch cluster.

Used together with the OS-patch rolling restart:
https://github.com/nwlterry/elk_cluster_post_os_patching_restart

Index pattern: `metrics-elasticsearch.stack_monitoring*`

## Rules

| Saved object / name | What it fires on | Slack |
|---------------------|------------------|-------|
| `Elasticsearch Node Monitoring Alert` | Disk used % > 85 per `elasticsearch.node.name` (5m) | yes |
| `Elasticsearch Node Down Alert` | Cluster reports fewer than **16** ES nodes, or a known node name is missing from stack-monitoring docs for 5m | yes |

Both rules are tagged `elasticsearch`, `monitoring`, `infrastructure`, **`maintenance-mute`**.

The rolling-restart playbook **disables** every Kibana rule whose name is in `stack_monitor_rule_names` or whose tags include `maintenance-mute` before the first reboot, and **enables** them again in post-check after the cluster is green and all 16 ES nodes are back.

That is required: a rolling OS reboot makes one node leave `_cat/nodes` and stops stack-monitoring documents for that node for several minutes. Without a mute window the node-down rule (and often "nodes changed" / "missing monitoring data") pages Slack on every host.

## Files

| File | Purpose |
|------|--------|
| `elasticsearch_monitoring_alert_rule.txt` | Disk-usage rule + Slack connector export |
| `elasticsearch_node_down_alert_rule.txt` | Node-down / missing-node rule export |
| `elasticsearch_monitoring_dashboard.txt` | Node status / disk / uptime dashboard |
| `test_in_dev_tools` | Disk-usage aggregation to paste in Console |

## Import

Kibana → Stack Management → Saved Objects → Import the `.txt` JSON exports.
Confirm the Slack connector webhook is real (the sample URL is a placeholder).

After import, confirm each rule is **enabled** and tagged `maintenance-mute`.

## Maintenance window (OS patching)

Do not disable rules by hand if you use the playbook. It calls:

```text
GET  {kibana}/api/alerting/rules/_find
POST {kibana}/api/alerting/rule/{id}/_disable   # precheck
POST {kibana}/api/alerting/rule/{id}/_enable    # postcheck
```

If a rolling restart is aborted, re-enable alerts:

```bash
ansible-playbook playbooks/rolling_restart.yml --tags unmute --ask-vault-pass
```

Built-in Stack Monitoring alerts that should use the same tag or be listed in `stack_monitor_rule_names`:

- Elasticsearch nodes changed
- Missing monitoring data
- Elasticsearch cluster status
