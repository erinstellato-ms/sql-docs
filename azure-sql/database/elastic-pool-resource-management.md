---
title: Resource management in dense elastic pools
description: Manage compute resources in Azure SQL Database elastic pools with many databases.
author: dimitri-furman
ms.author: dfurman
ms.reviewer: wiassaf, mathoma, jackli
ms.date: 3/30/2022
ms.service: azure-sql-database
ms.subservice: elastic-pools
ms.topic: conceptual
ms.custom: sqldbrb=1
---

# Resource management in dense elastic pools
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Azure SQL Database [elastic pools](./elastic-pool-overview.md) is a cost-effective solution for managing many databases with varying resource usage. All databases in an elastic pool share the same allocation of resources, such as CPU, memory, worker threads, storage space, `tempdb`, on the assumption that **only a subset of databases in the pool will use compute resources at any given time**. This assumption allows elastic pools to be cost-effective. Instead of paying for all resources each individual database could potentially need, customers pay for a much smaller set of resources, shared among all databases in the pool.

## Resource governance

Resource sharing requires the system to carefully control resource usage to minimize the "noisy neighbor" effect, where a database with high resource consumption affects other databases in the same elastic pool. Azure SQL Database achieves these goals by implementing [resource governance](resource-limits-logical-server.md#resource-governance). At the same time, the system must provide sufficient resources for features such as high availability and disaster recovery (HADR), backup and restore, monitoring, Query Store, Automatic tuning, etc. to function reliably. 

The primary design goal of elastic pools is to be cost-effective. For this reason, the system intentionally allows customers to create _dense_ pools, that is pools with the number of databases approaching or at the maximum allowed, but with a moderate allocation of compute resources. For the same reason, the system doesn't reserve all potentially needed resources for its internal processes, but allows resource sharing between internal processes and user workloads.

This approach allows customers to use dense elastic pools to achieve adequate performance and major cost savings. However, if the workload against many databases in a dense pool is sufficiently intense, resource contention becomes significant. Resource contention reduces user workload performance, and can negatively impact internal processes.

> [!IMPORTANT]
> In dense pools with many active databases, it may not be feasible to increase the number of databases in the pool up to the maximums documented for [DTU](resource-limits-dtu-elastic-pools.md) and [vCore](resource-limits-vcore-elastic-pools.md) elastic pools.
>
> The number of databases that can be placed in dense pools without causing resource contention and performance problems depends on the number of concurrently active databases, and on resource consumption by user workloads in each database. This number can change over time as user workloads change.
> 
> Additionally, if the min vCores per database, or min DTUs per database setting is set to a value greater than 0, the maximum number of databases in the pool will be implicitly limited. For more information, see [Database properties for pooled vCore databases](resource-limits-vcore-elastic-pools.md#database-properties-for-pooled-databases) and [Database properties for pooled DTU databases](resource-limits-dtu-elastic-pools.md#database-properties-for-pooled-databases).

When resource contention occurs in a densely packed pool, customers can choose one or more of the following actions to mitigate it:

- Tune query workload to reduce resource consumption, or spread resource consumption across multiple databases over time.
- Reduce pool density by moving some databases to another pool, or by making them standalone databases.
- Scale up the pool to get more resources.

For suggestions on how to implement the last two actions, see [Operational recommendations](#operational-recommendations) later in this article. Reducing resource contention benefits both user workloads and internal processes, and lets the system reliably maintain expected level of service.

## Monitoring resource consumption

To avoid performance degradation due to resource contention, customers using dense elastic pools should proactively monitor resource consumption, and take timely action if increasing resource contention starts affecting workloads. Continuous monitoring is important because resource usage in a pool changes over time, due to changes in user workload, changes in data volumes and distribution, changes in pool density, and changes in the Azure SQL Database service.

Azure SQL Database provides several metrics that are relevant for this type of monitoring. Exceeding the recommended average value for each metric indicates resource contention in the pool, and should be addressed using one of the actions mentioned earlier.

To send an alert when pool resource utilization (CPU, data IO, log IO, workers, etc.) exceeds a threshold, consider creating alerts via the [Azure portal](alerts-create.md) or the [Add-AzMetricAlertRulev2](/powershell/module/az.monitor/add-azmetricalertrulev2) PowerShell cmdlet. When monitoring elastic pools, consider also creating alerts for individual databases in the pool if needed in your scenario. For a sample scenario of monitoring elastic pools, see [Monitor and manage performance of Azure SQL Database in a multi-tenant SaaS app](saas-dbpertenant-performance-monitoring.md).

|Metric name|Description|Recommended average value|
|----------|--------------------------------|------------|
|`avg_instance_cpu_percent`|CPU utilization of the SQL process associated with an elastic pool, as measured by the underlying operating system. Available in the [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) view in every database, and in the [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database) view in the `master` database. This metric is also emitted to Azure Monitor, where it is [named](/azure/azure-monitor/essentials/metrics-supported#microsoftsqlserverselasticpools) `sql_instance_cpu_percent`, and can be viewed in Azure portal. This value is the same for every database in the same elastic pool.|Below 70%. Occasional short spikes up to 90% may be acceptable.|
|`max_worker_percent`|[Worker thread](/sql/relational-databases/thread-and-task-architecture-guide) utilization. Provided for each database in the pool, as well as for the pool itself. There are different limits on the number of worker threads at the database level, and at the pool level, therefore monitoring this metric at both levels is recommended. Available in the [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) view in every database, and in the [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database) view in the `master` database. This metric is also emitted to Azure Monitor, where it is [named](/azure/azure-monitor/essentials/metrics-supported#microsoftsqlserverselasticpools) `workers_percent`, and can be viewed in Azure portal.|Below 80%. Spikes up to 100% will cause connection attempts and queries to fail.|
|`avg_data_io_percent`|IOPS utilization for read and write physical IO. Provided for each database in the pool, as well as for the pool itself. There are different limits on the number of IOPS at the database level, and at the pool level, therefore monitoring this metric at both levels is recommended. Available in the [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) view in every database, and in the [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database) view in the `master` database. This metric is also emitted to Azure Monitor, where it is [named](/azure/azure-monitor/essentials/metrics-supported#microsoftsqlserverselasticpools) `physical_data_read_percent`, and can be viewed in Azure portal.|Below 80%. Occasional short spikes up to 100% may be acceptable.|
|`avg_log_write_percent`|Throughput utilizations for transaction log write IO. Provided for each database in the pool, as well as for the pool itself. There are different limits on the log throughput at the database level, and at the pool level, therefore monitoring this metric at both levels is recommended. Available in the [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) view in every database, and in the [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database) view in the `master` database. This metric is also emitted to Azure Monitor, where it is [named](/azure/azure-monitor/essentials/metrics-supported#microsoftsqlserverselasticpools) `log_write_percent`, and can be viewed in Azure portal. When this metric is close to 100%, all database modifications (INSERT, UPDATE, DELETE, MERGE statements, SELECT … INTO, BULK INSERT, etc.) will be slower.|Below 90%. Occasional short spikes up to 100% may be acceptable.|
|`oom_per_second`|The rate of out-of-memory (OOM) errors in an elastic pool, which is an indicator of memory pressure. Available in the [sys.dm_resource_governor_resource_pools_history_ex](/sql/relational-databases/system-dynamic-management-views/sys-dm-resource-governor-resource-pools-history-ex-azure-sql-database) view. See [Examples](#examples) for a sample query to calculate this metric. For more information, see resource limits for [elastic pools using DTUs](resource-limits-dtu-elastic-pools.md) or [elastic pools using vCores](resource-limits-vcore-elastic-pools.md), and [Troubleshoot out of memory errors with Azure SQL Database](troubleshoot-memory-errors-issues.md). If you encounter out of memory errors, review [sys.dm_os_out_of_memory_events](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-out-of-memory-events).|0|
|`avg_storage_percent`|Total storage space used by data in all databases within an elastic pool. Does not include empty space in database files. Available in the [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database) view in the `master` database. This metric is also emitted to Azure Monitor, where it is [named](/azure/azure-monitor/essentials/metrics-supported#microsoftsqlserverselasticpools) `storage_percent`, and can be viewed in Azure portal.|Below 80%. Can approach 100% for pools with no data growth.|
|`avg_allocated_storage_percent`|Total storage space used by database files in storage in all databases within an elastic pool. Includes empty space in database files. Available in the [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database) view in the `master` database. This metric is also emitted to Azure Monitor, where it is [named](/azure/azure-monitor/essentials/metrics-supported#microsoftsqlserverselasticpools) `allocated_data_storage_percent`, and can be viewed in Azure portal.|Below 90%. Can approach 100% for pools with no data growth.|
|`tempdb_log_used_percent`|Transaction log space utilization in the `tempdb` database. Even though temporary objects created in one database are not visible in other databases in the same elastic pool, `tempdb` is a shared resource for all databases in the same pool. A long running or orphaned transaction in `tempdb` started from one database in the pool can consume a large portion of transaction log, and cause failures for queries in other databases in the same pool. Derived from [sys.dm_db_log_space_usage](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-log-space-usage-transact-sql) and [sys.database_files](/sql/relational-databases/system-catalog-views/sys-database-files-transact-sql) views. This metric is also emitted to Azure Monitor, and can be viewed in Azure portal. See [Examples](#examples) for a sample query to return the current value of this metric.|Below 50%. Occasional spikes up to 80% are acceptable.|

In addition to these metrics, Azure SQL Database provides a view that returns actual resource governance limits, as well as additional views that return resource utilization statistics at the resource pool level, and at the workload group level.

|View name|Description|  
|-----------------|--------------------------------|  
|[sys.dm_user_db_resource_governance](/sql/relational-databases/system-dynamic-management-views/sys-dm-user-db-resource-governor-azure-sql-database)|Returns actual configuration and capacity settings used by resource governance mechanisms in the current database or elastic pool.|
|[sys.dm_resource_governor_resource_pools](/sql/relational-databases/system-dynamic-management-views/sys-dm-resource-governor-resource-pools-transact-sql)|Returns information about the current resource pool state, the current configuration of resource pools, and cumulative resource pool statistics.|
|[sys.dm_resource_governor_workload_groups](/sql/relational-databases/system-dynamic-management-views/sys-dm-resource-governor-workload-groups-transact-sql)|Returns cumulative workload group statistics and the current configuration of the workload group. This view can be joined with sys.dm_resource_governor_resource_pools on the `pool_id` column to get resource pool information.|
|[sys.dm_resource_governor_resource_pools_history_ex](/sql/relational-databases/system-dynamic-management-views/sys-dm-resource-governor-resource-pools-history-ex-azure-sql-database)|Returns resource pool utilization statistics for recent history, based on the number of snapshots available. Each row represents a time interval. The duration of the interval is provided in the `duration_ms` column. The `delta_` columns return the change in each statistic during the interval.|
|[sys.dm_resource_governor_workload_groups_history_ex](/sql/relational-databases/system-dynamic-management-views/sys-dm-resource-governor-workload-groups-history-ex-azure-sql-database)|Returns workload group utilization statistics for recent history, based on the number of snapshots available. Each row represents a time interval. The duration of the interval is provided in the `duration_ms` column. The `delta_` columns return the change in each statistic during the interval.|

> [!TIP]
> To query these and other dynamic management views using a principal other than server administrator, add this principal to the `##MS_ServerStateReader##` [server role](security-server-roles.md).

These views can be used to monitor resource utilization and troubleshoot resource contention in near real-time. User workload on the primary and readable secondary replicas, including geo-replicas, is classified into the `SloSharedPool1` resource pool and `UserPrimaryGroup.DBId[N]` workload group, where `N` stands for the database ID value.

In addition to monitoring current resource utilization, customers using dense pools can maintain historical resource utilization data in a separate data store. This data can be used in predictive analysis to proactively manage resource utilization based on historical and seasonal trends.

## Operational recommendations

**Leave sufficient resource headroom**. If resource contention and performance degradation occur, mitigation may involve moving some databases out of the affected elastic pool, or scaling up the pool, as noted earlier. However, these actions require additional compute resources to complete. In particular, for Premium and Business Critical pools, these actions require transferring all data for the databases being moved, or for all databases in the elastic pool if the pool is scaled up. Data transfer is a long running and resource-intensive operation. If the pool is already under high resource pressure, the mitigating operation itself will degrade performance even further. In extreme cases, it may not be possible to solve resource contention via database move or pool scale-up because the required resources are not available. In this case, temporarily reducing query workload on the affected elastic pool may be the only solution.

Customers using dense pools should closely monitor resource utilization trends as described earlier, and take mitigating action while metrics remain within the recommended ranges and there are still sufficient resources in the elastic pool.

Resource utilization depends on multiple factors that change over time for each database and each elastic pool. Achieving optimal price/performance ratio in dense pools requires continuous monitoring and rebalancing, that is moving databases from more utilized pools to less utilized pools, and creating new pools as necessary to accommodate increased workload.

> [!NOTE]
> For DTU elastic pools, the eDTU metric at the pool level is not a MAX or a SUM of individual database utilization. It is derived from the utilization of various pool level metrics. Pool level resource limits may be higher than individual database level limits, so it is possible that an individual database can reach a specific resource limit (CPU, data IO, log IO, etc.), even when the eDTU reporting for the pool indicates no limit been reached. 

**Do not move "hot" databases**. If resource contention at the pool level is primarily caused by a small number of highly utilized databases, it may be tempting to move these databases to a less utilized pool, or make them standalone databases. However, doing this while a database remains highly utilized is not recommended, because the move operation will further degrade performance, both for the database being moved, and for the entire pool. Instead, either wait until high utilization subsides, or move less utilized databases instead to relieve resource pressure at the pool level. But moving databases with very low utilization does not provide any benefit in this case, because it does not materially reduce resource utilization at the pool level.

**Create new databases in a "quarantine" pool**. In scenarios where new databases are created frequently, such as applications using the tenant-per-database model, there is risk that a new database placed into an existing elastic pool will unexpectedly consume significant resources and affect other databases and internal processes in the pool. To mitigate this risk, create a separate "quarantine" pool with ample allocation of resources. Use this pool for new databases with yet unknown resource consumption patterns. Once a database has stayed in this pool for a business cycle, such as a week or a month, and its resource consumption is known, it can be moved to a pool with sufficient capacity to accommodate this additional resource usage.

**Monitor both used and allocated space**. When allocated pool space (total size of all database files in storage for all databases in a pool) reaches maximum pool size, out-of-space errors may occur. If allocated space trends high and is on track to reach maximum pool size, mitigation options include:
- Move some databases out of the pool to reduce total allocated space
- [Shrink](file-space-manage.md) database files to reduce empty allocated space in files
- Scale up the pool to a service objective with a larger maximum pool size

If used pool space (total size of data in all databases in a pool, not including empty space in files) trends high and is on track to reach maximum pool size, mitigation options include:
- Move some databases out of the pool to reduce total used space
- Move (archive) data outside of the database, or delete no longer needed data
- Implement [data compression](/sql/relational-databases/data-compression/data-compression)
- Scale up the pool to a service objective with a larger maximum pool size

**Avoid overly dense servers**. Azure SQL Database [supports](./resource-limits-logical-server.md) up to 5000 databases per server. Customers using elastic pools with thousands of databases may consider placing multiple elastic pools on a single server, with the total number of databases up to the supported limit. However, servers with many thousands of databases create operational challenges. Operations that require enumerating all databases on a server, for example viewing databases in the portal, will be slower. Operational errors, such as incorrect modification of server level logins or firewall rules, will affect a larger number of databases. Accidental deletion of the server will require assistance from Microsoft Support to recover databases on the deleted server, and will cause a prolonged outage for all affected databases.

Limit the number of databases per server to a lower number than the maximum supported. In many scenarios, using up to 1000-2000 databases per server is optimal. To reduce the likelihood of accidental server deletion, place a [delete lock](/azure/azure-resource-manager/management/lock-resources) on the server or its resource group.

## Examples

### View individual database capacity settings

Use the `sys.dm_user_db_resource_governance` dynamic management view to view the actual configuration and capacity settings used by resource governance in the current database or elastic pool. For more information, see [sys.dm_user_db_resource_governance](/sql/relational-databases/system-dynamic-management-views/sys-dm-user-db-resource-governor-azure-sql-database). 

Run this query in any database in an elastic pool. All databases in the pool have the same resource governance settings. 

```sql
SELECT * FROM sys.dm_user_db_resource_governance AS rg
WHERE database_id = DB_ID();
```

### Monitoring overall elastic pool resource consumption

Use the `sys.elastic_pool_resource_stats` system catalog view to monitor the resource consumption of the entire pool. For more information, see [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database).

This sample query to view the last 10 minutes should be run in the `master` database of the logical Azure SQL server containing the desired elastic pool. 

```sql
SELECT * FROM sys.elastic_pool_resource_stats AS rs
WHERE rs.start_time > DATEADD(mi, -10, SYSUTCDATETIME()) 
AND rs.elastic_pool_name = '<elastic pool name>';
```

### Monitoring individual database resource consumption

Use the `sys.dm_db_resource_stats` dynamic management view to monitor the resource consumption of individual databases. For more information, see [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database). One row exists for every 15 seconds, even if there is no activity. Historical data is maintained for approximately one hour.

This sample query to view the last 10 minutes of data should be run in the desired database. 

```sql
SELECT * FROM sys.dm_db_resource_stats AS rs
WHERE rs.end_time > DATEADD(mi, -10, SYSUTCDATETIME());
```

For longer retention time with less frequency, consider the following query on `sys.resource_stats`, run in the `master` database of the Azure SQL logical server. For more information, see [sys.resource_stats (Azure SQL Database)](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database). One row exists every five minutes, and historical data is maintained for two weeks.

```sql
SELECT * FROM sys.resource_stats
WHERE [database_name] = 'sample'
ORDER BY [start_time] desc;
```

### Monitoring memory utilization

This query calculates the `oom_per_second` metric for each resource pool for recent history, based on the number of snapshots available. This sample query helps identify the recent average number of failed memory allocations in the pool. This query can be run in any database in an elastic pool.

```sql
SELECT pool_id,
       name AS resource_pool_name,
       IIF(name LIKE 'SloSharedPool%' OR name LIKE 'UserPool%', 'user', 'system') AS resource_pool_type,
       SUM(CAST(delta_out_of_memory_count AS decimal))/(SUM(duration_ms)/1000.) AS oom_per_second
FROM sys.dm_resource_governor_resource_pools_history_ex
GROUP BY pool_id, name
ORDER BY pool_id;
```

### Monitoring `tempdb` log space utilization

This query returns the current value of the `tempdb_log_used_percent` metric, showing the relative utilization of the `tempdb` transaction log relative to its maximum allowed size. This query can be run in any database in an elastic pool.

```sql
SELECT (lsu.used_log_space_in_bytes / df.log_max_size_bytes) * 100 AS tempdb_log_space_used_percent
FROM tempdb.sys.dm_db_log_space_usage AS lsu
CROSS JOIN (
           SELECT SUM(CAST(max_size AS bigint)) * 8 * 1024. AS log_max_size_bytes
           FROM tempdb.sys.database_files
           WHERE type_desc = N'LOG'
           ) AS df
;
```

## Next steps

- For an introduction to elastic pools, see [Elastic pools help you manage and scale multiple databases in Azure SQL Database](./elastic-pool-overview.md).
- For information on tuning query workloads to reduce resource utilization, see [Monitoring and tuning](monitoring-tuning-index.yml), and [Monitoring and performance tuning](./monitor-tune-overview.md).
