---
title: "SQL Server to Azure SQL Database: Migration overview"
description: Learn about the tools and options available to migrate your SQL Server databases to Azure SQL Database.
author: rwestMSFT
ms.author: randolphwest
ms.reviewer: mathoma
ms.date: 06/26/2024
ms.service: azure-sql-database
ms.subservice: migration-guide
ms.topic: how-to
ms.collection:
  - sql-migration-content
---
# Migration overview: SQL Server to Azure SQL Database

[!INCLUDE [appliesto-sqldb](../../includes/appliesto-sqldb.md)]

Learn about the options and considerations for migrating your SQL Server databases to Azure SQL Database.

You can migrate existing SQL Server databases running on:

- SQL Server on Virtual Machines
- Amazon EC2 (Elastic Compute Cloud)
- Amazon RDS (Relational Database Service) for SQL Server
- Google Compute Engine
- Cloud SQL for SQL Server - GCP (Google Cloud Platform)

For other migration guides, see [Database Migration](/data-migration).

## Overview

[Azure SQL Database](/azure/azure-sql/database/sql-database-paas-overview) is a recommended target option for SQL Server workloads that require a fully managed platform as a service (PaaS). SQL Database handles most database management functions. It also has built-in high availability, intelligent query processing, scalability, and performance capabilities to suit many application types.

SQL Database provides flexibility with multiple [deployment models](/azure/azure-sql/database/sql-database-paas-overview#deployment-models) and [service tiers](/azure/azure-sql/database/service-tiers-vcore#service-tiers) that cater to different types of applications or workloads.

One of the key benefits of migrating to SQL Database is that you can modernize your application by using the PaaS capabilities. You can then eliminate any dependency on technical components that are scoped at the instance level, such as SQL Agent jobs.

You can also save costs by using the [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/) for SQL Server to migrate your SQL Server on-premises licenses to Azure SQL Database. This option is available if you choose the [vCore-based purchasing model](/azure/azure-sql/database/service-tiers-vcore).

Be sure to review the SQL Server database engine features [available in Azure SQL Database](/azure/azure-sql/database/features-comparison) to validate the supportability of your migration target.

## Considerations

The key factors to consider when you're evaluating migration options are:

- Number of servers and databases
- Size of databases
- Acceptable business downtime during the migration process

The migration options listed in this guide take these factors into account. For logical data migration to Azure SQL Database, the time to migrate can depend on both the number of objects in a database and the size of the database.

Tools are available for various workloads and user preferences. Some tools can be used to perform a quick migration of a single database through a UI-based tool. Other tools can automate the migration of multiple databases to handle migrations at scale.

## Choose an appropriate target

Consider general guidelines to help you choose the right deployment model and service tier of Azure SQL Database. You can choose compute and storage resources during deployment and then [change them afterward by using the Azure portal](/azure/azure-sql/database/scale-resources) without incurring downtime for your application.

**Deployment models**: Understand your application workload and the usage pattern to decide between a single database or an elastic pool.

- A [single database](/azure/azure-sql/database/single-database-overview) represents a fully managed database that's suitable for most modern cloud applications and microservices.
- An [elastic pool](/azure/azure-sql/database/elastic-pool-overview) is a collection of single databases with a shared set of resources, such as CPU or memory. It's suitable for combining databases in a pool with predictable usage patterns that can effectively share the same set of resources.

**Purchasing models**: Choose between the vCore, database transaction unit (DTU), or serverless purchasing models.

- The [vCore model](/azure/azure-sql/database/service-tiers-vcore) lets you choose the number of vCores for Azure SQL Database, so it's the easiest choice when you're translating from on-premises SQL Server. This is the only option that supports saving license costs with the [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/).
- The [DTU-based purchasing model](/azure/azure-sql/database/service-tiers-dtu) abstracts the underlying compute, memory, and I/O resources to provide a blended DTU.
- The [serverless model](/azure/azure-sql/database/serverless-tier-overview) is for workloads that require automatic on-demand scaling with compute resources billed per second of usage. The serverless compute tier automatically pauses databases during inactive periods (where only storage is billed). It automatically resumes databases when activity returns.

**Service tiers**: Choose between three service tiers designed for different types of applications.

- [General Purpose/Standard service tier](/azure/azure-sql/database/service-tier-general-purpose) offers a balanced budget-oriented option with compute and storage suitable to deliver applications in the middle and lower tiers. Redundancy is built in at the storage layer to recover from failures. It's designed for most database workloads.
- [Business Critical/Premium service tier](/azure/azure-sql/database/service-tier-business-critical) is for high-tier applications that require high transaction rates, low-latency I/O, and a high level of resiliency. Secondary replicas are available for failover and to offload read workloads.
- [Hyperscale service tier](/azure/azure-sql/database/service-tier-hyperscale) is intended for all customers who require higher performance and availability, fast backup and restore, and/or fast storage and compute scalability. This includes customers who are moving to the cloud to modernize their applications as well as customers who are already using other service tiers in Azure SQL Database. The Hyperscale service tier supports a broad range of database workloads, from pure OLTP to pure analytics. It's optimized for OLTP and hybrid transaction and analytical processing (HTAP) workloads.

> [!IMPORTANT]  
> [Transaction log rate is governed](/azure/azure-sql/database/resource-limits-logical-server#transaction-log-rate-governance) in Azure SQL Database to limit high ingestion rates. As such, during migration, you might have to scale target database resources (vCores or DTUs) to ease pressure on CPU or throughput. Choose the appropriately sized target database, but plan to scale resources up for the migration if necessary.

### SQL Server VM alternative

Your business might have requirements that make [SQL Server on Azure Virtual Machines](/azure/azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview) a more suitable target than Azure SQL Database.

If one of the following conditions applies to your business, consider moving to a SQL Server virtual machine (VM) instead:

- You require direct access to the operating system or file system, such as to install third-party or custom agents on the same virtual machine with SQL Server.
- You have strict dependency on features that are still not supported, such as FileStream/FileTable, PolyBase, and cross-instance transactions.
- You need to stay at a specific version of SQL Server (2012, for example).
- Your compute requirements are much lower than a managed instance offers (one vCore, for example), and database consolidation isn't an acceptable option.

## Migration tools

We recommend the following migration tools:

| Technology | Description |
| --- | --- |
| [Azure Migrate](/azure/migrate/how-to-create-azure-sql-assessment) | This Azure service helps you discover and assess your SQL data estate at scale on VMware. It provides Azure SQL deployment recommendations, target sizing, and monthly estimates. |
| [Azure SQL migration extension for Azure Data Studio](/azure/dms/migration-using-azure-data-studio) | Powered by the [Azure Database Migration Service](/azure/dms/dms-overview), the Azure SQL Migration extension for Azure Data Studio helps you to assess your database requirements to understand your migration readiness, get the right-sized SKU recommendations for Azure resources, and migrate your SQL Server database to Azure. You can migrate single databases or at scale using [PowerShell and Azure CLI](/azure/dms/migration-dms-powershell-cli). |

The following table lists alternative migration tools:

| Technology | Description |
| --- | --- |
| [Transactional replication](/azure/azure-sql/database/replication-to-sql-database) | Replicate data from source SQL Server database tables to Azure SQL Database by providing a publisher-subscriber type migration option while maintaining transactional consistency. Incremental data changes are propagated to subscribers as they occur on the publishers. |
| [Import Export Service/BACPAC](/azure/azure-sql/database/database-import) | [BACPAC](/sql/relational-databases/data-tier-applications/data-tier-applications#bacpac) is a Windows file with a .bacpac extension that encapsulates a database's schema and data. You can use BACPAC to both export data from a SQL Server source and import the data into Azure SQL Database. A BACPAC file can be imported to a new SQL database through the Azure portal.<br /><br />For scale and performance with large databases sizes or a large number of databases, consider using the [SqlPackage](/azure/azure-sql/database/database-import#use-sqlpackage) command-line tool to export and import databases. |
| [Bulk copy](/sql/relational-databases/import-export/import-and-export-bulk-data-by-using-the-bcp-utility-sql-server) | The [bulk copy program (bcp) tool](/sql/tools/bcp-utility) copies data from an instance of SQL Server into a data file. Use the tool to export the data from your source and import the data file into the target SQL database.<br /><br />For high-speed bulk copy operations to move data to Azure SQL Database, you can use the [Smart Bulk Copy tool](/samples/azure-samples/smartbulkcopy/smart-bulk-copy/) to maximize transfer speed by taking advantage of parallel copy tasks. |
| [Azure Data Factory](/azure/data-factory/connector-azure-sql-database) | The [Copy activity](/azure/data-factory/copy-activity-overview) in Azure Data Factory migrates data from source SQL Server databases to Azure SQL Database by using built-in connectors and an [integration runtime](/azure/data-factory/concepts-integration-runtime).<br /><br />Data Factory supports a wide range of [connectors](/azure/data-factory/connector-overview) to move data from SQL Server sources to Azure SQL Database. |
| [SQL Data Sync](/azure/azure-sql/database/sql-data-sync-data-sql-server-sql-database) | SQL Data Sync is a service built on Azure SQL Database that lets you synchronize selected data bidirectionally across multiple databases, both on-premises and in the cloud.<br />Data Sync is useful in cases where data needs to be kept updated across several databases in Azure SQL Database or SQL Server. |

## Compare migration options

Compare migration options to choose the path that's appropriate to your business needs.

The following table compares the migration options that we recommend:

| Migration option | When to use | Considerations |
| --- | --- | --- |
| [Azure Migrate](/azure/migrate/how-to-create-azure-sql-assessment) | - Discovery and assess single databases or at scale from different environments. | - All pre-migration steps such as discovery, assessments, and right-sizing of on-premises resources are included for infrastructure, data, and applications. |
| [Azure SQL migration extension for Azure Data Studio](/azure/dms/migration-using-azure-data-studio) | - Migrate single databases or at scale.<br />- Offline mode only.<br /><br />Supported sources:<br />- SQL Server (2008 onward) on-premises, or on Azure Virtual Machines<br />- SQL Server on Amazon EC2<br />- Amazon RDS for SQL Server<br />- SQL Server on Google Compute Engine | - Migrations at scale can be automated via [PowerShell or Azure CLI](/azure/dms/migration-dms-powershell-cli).<br />- Time to complete migration depends on database size and the number of objects in the database.<br />- Azure Data Studio is required when you aren't using PowerShell or Azure CLI. |

The following table compares the alternative migration options:

| Method or technology | When to use | Considerations |
| --- | --- | --- |
| [Transactional replication](/azure/azure-sql/database/replication-to-sql-database) | - Migrate by continuously publishing changes from source database tables to target SQL Database tables.<br />- Do full or partial database migrations of selected tables (subset of a database).<br /><br />Supported sources:<br />- [SQL Server (2016 to 2019) with some limitations](/sql/relational-databases/replication/replication-backward-compatibility)<br />- AWS EC2<br />- GCP Compute SQL Server VM | - Setup is relatively complex compared to other migration options.<br />- Provides a continuous replication option to migrate data (without taking the databases offline).<br />- Transactional replication has limitations to consider when you're setting up the publisher on the source SQL Server instance. See [Limitations on publishing objects](/sql/relational-databases/replication/publish/publish-data-and-database-objects#limitations-on-publishing-objects) to learn more.<br />- It's possible to [monitor replication activity](/sql/relational-databases/replication/monitor/monitoring-replication). |
| [Import Export Service/BACPAC](/azure/azure-sql/database/database-import) | - Migrate individual line-of-business application databases.<br />- Suited for smaller databases.<br />- Doesn't require a separate migration service or tool.<br /><br />Supported sources:<br />- SQL Server (2005 to 2019) on-premises or Azure VM<br />- AWS EC2<br />- AWS RDS<br />- GCP Compute SQL Server VM | - Requires downtime because data needs to be exported at the source and imported at the destination.<br />- The file formats and data types used in the export or import need to be consistent with table schemas to avoid truncation or data-type mismatch errors.<br />- Time taken to export a database with a large number of objects can be significantly higher. |
| [Bulk copy](/sql/relational-databases/import-export/import-and-export-bulk-data-by-using-the-bcp-utility-sql-server) | - Do full or partial data migrations.<br />- Can accommodate downtime.<br /><br />Supported sources:<br />- SQL Server (2005 to 2019) on-premises or Azure VM<br />- AWS EC2<br />- AWS RDS<br />- GCP Compute SQL Server VM | - Requires downtime for exporting data from the source and importing into the target.<br />- The file formats and data types used in the export or import need to be consistent with table schemas. |
| [Azure Data Factory](/azure/data-factory/connector-azure-sql-database) | - Migrate and/or transform data from source SQL Server databases.<br />- Merging data from multiple sources of data to Azure SQL Database is typically for business intelligence (BI) workloads. | - Requires creating data movement pipelines in Data Factory to move data from source to destination.<br />- [Cost](https://azure.microsoft.com/pricing/details/data-factory/data-pipeline/) is an important consideration and is based on factors like pipeline triggers, activity runs, and duration of data movement. |
| [SQL Data Sync](/azure/azure-sql/database/sql-data-sync-data-sql-server-sql-database) | - Synchronize data between source and target databases.<br />- Suitable to run continuous sync between Azure SQL Database and on-premises SQL Server in a bidirectional flow. | - Azure SQL Database must be the hub database for sync with an on-premises SQL Server database as a member database.<br />- Compared to transactional replication, SQL Data Sync supports bidirectional data sync between on-premises and Azure SQL Database.<br />- Can have a higher performance impact, depending on the workload. |

## Feature interoperability

There are more considerations when you're migrating workloads that rely on other SQL Server features.

### SQL Server Integration Services

Migrate SQL Server Integration Services (SSIS) packages to Azure by redeploying the packages to the Azure-SSIS runtime in [Azure Data Factory](/azure/data-factory/introduction). Azure Data Factory [supports migration of SSIS packages](/azure/data-factory/scenario-ssis-migration-overview#azure-sql-database-as-database-workload-destination) by providing a runtime built to run SSIS packages in Azure. Alternatively, you can rewrite the SSIS ETL (extract, transform, load) logic natively in Azure Data Factory by using [data flows](/azure/data-factory/concepts-data-flow-overview).

### SQL Server Reporting Services

Migrate SQL Server Reporting Services (SSRS) reports to paginated reports in Power BI. Use the [RDL Migration Tool](https://github.com/microsoft/RdlMigration) to help prepare and migrate your reports. Microsoft developed this tool to help customers migrate Report Definition Language (RDL) reports from their SSRS servers to Power BI. It's available on GitHub, and it documents an end-to-end walkthrough of the migration scenario.

### High availability

Manual setup of SQL Server high-availability features like Always On failover cluster instances and Always On availability groups becomes obsolete on the target SQL database. High-availability architecture is already built into both [General Purpose (standard availability model)](/azure/azure-sql/database/high-availability-sla-local-zone-redundancy#locally-redundant-availability) and [Business Critical (premium availability model)](/azure/azure-sql/database/high-availability-sla-local-zone-redundancy#locally-redundant-availability) service tiers for Azure SQL Database. The Business Critical/Premium service tier also provides read scale-out that allows connecting into one of the secondary nodes for read-only purposes.

Beyond the high-availability architecture that's included in Azure SQL Database, the [failover groups](/azure/azure-sql/database/failover-group-sql-db) feature allows you to manage the replication and failover of databases to a server in another region.

### Logins and groups

Windows logins aren't supported in Azure SQL Database, create a login from Microsoft Entra ID ([formerly Azure Active Directory](/entra/fundamentals/new-name)) instead. Manually recreate any SQL logins.

### SQL Agent jobs

SQL Agent jobs aren't directly supported in Azure SQL Database. Instead, use [elastic jobs](/azure/azure-sql/database/job-automation-overview).

### System databases

For Azure SQL Database, the only applicable system databases are [master](/sql/relational-databases/databases/master-database) and `tempdb`. To learn more, see [tempdb in Azure SQL Database](/sql/relational-databases/databases/tempdb-database#tempdb-database-in-sql-database).

## Advanced features

Be sure to take advantage of the advanced cloud-based features in SQL Database. For example, you don't need to worry about managing backups because the service does it for you. You can restore to any [point in time within the retention period](/azure/azure-sql/database/recovery-using-backups#point-in-time-restore).

To strengthen security, consider using [Microsoft Entra authentication](/azure/azure-sql/database/authentication-aad-overview), [auditing](/azure/azure-sql/database/auditing-overview), [threat detection](/azure/azure-sql/database/azure-defender-for-sql), [row-level security](/sql/relational-databases/security/row-level-security), and [dynamic data masking](/sql/relational-databases/security/dynamic-data-masking).

In addition to advanced management and security features, SQL Database provides tools that can help you [monitor and tune your workload](/azure/azure-sql/database/monitor-tune-overview). [Azure SQL Analytics (Preview)](/azure/azure-monitor/insights/azure-sql) is an advanced solution for monitoring the performance of all of your databases in Azure SQL Database at scale and across multiple subscriptions in a single view. Azure SQL Analytics collects and visualizes key performance metrics with built-in intelligence for performance troubleshooting.

[Automatic tuning](/sql/relational-databases/automatic-tuning/automatic-tuning#automatic-plan-correction) continuously monitors performance of your SQL execution plan and automatically fixes identified performance issues.

## Migration assets

For more assistance, see the following resources that were developed for real-world migration projects.

| Asset | Description |
| --- | --- |
| [Data workload assessment model and tool](https://www.microsoft.com/download/details.aspx?id=103130) | This tool provides suggested "best fit" target platforms, cloud readiness, and an application/database remediation level for a workload. It offers simple, one-click calculation and report generation that helps to accelerate large estate assessments by providing an automated and uniform decision process for target platforms. |
| [Bulk database creation with PowerShell](https://www.microsoft.com/download/details.aspx?id=103107) | You can use a set of three PowerShell scripts that create a resource group (create_rg.ps1), the [logical server in Azure](/azure/azure-sql/database/logical-servers) (create_sqlserver.ps1), and a SQL database (create_sqldb.ps1). The scripts include loop capabilities so you can iterate and create as many servers and databases as necessary. |
| [Bulk schema deployment with MSSQL-Scripter and PowerShell](https://www.microsoft.com/download/details.aspx?id=103032) | This asset creates a resource group, creates one or multiple [logical servers in Azure](/azure/azure-sql/database/logical-servers) to host Azure SQL Database, exports every schema from an on-premises SQL Server instance (or multiple SQL Server 2005+ instances), and imports the schemas to Azure SQL Database. |
| [Convert SQL Server Agent jobs into elastic database jobs](https://www.microsoft.com/download/details.aspx?id=103123) | This script migrates your source SQL Server Agent jobs to elastic database jobs. |
| [Utility to move on-premises SQL Server logins to Azure SQL Database](https://www.microsoft.com/download/details.aspx?id=103111) | A PowerShell script can create a T-SQL command script to re-create logins and select database users from on-premises SQL Server to Azure SQL Database. The tool allows automatic mapping of Windows Server Active Directory accounts to Microsoft Entra accounts, along with optionally migrating SQL Server native logins. |
| [Perfmon data collection automation by using Logman](https://www.microsoft.com/download/details.aspx?id=103114) | You can use the Logman tool to collect Perfmon data (to help you understand baseline performance) and get migration target recommendations. This tool uses logman.exe to create the command that will create, start, stop, and delete performance counters set on a remote SQL Server instance. |

The Data SQL Engineering team developed these resources. This team's core charter is to unblock and accelerate complex modernization for data platform migration projects to Microsoft's Azure data platform.

## Related content

- [Migration guide: SQL Server to Azure SQL Database](guide.md)
- [Services and tools available for data migration scenarios](/azure/dms/dms-tools-matrix)
- [What is Azure SQL Database?](/azure/azure-sql/database/sql-database-paas-overview)
- [Azure Total Cost of Ownership Calculator](https://azure.microsoft.com/pricing/tco/calculator/)
- [Cloud Adoption Framework for Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale)
- [Best practices for costing and sizing workloads migrated to Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs)
- [Data Access Migration Toolkit (Preview)](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit)
- [Overview of Database Experimentation Assistant](/sql/dea/database-experimentation-assistant-overview)
