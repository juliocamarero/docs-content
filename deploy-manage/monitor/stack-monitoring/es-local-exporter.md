---
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/local-exporter.html
applies_to:
  deployment:
    self: deprecated 7.16.0
---

# Local exporters [local-exporter]

:::{include} _snippets/legacy-warning.md
:::

The `local` exporter is the default exporter in {{monitoring}}. It routes data back into the same (local) cluster. In other words, it uses the production cluster as the monitoring cluster. For example:

```yaml
xpack.monitoring.exporters.my_local_exporter: <1>
  type: local
```

1. The exporter name uniquely defines the exporter, but it is otherwise unused.


This exporter exists to provide a convenient option when hardware is simply not available. It is also a way for developers to get an idea of what their actions do for pre-production clusters when they do not have the time or resources to provide a separate monitoring cluster. However, this exporter has disadvantages that impact the local cluster:

* All indexing impacts the local cluster and the nodes that hold the monitoring indices' shards.
* Most collectors run on the elected master node. Therefore most indexing occurs with the elected master node as the coordinating node, which is a bad practice.
* Any usage of {{monitoring}} for {{kib}} uses the local cluster’s resources for searches and aggregations, which means that they might not be available for non-monitoring tasks.
* If the local cluster goes down, the monitoring cluster has inherently gone down with it (and vice versa), which generally defeats the purpose of monitoring.

For the `local` exporter, all setup occurs only on the elected master node. This means that if you do not see any monitoring templates or ingest pipelines, the elected master node is having issues or it is not configured in the same way. Unlike the `http` exporter, the `local` exporter has the advantage of accessing the monitoring cluster’s up-to-date cluster state. It can therefore always check that the templates and ingest pipelines exist without a performance penalty. If the elected master node encounters errors while trying to create the monitoring resources, it logs errors, ignores that collection, and tries again after the next collection.

The elected master node is the only node to set up resources for the `local` exporter. Therefore all other nodes wait for the resources to be set up before indexing any monitoring data from their own collectors. Each of these nodes logs a message indicating that they are waiting for the resources to be set up.

One benefit of the `local` exporter is that it lives within the cluster and therefore no extra configuration is required when the cluster is secured with {{stack}} {{security-features}}. All operations, including indexing operations, that occur from a `local` exporter make use of the internal transport mechanisms within {{es}}. This behavior enables the exporter to be used without providing any user credentials when {{security-features}} are enabled.

For more information about the configuration options for the `local` exporter, see [Local exporter settings](elasticsearch://reference/elasticsearch/configuration-reference/monitoring-settings.md#local-exporter-settings).

## Cleaner service [local-exporter-cleaner]

One feature of the `local` exporter, which is not present in the `http` exporter, is a cleaner service. The cleaner service runs once per day at 01:00 AM UTC on the elected master node.

The role of the cleaner service is to clean, or curate, the monitoring indices that are older than a configurable amount of time (the default is `7d`). This cleaner exists as part of the `local` exporter as a safety mechanism. The `http` exporter does not make use of it because it could enable a single misconfigured node to prematurely curate data from other production clusters that share the same monitoring cluster.

In a dedicated monitoring cluster, you can use the cleaner service without having to monitor the monitoring cluster itself. For example:

```yaml
xpack.monitoring.collection.enabled: false <1>
xpack.monitoring.history.duration: 3d <2>
```

1. Disables the collection of data on the monitoring cluster.
2. Lowers the default history duration from `7d` to `3d`. The minimum value is `1d`. This setting can be modified only when using a Gold or higher level license. For the Basic license level, it uses the default of 7 days.


To disable the cleaner service, add a disabled local exporter:

```yaml
xpack.monitoring.exporters.my_local.type: local <1>
xpack.monitoring.exporters.my_local.enabled: false <2>
```

1. Adds a local exporter named `my_local`
2. Disables the local exporter. This also disables the cleaner service.



