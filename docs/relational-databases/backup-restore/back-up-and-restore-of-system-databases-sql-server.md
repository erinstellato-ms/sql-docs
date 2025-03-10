---
title: "Back up and restore: System databases"
description: SQL Server maintains system databases essential for operation of a server instance. Several system databases must be backed up after every significant update.
author: MashaMSFT
ms.author: mathoma
ms.reviewer: randolphwest
ms.date: 11/21/2023
ms.service: sql
ms.subservice: backup-restore
ms.topic: conceptual
helpviewer_keywords:
  - "system databases [SQL Server], backing up and restoring"
  - "restoring system databases [SQL Server]"
  - "backing up [SQL Server], system databases"
  - "database backups [SQL Server], system databases"
  - "servers [SQL Server], backup"
---
# Back up and restore: System databases (SQL Server)

[!INCLUDE [SQL Server](../../includes/applies-to-version/sqlserver.md)]

[!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] maintains a set of system-level databases, called *system databases*, which are essential for the operation of a server instance. Several of the system databases must be backed up after every significant update. The system databases that you must always back up include `msdb`, `master`, and `model`. If any database uses replication on the server instance, you must also back up the `distribution` system database. Backups of these system databases let you restore and recover the [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] system after a system failure, such as the loss of a storage device.

The following table summarizes all of the system databases.

| System database | Description | Backups required? | Recovery model | Comments |
| --- | --- | --- | --- | --- |
| [master](../databases/master-database.md) | The database that records all of the system level information for a [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] system. | Yes | Simple | Back up `master` as often as necessary to protect the data sufficiently for your business needs. We recommend a regular backup schedule, which you can supplement with an additional backup after a substantial update. Transaction log backups of `master` aren't supported. |
| [model](../databases/model-database.md) | The template for all databases that are created on the instance of [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)]. | Yes | User configurable<sup>1</sup> | Back up `model` only when necessary for your business needs; for example, immediately after customizing its database options.<br /><br />**Best practice:** We recommend that you create only full database backups of `model`, as required. Because `model` is small and rarely changes, backing up the log is unnecessary. |
| [msdb](../databases/msdb-database.md) | The database used by [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] Agent for scheduling alerts and jobs, and for recording operators. `msdb` also contains history tables such as the backup and restore history tables. | Yes | Simple (default) | Back up `msdb` whenever it's updated. |
| [Resource Database](../databases/resource-database.md) (RDB) | A read-only database that contains copies of all system objects that ship with [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] | No | None | The Resource database resides in the `mssqlsystemresource.mdf` file, which contains only code. Therefore, [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] can't back up the Resource database.<br /><br />**Note:** You can perform a file-based or a disk-based backup on the `mssqlsystemresource.mdf` file by treating the file as if it were a binary (`.exe`) file, instead of a database file. But you can't use [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] restore on the backups. Restoring a backup copy of `mssqlsystemresource.mdf` can only be done manually, and you must be careful not to overwrite the current Resource database with an out-of-date or potentially insecure version. |
| [tempdb](../databases/tempdb-database.md) | A workspace for holding temporary or intermediate result sets. This database is re-created every time an instance of [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] is started. When the server instance is shut down, any data in `tempdb` is deleted permanently. | No | Simple | You can't back up the `tempdb` system database. |
| [Configure Distribution](../replication/configure-distribution.md) | A database that exists only if the server is configured as a replication Distributor. This database stores metadata and history data for all types of replication, and transactions for transactional replication. | Yes | Simple | For information about when to back up the `distribution` database, see [Back Up and Restore Replicated Databases](../replication/administration/back-up-and-restore-replicated-databases.md). |

<sup>1</sup> To learn the current recovery model of the model, see [View or change the recovery model of a database (SQL Server)](view-or-change-the-recovery-model-of-a-database-sql-server.md) or [sys.databases (Transact-SQL)](../system-catalog-views/sys-databases-transact-sql.md).

## Limitations on restoring system databases

System databases can be restored only from backups that are created on the version of [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] that the server instance is currently running. For example, to restore a system database on a server instance that is running [!INCLUDE [sssql16-md](../../includes/sssql16-md.md)] with Service Pack 1, you must use a database backup that was created after the server instance was upgraded to [!INCLUDE [sssql16-md](../../includes/sssql16-md.md)] SP 1.

To restore any database, the instance of [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] must be running. Startup of an instance of [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] requires that the `master` database is accessible and at least partly usable. If `master` becomes unusable, you can return the database to a usable state in either of the following ways:

- Restore `master` from a current database backup.  
  
  If you can start the server instance, you should be able to restore `master` from a full database backup. For more information, see [Restore the master database (Transact-SQL)](restore-the-master-database-transact-sql.md).

- Rebuild `master` completely.

  If severe damage to `master` prevents you from starting [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)], you must rebuild `master`. For more information, see [Rebuild system databases](../databases/rebuild-system-databases.md).

  > [!IMPORTANT]  
  >  Rebuilding `master` rebuilds all of the system databases.

Under some circumstances, problems recovering the `model` database might require rebuilding the system databases or replacing the `mdf` and `ldf` files for the `model` database. For more information, see [Rebuild system databases](../databases/rebuild-system-databases.md).

## <a id="RelatedTasks"></a> Related tasks

- [Create a Full Database Backup](create-a-full-database-backup-sql-server.md)
- [Complete Database Restores (Simple Recovery Model)](complete-database-restores-simple-recovery-model.md)
- [Restore the master Database (Transact-SQL)](../../relational-databases/backup-restore/restore-the-master-database-transact-sql.md)
- [View or change the recovery model of a database (SQL Server)](view-or-change-the-recovery-model-of-a-database-sql-server.md)
- [Move system databases](../databases/move-system-databases.md)

## Related content

- [Distribution Database](../replication/distribution-database.md)
- [master Database](../databases/master-database.md)
- [msdb Database](../databases/msdb-database.md)
- [model Database](../databases/model-database.md)
- [Resource Database](../databases/resource-database.md)
- [tempdb Database](../databases/tempdb-database.md)
