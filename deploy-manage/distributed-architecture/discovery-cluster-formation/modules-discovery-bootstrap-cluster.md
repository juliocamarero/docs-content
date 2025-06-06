---
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-bootstrap-cluster.html
applies_to:
  stack:
---

# Bootstrapping a cluster [modules-discovery-bootstrap-cluster]

Starting an {{es}} cluster for the very first time requires the initial set of [master-eligible nodes](../clusters-nodes-shards/node-roles.md#master-node-role) to be explicitly defined on one or more of the master-eligible nodes in the cluster. This is known as *cluster bootstrapping*. This is only required the first time a cluster starts up. Freshly-started nodes that are joining a running cluster obtain this information from the cluster’s elected master.

The initial set of master-eligible nodes is defined in the [`cluster.initial_master_nodes` setting](../../deploy/self-managed/important-settings-configuration.md#initial_master_nodes). This should be set to a list containing one of the following items for each master-eligible node:

* The [node name](../../deploy/self-managed/important-settings-configuration.md#node-name) of the node.
* The node’s hostname if `node.name` is not set, because `node.name` defaults to the node’s hostname. You must use either the fully-qualified hostname or the bare hostname [depending on your system configuration](#modules-discovery-bootstrap-cluster-fqdns).
* The IP address of the node’s [transport publish address](elasticsearch://reference/elasticsearch/configuration-reference/networking-settings.md#modules-network-binding-publishing), if it is not possible to use the `node.name` of the node. This is normally the IP address to which [`network.host`](elasticsearch://reference/elasticsearch/configuration-reference/networking-settings.md#common-network-settings) resolves but [this can be overridden](elasticsearch://reference/elasticsearch/configuration-reference/networking-settings.md#advanced-network-settings).
* The IP address and port of the node’s publish address, in the form `IP:PORT`, if it is not possible to use the `node.name` of the node and there are multiple nodes sharing a single IP address.

Do not set `cluster.initial_master_nodes` on master-ineligible nodes.

::::{important}
After the cluster has formed, remove the `cluster.initial_master_nodes` setting from each node’s configuration and never set it again for this cluster. Do not configure this setting on nodes joining an existing cluster. Do not configure this setting on nodes which are restarting. Do not configure this setting when performing a full-cluster restart.

If you leave `cluster.initial_master_nodes` in place once the cluster has formed then there is a risk that a future misconfiguration may result in bootstrapping a new cluster alongside your existing cluster. It may not be possible to recover from this situation without losing data.

::::

The simplest way to create a new cluster is for you to select one of your master-eligible nodes that will bootstrap itself into a single-node cluster, which all the other nodes will then join. This simple approach is not resilient to failures until the other master-eligible nodes have joined the cluster. For example, if you have a master-eligible node with [node name](../../deploy/self-managed/important-settings-configuration.md#node-name) `master-a` then configure it as follows (omitting `cluster.initial_master_nodes` from the configuration of all other nodes):

```yaml
cluster.initial_master_nodes: master-a
```

For fault-tolerant cluster bootstrapping, use all the master-eligible nodes. For instance, if your cluster has 3 master-eligible nodes with [node names](../../deploy/self-managed/important-settings-configuration.md#node-name) `master-a`, `master-b` and `master-c` then configure them all as follows:

```yaml
cluster.initial_master_nodes:
  - master-a
  - master-b
  - master-c
```

::::{important}
You must set `cluster.initial_master_nodes` to the same list of nodes on each node on which it is set in order to be sure that only a single cluster forms during bootstrapping. If `cluster.initial_master_nodes` varies across the nodes on which it is set then you may bootstrap multiple clusters. It is usually not possible to recover from this situation without losing data.
::::

::::{admonition} Node name formats must match
:name: modules-discovery-bootstrap-cluster-fqdns

The node names used in the `cluster.initial_master_nodes` list must exactly match the `node.name` properties of the nodes. By default the node name is set to the machine’s hostname which may or may not be fully-qualified depending on your system configuration. If each node name is a fully-qualified domain name such as `master-a.example.com` then you must use fully-qualified domain names in the `cluster.initial_master_nodes` list too; conversely if your node names are bare hostnames (without the `.example.com` suffix) then you must use bare hostnames in the `cluster.initial_master_nodes` list. If you use a mix of fully-qualified and bare hostnames, or there is some other mismatch between `node.name` and `cluster.initial_master_nodes`, then the cluster will not form successfully and you will see log messages like the following.

```text
[master-a.example.com] master not discovered yet, this node has
not previously joined a bootstrapped (v7+) cluster, and this
node must discover master-eligible nodes [master-a, master-b] to
bootstrap a cluster: have discovered [{master-b.example.com}{...
```

This message shows the node names `master-a.example.com` and `master-b.example.com` as well as the `cluster.initial_master_nodes` entries `master-a` and `master-b`, and it is clear from this message that they do not match exactly.

::::

## Choosing a cluster name [bootstrap-cluster-name]

The [`cluster.name`](elasticsearch://reference/elasticsearch/configuration-reference/miscellaneous-cluster-settings.md#cluster-name) setting enables you to create multiple clusters which are separated from each other. Nodes verify that they agree on their cluster name when they first connect to each other, and {{es}} will only form a cluster from nodes that all have the same cluster name. The default value for the cluster name is `elasticsearch`, but it is recommended to change this to reflect the logical name of the cluster.

## Auto-bootstrapping in development mode [bootstrap-auto-bootstrap]

By default each node will automatically bootstrap itself into a single-node cluster the first time it starts. If any of the following settings are configured then auto-bootstrapping will not take place:

* `discovery.seed_providers`
* `discovery.seed_hosts`
* `cluster.initial_master_nodes`

To add a new node into an existing cluster, configure `discovery.seed_hosts` or other relevant discovery settings so that the new node can discover the existing master-eligible nodes in the cluster. To bootstrap a new multi-node cluster, configure `cluster.initial_master_nodes` as described in the section on cluster bootstrapping as well as `discovery.seed_hosts` or other relevant discovery settings.

::::{admonition} Forming a single cluster
:name: modules-discovery-bootstrap-cluster-joining

Once an {{es}} node has joined an existing cluster, or bootstrapped a new cluster, it will not join a different cluster. {{es}} will not merge separate clusters together after they have formed, even if you subsequently try and configure all the nodes into a single cluster. This is because there is no way to merge these separate clusters together without a risk of data loss. You can tell that you have formed separate clusters by checking the cluster UUID reported by `GET /` on each node.

If you intended to add a node into an existing cluster but instead bootstrapped a separate single-node cluster then you must start again:

1. Shut down the node.
2. Completely wipe the node by deleting the contents of its [data folder](elasticsearch://reference/elasticsearch/configuration-reference/node-settings.md#data-path).
3. Configure `discovery.seed_hosts` or `discovery.seed_providers` and other relevant discovery settings. Ensure `cluster.initial_master_nodes` is not set on any node.
4. Restart the node and verify that it joins the existing cluster rather than forming its own one-node cluster.

If you intended to form a new multi-node cluster but instead bootstrapped a collection of single-node clusters then you must start again:

1. Shut down all the nodes.
2. Completely wipe each node by deleting the contents of their [data folders](elasticsearch://reference/elasticsearch/configuration-reference/node-settings.md#data-path).
3. Configure `cluster.initial_master_nodes` as described above.
4. Configure `discovery.seed_hosts` or `discovery.seed_providers` and other relevant discovery settings.
5. Restart all the nodes and verify that they have formed a single cluster.
6. Remove `cluster.initial_master_nodes` from every node’s configuration.

::::
