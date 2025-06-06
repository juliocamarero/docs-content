---
navigation_title: "Migration and upgrades"
applies_to:
  deployment:
    ess: all
    ece: all
    self: all
    eck: all
mapped_pages:
  - https://www.elastic.co/guide/en/kibana/current/resolve-migrations-failures.html
---

# Troubleshoot {{kib}} migration and upgrades [resolve-migrations-failures]

Migrating {{kib}} primarily involves migrating saved object documents to be compatible with the new version.


## Saved object migration failures [_saved_object_migration_failures]

If {{kib}} unexpectedly terminates while migrating a saved object index, {{kib}} automatically attempts to perform the migration again when the process restarts. Do not delete any saved objects indices to fix a failed migration. Unlike previous versions, {{kib}} 7.12.0 and later does not require deleting indices to release a failed migration lock.

If upgrade migrations fail repeatedly, refer to [preparing for migration](../../deploy-manage/upgrade/prepare-to-upgrade.md). When you address the root cause for the migration failure, {{kib}} automatically retries the migration. If you’re unable to resolve a failed migration, contact Support.


## Corrupt saved objects [_corrupt_saved_objects]

To find and remedy problems caused by corrupt documents, we highly recommend testing your {{kib}} upgrade in a development cluster, especially when there are custom integrations that create saved objects in your environment.

Saved objects that are corrupted through manual editing or integrations cause migration failures with a log message, such as `Unable to migrate the corrupt Saved Object document ...`. For a successful upgrade migration, you must fix or delete corrupt documents.

For example, you receive the following error message:

```sh
Unable to migrate the corrupt saved object document with _id: 'marketing_space:dashboard:e3c5fc71-ac71-4805-bcab-2bcc9cc93275'. To allow migrations to proceed, please delete this document from the [.kibana_7.12.0_001] index.
```

To delete the documents that cause migrations to fail, take the following steps:

1. Create a role as follows:

    ```console
    PUT _security/role/grant_kibana_system_indices
    {
      "indices": [
        {
          "names": [
            ".kibana*"
          ],
          "privileges": [
            "all"
          ],
          "allow_restricted_indices": true
        }
      ]
    }
    ```

2. Create a user with the role above and `superuser` built-in role:

    ```console
    POST /_security/user/temporarykibanasuperuser
    {
      "password" : "l0ng-r4nd0m-p@ssw0rd",
      "roles" : [ "superuser", "grant_kibana_system_indices" ]
    }
    ```

3. Remove the write block which the migration system has placed on the previous index:

    ```console
    PUT .kibana_7.12.1_001/_settings
    {
      "index": {
        "blocks.write": false
      }
    }
    ```

4. Delete the corrupt document:

    ```console
    DELETE .kibana_7.12.0_001/_doc/marketing_space:dashboard:e3c5fc71-ac71-4805-bcab-2bcc9cc93275
    ```

5. Restart {{kib}}.

    The dashboard with the `e3c5fc71-ac71-4805-bcab-2bcc9cc93275` ID that belongs to the `marketing_space` space **is no longer available**.


You can configure {{kib}} to automatically discard all corrupt objects and transform errors that occur during a migration. When setting the configuration option `migrations.discardCorruptObjects`, {{kib}} will delete the conflicting objects and proceed with the migration. For instance, for an upgrade to 8.4.0, you can define the following setting in kibana.yml:

```yaml
migrations.discardCorruptObjects: "8.4.0"
```

:::{warning}
Enabling the flag above will cause the system to discard all corrupt objects, as well as those causing transform errors. Thus, it is HIGHLY recommended that you carefully review the list of conflicting objects in the logs.
:::

## Documents for unknown saved objects [unknown-saved-object-types]

Migrations will fail if saved objects belong to an unknown saved object type. Unknown saved objects are typically caused by performing manual modifications to the {{es}} index (no longer allowed in 8.x), or by disabling a plugin that had previously created a saved object.

We recommend using the [Upgrade Assistant](../../deploy-manage/upgrade/prepare-to-upgrade/upgrade-assistant.md) to discover and remedy any unknown saved object types. {{kib}} version 7.17.0 deployments containing unknown saved object types will also log the following warning message:

```sh
CHECK_UNKNOWN_DOCUMENTS Upgrades will fail for 8.0+ because documents were found for unknown saved object types. To ensure that future upgrades will succeed, either re-enable plugins or delete these documents from the ".kibana_7.17.0_001" index after the current upgrade completes.
```

If you fail to remedy this, your upgrade to 8.0+ will fail with a message like:

```sh
Unable to complete saved object migrations for the [.kibana] index: Migration failed because some documents were found which use unknown saved object types: someType,someOtherType

To proceed with the migration you can configure Kibana to discard unknown saved objects for this migration.
```

To proceed with the migration, re-enable any plugins that previously created these saved objects. Alternatively, carefully review the list of unknown saved objects in the {{kib}} log entry. If the corresponding disabled plugins and their associated saved objects will no longer be used, they can be deleted by setting the configuration option `migrations.discardUnknownObjects` to the version you are upgrading to. For instance, for an upgrade to 8.4.0, you can define the following setting in kibana.yml:

```yaml
migrations.discardUnknownObjects: "8.4.0"
```


## Incompatible settings or mappings [_incompatible_settings_or_mappings]

Matching index templates that specify `settings.refresh_interval` or `mappings` are known to interfere with {{kib}} upgrades. This can happen when index templates are defined manually.

To make sure the index templates won’t apply to new `.kibana*` indices, narrow down the {{data-sources}} of any user-defined index templates.


## Incompatible `xpack.tasks.index` configuration setting [_incompatible_xpack_tasks_index_configuration_setting]

In {{kib}} 7.5.0 and earlier, when the task manager index is set to `.tasks` with the configuration setting `xpack.tasks.index: ".tasks"`, upgrade migrations fail. In {{kib}} 7.5.1 and later, the incompatible configuration setting prevents upgrade migrations from starting.


## Repeated time-out requests that eventually fail [_repeated_time_out_requests_that_eventually_fail]

Migrations get stuck in a loop of retry attempts waiting for index yellow status that’s never reached. In the CLONE_TEMP_TO_TARGET or CREATE_REINDEX_TEMP steps, you might see a log entry similar to:

```sh
"Action failed with [index_not_yellow_timeout] Timeout waiting for the status of the [.kibana_8.1.0_001] index to become "yellow". Retrying attempt 1 in 2 seconds."
```

The process is waiting for a yellow index status. There are two known causes:

* Cluster hits the low watermark for disk usage
* Cluster has [routing allocation disabled](#routing-allocation-disabled)

Before retrying the migration, inspect the output of the `_cluster/allocation/explain?index=${targetIndex}` API to identify why the index isn’t yellow:

```console
GET _cluster/allocation/explain
{
  "index": ".kibana_8.1.0_001",
  "shard": 0,
  "primary": true,
}
```

If the cluster exceeded the low watermark for disk usage, the output should contain a message similar to this:

```sh
"The node is above the low watermark cluster setting [cluster.routing.allocation.disk.watermark.low=85%], using more disk space than the maximum allowed [85.0%], actual free: [11.692661332965082%]"
```

Refer to the {{es}} guide for how to [fix common cluster issues](/troubleshoot/elasticsearch/fix-watermark-errors.md).

If routing allocation is the issue, the `_cluster/allocation/explain` API will return an entry similar to this:

```sh
"allocate_explanation" : "cannot allocate because allocation is not permitted to any of the nodes"
```


## Routing allocation disabled or restricted [routing-allocation-disabled]

Upgrade migrations fail because routing allocation is disabled or restricted (`cluster.routing.allocation.enable: none/primaries/new_primaries`), which causes {{kib}} to log errors such as:

```sh
Unable to complete saved object migrations for the [.kibana] index: [incompatible_cluster_routing_allocation] The elasticsearch cluster has cluster routing allocation incorrectly set for migrations to continue. To proceed, please remove the cluster routing allocation settings with PUT /_cluster/settings {"transient": {"cluster.routing.allocation.enable": null}, "persistent": {"cluster.routing.allocation.enable": null}}
```

To get around the issue, remove the transient and persisted routing allocation settings:

```console
PUT /_cluster/settings
{
  "transient": {
    "cluster.routing.allocation.enable": null
  },
  "persistent": {
    "cluster.routing.allocation.enable": null
  }
}
```


## {{es}} cluster shard limit exceeded [cluster-shard-limit-exceeded]

When upgrading, {{kib}} creates new indices requiring a small number of new shards. If the amount of open {{es}} shards approaches or exceeds the {{es}} `cluster.max_shards_per_node` setting, {{kib}} is unable to complete the upgrade. Ensure that {{kib}} is able to add at least 10 more shards by removing indices to clear up resources, or by increasing the `cluster.max_shards_per_node` setting.

For more information, refer to the documentation on [total shards per node](elasticsearch://reference/elasticsearch/index-settings/total-shards-per-node.md).
