---
title: Building scalable cloud databases
description: Build scalable .NET database apps with the Elastic Database client library.
author: scoriani
ms.author: scoriani
ms.reviewer: wiassaf, mathoma
ms.date: 09/25/2018
ms.service: azure-sql-database
ms.subservice: scale-out
ms.topic: conceptual
ms.custom: sqldbrb=1
---
# Building scalable cloud databases
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Scaling out databases can be easily accomplished using scalable tools and features for Azure SQL Database. In particular, you can use the **Elastic Database client library** to create and manage scaled-out databases. This feature lets you easily develop sharded applications using hundreds—or even thousands—of databases in Azure SQL Database.

To download:

* The Java version of the library, see [Maven Central Repository](https://search.maven.org/#search%7Cga%7C1%7Celastic-db-tools).
* The .NET version of the library, see [NuGet](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/).

## Documentation

1. [Get started with Elastic Database tools](elastic-scale-get-started.md)
2. [Elastic Database features](elastic-scale-introduction.md)
3. [Shard map management](elastic-scale-shard-map-management.md)
4. [Migrate existing databases to scale out](elastic-convert-to-use-elastic-tools.md)
5. [Data dependent routing](elastic-scale-data-dependent-routing.md)
6. [Multi-shard queries](elastic-scale-multishard-querying.md)
7. [Adding a shard using Elastic Database tools](elastic-scale-add-a-shard.md)
8. [Multi-tenant applications with Elastic Database tools and row-level security](saas-tenancy-elastic-tools-multi-tenant-row-level-security.md)
9. [Upgrade client library apps](elastic-scale-upgrade-client-library.md) 
10. [Elastic queries overview](elastic-query-overview.md)
11. [Elastic Database tools glossary](elastic-scale-glossary.md)
12. [Elastic Database client library with Entity Framework](elastic-scale-use-entity-framework-applications-visual-studio.md)
13. [Elastic Database client library with Dapper](elastic-scale-working-with-dapper.md)
14. [Split-merge tool](elastic-scale-overview-split-and-merge.md)
15. [Performance counters for shard map manager](elastic-database-client-library.md) 
16. [FAQ for Elastic Database tools](elastic-scale-faq.yml)

## Client capabilities

Scaling out applications using *sharding* presents challenges for both the developer as well as the administrator. The client library simplifies the management tasks by providing tools that let both developers and administrators manage scaled-out databases. In a typical example, there are many databases, known as "shards," to manage. Customers are co-located in the same database, and there is one database per customer (a single-tenant scheme). The client library includes these features:

- **Shard map management**: A special database called the "shard map manager" is created. Shard map management is the ability for an application to manage metadata about its shards. Developers can use this functionality to register databases as shards, describe mappings of individual sharding keys or key ranges to those databases, and maintain this metadata as the number and composition of databases evolves to reflect capacity changes. Without the Elastic Database client library, you would need to spend a lot of time writing the management code when implementing sharding. For details, see [Shard map management](elastic-scale-shard-map-management.md).

- **Data dependent routing**: Imagine a request coming into the application. Based on the sharding key value of the request, the application needs to determine the correct database based on the key value. It then opens a connection to the database to process the request. Data dependent routing provides the ability to open connections with a single easy call into the shard map of the application. Data dependent routing was another area of infrastructure code that is now covered by functionality in the Elastic Database client library. For details, see [Data dependent routing](elastic-scale-data-dependent-routing.md).
- **Multi-shard queries (MSQ)**: Multi-shard querying works when a request involves several (or all) shards. A multi-shard query executes the same T-SQL code on all shards or a set of shards. The results from the participating shards are merged into an overall result set using UNION ALL semantics. The functionality as exposed through the client library handles many tasks, including: connection management, thread management, fault handling, and intermediate results processing. MSQ can query up to hundreds of shards. For details, see [Multi-shard querying](elastic-scale-multishard-querying.md).

In general, customers using Elastic Database tools can expect to get full T-SQL functionality when submitting shard-local operations as opposed to cross-shard operations that have their own semantics.



## Next steps

- Elastic Database client library ([Java](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-elasticdb-tools%22), [.NET](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/)) - to **download** the library.

- [Get started with Elastic Database tools](elastic-scale-get-started.md) - to try the **sample app** that demonstrates client functions.

- GitHub ([Java](https://github.com/Microsoft/elastic-db-tools-for-java/blob/master/README.md), [.NET](https://github.com/Azure/elastic-db-tools)) - to make contributions to the code.
- [Azure SQL Database elastic query overview](elastic-query-overview.md) - to use elastic queries.

- [Moving data between scaled-out cloud databases](elastic-scale-overview-split-and-merge.md) - for instructions on using the **split-merge tool**.



<!-- Additional resources H2 -->

[!INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]


<!--Anchors-->
<!--Image references-->

[1]: ./media/sql-database-elastic-database-client-library/glossary.png

