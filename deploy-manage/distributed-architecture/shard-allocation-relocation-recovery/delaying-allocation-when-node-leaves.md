---
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/delayed-allocation.html
applies_to:
  stack:
---

# Delaying allocation when a node leaves [delayed-allocation]

When a node leaves the cluster for whatever reason, intentional or otherwise, the master reacts by:

* Promoting a replica shard to primary to replace any primaries that were on the node.
* Allocating replica shards to replace the missing replicas (assuming there are enough nodes).
* Rebalancing shards evenly across the remaining nodes.

These actions are intended to protect the cluster against data loss by ensuring that every shard is fully replicated as soon as possible.

Even though we throttle concurrent recoveries both at the [node level](elasticsearch://reference/elasticsearch/configuration-reference/index-recovery-settings.md) and at the [cluster level](elasticsearch://reference/elasticsearch/configuration-reference/cluster-level-shard-allocation-routing-settings.md#cluster-shard-allocation-settings), this shard-shuffle can still put a lot of extra load on the cluster which may not be necessary if the missing node is likely to return soon. Imagine this scenario:

* Node 5 loses network connectivity.
* The master promotes a replica shard to primary for each primary that was on Node 5.
* The master allocates new replicas to other nodes in the cluster.
* Each new replica makes an entire copy of the primary shard across the network.
* More shards are moved to different nodes to rebalance the cluster.
* Node 5 returns after a few minutes.
* The master rebalances the cluster by allocating shards to Node 5.

If the master had just waited for a few minutes, then the missing shards could have been re-allocated to Node 5 with the minimum of network traffic. This process would be even quicker for idle shards (shards not receiving indexing requests) which have been automatically [flushed](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-indices-flush).

The allocation of replica shards which become unassigned because a node has left can be delayed with the `index.unassigned.node_left.delayed_timeout` dynamic setting, which defaults to `1m`.

This setting can be updated on a live index (or on all indices):

```console
PUT _all/_settings
{
  "settings": {
    "index.unassigned.node_left.delayed_timeout": "5m"
  }
}
```

With delayed allocation enabled, the above scenario changes to look like this:

* Node 5 loses network connectivity.
* The master promotes a replica shard to primary for each primary that was on Node 5.
* The master logs a message that allocation of unassigned shards has been delayed, and for how long.
* The cluster remains yellow because there are unassigned replica shards.
* Node 5 returns after a few minutes, before the `timeout` expires.
* The missing replicas are re-allocated to Node 5 (and sync-flushed shards recover almost immediately).

::::{note}
This setting will not affect the promotion of replicas to primaries, nor will it affect the assignment of replicas that have not been assigned previously. In particular, delayed allocation does not come into effect after a full cluster restart. Also, in case of a master failover situation, elapsed delay time is forgotten (i.e. reset to the full initial delay).
::::

## Cancellation of shard relocation [_cancellation_of_shard_relocation]

If delayed allocation times out, the master assigns the missing shards to another node which will start recovery. If the missing node rejoins the cluster, and its shards still have the same sync-id as the primary, shard relocation will be cancelled and the synced shard will be used for recovery instead.

For this reason, the default `timeout` is set to just one minute: even if shard relocation begins, cancelling recovery in favour of the synced shard is cheap.

## Monitoring delayed unassigned shards [_monitoring_delayed_unassigned_shards]

The number of shards whose allocation has been delayed by this timeout setting can be viewed with the [cluster health API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-cluster-health):

```console
GET _cluster/health <1>
```

1. This request will return a `delayed_unassigned_shards` value.

## Removing a node permanently [_removing_a_node_permanently]

If a node is not going to return and you would like {{es}} to allocate the missing shards immediately, just update the timeout to zero:

```console
PUT _all/_settings
{
  "settings": {
    "index.unassigned.node_left.delayed_timeout": "0"
  }
}
```

You can reset the timeout as soon as the missing shards have started to recover.