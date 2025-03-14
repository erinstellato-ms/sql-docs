---
title: "Grant Permissions on a Stored Procedure"
description: Learn how to grant permissions on a stored procedure in SQL Server by using SQL Server Management Studio or Transact-SQL.
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: vanto
ms.date: 09/12/2024
ms.service: sql
ms.subservice: stored-procedures
ms.topic: conceptual
ms.custom:
  - "UpdateFrequency5"
helpviewer_keywords:
  - "stored procedures [SQL Server], permissions"
monikerRange: ">=aps-pdw-2016 || =azuresqldb-current || =azure-sqldw-latest || >=sql-server-2016 || >=sql-server-linux-2017 || =azuresqldb-mi-current"
---
# Grant Permissions on a Stored Procedure

[!INCLUDE [SQL Server Azure SQL Database Synapse Analytics PDW](../../includes/applies-to-version/sql-asdb-asdbmi-asa-pdw.md)]

This article describes how to grant permissions on a stored procedure in [!INCLUDE [ssnoversion](../../includes/ssnoversion-md.md)] by using [!INCLUDE [ssManStudioFull](../../includes/ssmanstudiofull-md.md)] or [!INCLUDE [tsql](../../includes/tsql-md.md)]. Permissions can be granted to an existing user, database role, or application role in the database.

## <a name="Restrictions"></a> Limitations and restrictions

- You can't use [!INCLUDE [ssManStudioFull](../../includes/ssmanstudiofull-md.md)] to grant permissions on system procedures or system functions. Use [GRANT object permissions (Transact-SQL)](../../t-sql/statements/grant-object-permissions-transact-sql.md) instead.

## <a name="Security"></a><a name="Permissions"></a> Permissions

The grantor (or the principal specified with the `AS` option) must have either the permission itself with `GRANT OPTION`, or a higher permission that implies the permission being granted. Requires `ALTER` permission on the schema to which the procedure belongs, or `CONTROL` permission on the procedure. For more information, see [GRANT object permissions](../../t-sql/statements/grant-object-permissions-transact-sql.md).

## <a name="SSMSProcedure"></a> Use SQL Server Management Studio

#### To grant permissions on a stored procedure

1. In Object Explorer, connect to an instance of [!INCLUDE [ssDE](../../includes/ssde-md.md)] and then expand that instance.

1. Expand **Databases**, expand the database in which the procedure belongs, and then expand **Programmability**.

1. Expand **Stored Procedures**, right-click the procedure to grant permissions on, and then select **Properties**.

1. From **Stored Procedure Properties**, select the **Permissions** page.

1. To grant permissions to a user, database role, or application role, select **Search**.

1. In **Select Users or Roles**, select **Object Types** to add or clear the users and roles you want.

1. Select **Browse** to display the list of users or roles. Select the users or roles to whom permissions should be granted.

1. In the **Explicit Permissions** grid, select the permissions to grant to the specified user or role. For a description of the permissions, see [Permissions (Database Engine)](../security/permissions-database-engine.md).

Selecting **Grant** indicates the grantee will be given the specified permission. Selecting **Grant With** indicates that the grantee will also be able to grant the specified permission to other principals.

## <a name="TsqlProcedure"></a> Use Transact-SQL

### To grant permissions on a stored procedure

1. Connect to the [!INCLUDE [ssDE](../../includes/ssde-md.md)].

1. From the Standard bar, select **New Query**.

1. Copy and paste the following example into the query window and select **Execute**. This example grants `EXECUTE` permission on the stored procedure `HumanResources.uspUpdateEmployeeHireInfo` to an application role named `Recruiting11`.

```sql
USE AdventureWorks2022;
GRANT EXECUTE ON OBJECT::HumanResources.uspUpdateEmployeeHireInfo
    TO Recruiting11;
GO
```

### To grant permissions on all stored procedures in a schema

1. Connect to the [!INCLUDE [ssDE](../../includes/ssde-md.md)].

1. From the Standard bar, select **New Query**.

1. Copy and paste the following example into the query window and select **Execute**. This example grants `EXECUTE` permission to all stored procedures that exist, or will exist, in the `HumanResources` schema, to an application role named `Recruiting11`.

```sql
USE AdventureWorks2022;
GRANT EXECUTE ON SCHEMA::HumanResources
    TO Recruiting11;
GO
```

## Related content

- [sys.fn_builtin_permissions (Transact-SQL)](../system-functions/sys-fn-builtin-permissions-transact-sql.md)
- [GRANT object permissions (Transact-SQL)](../../t-sql/statements/grant-object-permissions-transact-sql.md)
- [Create a stored procedure](create-a-stored-procedure.md)
- [Modify a stored procedure](modify-a-stored-procedure.md)
- [Delete a stored procedure](delete-a-stored-procedure.md)
- [Rename a Stored Procedure](rename-a-stored-procedure.md)
