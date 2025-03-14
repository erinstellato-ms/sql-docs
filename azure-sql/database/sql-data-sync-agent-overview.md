---
title: Data Sync Agent for SQL Data Sync
description: Learn how to install and run the Data Sync Agent for SQL Data Sync in Azure to sync data with SQL Server databases
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: mathoma, hudequei
ms.date: 09/23/2024
ms.service: azure-sql-database
ms.subservice: sql-data-sync
ms.topic: conceptual
ms.custom: sqldbrb=1
---
# Data Sync Agent for SQL Data Sync
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Sync data with SQL Server databases by installing and configuring the Data Sync Agent for SQL Data Sync in Azure. For more info about SQL Data Sync, see [Sync data across multiple cloud and on-premises databases with SQL Data Sync](sql-data-sync-data-sql-server-sql-database.md).

> [!IMPORTANT]
> SQL Data Sync does **not** support Azure SQL Managed Instance or Azure Synapse Analytics at this time.

## Download and install

To download the Data Sync Agent, go to [SQL Data Sync Agent](https://www.microsoft.com/download/details.aspx?id=27693). To upgrade the Data Sync Agent, install the Agent in the same location as the old Agent and it will override the original one.

### Install silently

To install the Data Sync Agent silently from the command prompt, enter a command similar to the following example. Check the file name of the downloaded .msi file, and provide your own values for the **TARGETDIR** and **SERVICEACCOUNT** arguments.

- If you don't provide a value for **TARGETDIR**, the default value is `C:\Program Files (x86)\Microsoft SQL Data Sync 2.0`.

- If you provide `LocalSystem` as the value of **SERVICEACCOUNT**, use SQL Server authentication when you configure the agent to connect to SQL Server.

- If you provide a domain user account or a local user account as the value of **SERVICEACCOUNT**, you also have to provide the password with the **SERVICEPASSWORD** argument. For example, `SERVICEACCOUNT="<domain>\<user>"  SERVICEPASSWORD="<password>"`.

```cmd
msiexec /i "SQLDataSyncAgent-2.0-x86-ENU.msi" TARGETDIR="C:\Program Files (x86)\Microsoft SQL Data Sync 2.0" SERVICEACCOUNT="LocalSystem" /qn
```

## Sync data with a SQL Server database

To configure the Data Sync Agent so you can sync data with one or more SQL Server databases, see [Add a SQL Server database](sql-data-sync-sql-server-configure.md#add-on-prem).

## <a name="agent-faq"></a> Data Sync Agent FAQ

### Why do I need a client agent

The SQL Data Sync service communicates with SQL Server databases via the client agent. This security feature prevents direct communication with databases behind a firewall. When the SQL Data Sync service communicates with the agent, it does so using encrypted connections and a unique token or *agent key*. The SQL Server databases authenticate the agent using the connection string and agent key. This design provides a high level of security for your data.

### How many instances of the local agent UI can be run

Only one instance of the UI can be run.

### How can I change my service account

After you install a client agent, the only way to change the service account is to uninstall it and install a new client agent with the new service account.

### How do I change my agent key

An agent key can only be used once by an agent. It cannot be reused when you remove then reinstall a new agent, nor can it be used by multiple agents. If you need to create a new key for an existing agent, you must be sure that the same key is recorded with the client agent and with the SQL Data Sync service.

### How do I retire a client agent

To immediately invalidate or retire an agent, regenerate its key in the portal but do not submit it in the Agent UI. Regenerating a key invalidates the previous key irrespective if the corresponding agent is online or offline.

### How do I move a client agent to another computer

If you want to run the local agent from a different computer than it is currently on, and reuse the same agent, do the following steps:

1. Install the agent on desired computer.
2. Sign in to the SQL Data Sync portal and regenerate an agent key for the existing agent.
3. Use the new agent's UI to submit the agent key.
4. Wait while the client agent downloads the list of on-premises databases that were registered earlier.
5. Provide database credentials for all databases that display as unreachable. These databases must be reachable from the new computer on which the agent is installed.

### How do I delete the Sync metadata database if the Sync agent is still associated with it

In order to delete a Sync metadata database that has a Sync agent associated with it, you must first delete the Sync agent. To delete the agent, do the following things: 

1. Select the Sync database. 
2. Go to the **Sync to other databases** page.
3. Select the Sync agent and select **Delete**. 

## <a name="agent-tshoot"></a> Troubleshoot Data Sync Agent issues

- [The client agent install, uninstall, or repair fails](#agent-install)

- [The client agent doesn't work after I cancel the uninstall](#agent-uninstall)

- [My database isn't listed in the agent list](#agent-list)

- [Client agent doesn't start (Error 1069)](#agent-start)

- [I can't submit the agent key](#agent-key)

- [The client agent can't be deleted from the portal if its associated on-premises database is unreachable](#agent-delete)

- [Local Sync Agent app can't connect to the local sync service](#agent-connect)

### <a name="agent-install"></a> The client agent install, uninstall, or repair fails

- **Cause**. Many scenarios might cause this failure. To determine the specific cause for this failure, look at the logs.

- **Resolution**. To find the specific cause of the failure, generate and look at the Windows Installer logs. You can turn on logging at a command prompt. For example, if the downloaded installation file is `SQLDataSyncAgent-2.0-x86-ENU.msi`, generate and examine log files by using the following command lines:

  - For installs: `msiexec.exe /i SQLDataSyncAgent-2.0-x86-ENU.msi /l*v LocalAgentSetup.Log`
  - For uninstalls: `msiexec.exe /x SQLDataSyncAgent-2.0-x86-ENU.msi /l*v LocalAgentSetup.Log`

    You can also turn on logging for all installations that are performed by Windows Installer. The Microsoft Knowledge Base article [How to enable Windows Installer logging](https://support.microsoft.com/help/223300/how-to-enable-windows-installer-logging) provides a one-click solution to turn on logging for Windows Installer. It also provides the location of the logs.

### <a name="agent-uninstall"></a> The client agent doesn't work after I cancel the uninstall

The client agent doesn't work, even after you cancel its uninstallation.

- **Cause**. This occurs because the SQL Data Sync client agent doesn't store credentials.

- **Resolution**. You can try these two solutions:

    -   Use services.msc to reenter the credentials for the client agent.
    -   Uninstall this client agent and then install a new one. Download and install the latest client agent from [Download Center](https://www.microsoft.com/download/details.aspx?id=27693).

### <a name="agent-list"></a> My database isn't listed in the agent list

When you attempt to add an existing SQL Server database to a sync group, the database doesn't appear in the list of agents.

These scenarios might cause this issue:

- **Cause**. The client agent and sync group are in different datacenters.

- **Resolution**. The client agent and the sync group must be in the same datacenter. To set this up, you have two options:

    -   Create a new agent in the datacenter where the sync group is located. Then, register the database with that agent.
    -   Delete the current sync group. Then, re-create the sync group in the datacenter where the agent is located.

- **Cause**. The client agent's list of databases isn't current.

- **Resolution**. Stop and then restart the client agent service.

    The local agent downloads the list of associated databases only on the first submission of the agent key. It doesn't download the list of associated databases on subsequent agent key submissions. Databases that are registered during an agent move don't show up in the original agent instance.

### <a name="agent-start"></a> Client agent doesn't start (Error 1069)

You discover that the agent isn't running on a computer that hosts SQL Server. When you attempt to manually start the agent, you see a dialog box that displays the message, "Error 1069: The service did not start due to a logon failure."

![Data Sync error 1069 dialog box](./media/sql-data-sync-agent-overview/sync-error-1069.png)

- **Cause**. A likely cause of this error is that the password on the local server has changed since you created the agent and agent password.

- **Resolution**. Update the agent's password to your current server password:

  1. Locate the SQL Data Sync client agent service.  
    a. Select **Start**.  
    b. In the search box, enter **services.msc**.  
    c. In the search results, select **Services**.  
    d. In the **Services** window, scroll to the entry for **SQL Data Sync Agent**.  
  1. Right-click **SQL Data Sync Agent**, and then select **Stop**.
  1. Right-click **SQL Data Sync Agent**, and then select **Properties**.
  1. On **SQL Data Sync Agent Properties**, select the **Log in** tab.
  1. In the **Password** box, enter your password.
  1. In the **Confirm Password** box, reenter your password.
  1. Select **Apply**, and then select **OK**.
  1. In the **Services** window, right-click the **SQL Data Sync Agent** service, and then select **Start**.
  1. Close the **Services** window.

### <a name="agent-key"></a> I can't submit the agent key

After you create or re-create a key for an agent, you try to submit the key through the SqlAzureDataSyncAgent application. The submission fails to complete.

![Sync Error dialog box - Can't submit agent key](./media/sql-data-sync-agent-overview/sync-error-cant-submit-agent-key.png)

- **Prerequisites**. Before you proceed, check the following prerequisites:

  - The SQL Data Sync Windows service is running.

  - The service account for SQL Data Sync Windows service has network access.

  - The outbound 1433 port is open in your local firewall rule.

  - The local ip is added to the server or database firewall rule for the sync metadata database.

- **Cause**. The agent key uniquely identifies each local agent. The key must meet two conditions:

  -   The client agent key on the SQL Data Sync server and the local computer must be identical.
  -   The client agent key can be used only once.

- **Resolution**. If your agent isn't working, it's because one or both of these conditions are not met. To get your agent to work again:

  1. Generate a new key.
  1. Apply the new key to the agent.

  To apply the new key to the agent:

  1. In File Explorer, go to your agent installation directory. The default installation directory is C:\\Program Files (x86)\\Microsoft SQL Data Sync.
  1. Double-click the bin subdirectory.
  1. Open the SqlAzureDataSyncAgent application.
  1. Select **Submit Agent Key**.
  1. In the space provided, paste the key from your clipboard.
  1. Select **OK**.
  1. Close the program.

### <a name="agent-delete"></a> The client agent can't be deleted from the portal if its associated on-premises database is unreachable

If a local endpoint (that is, a database) that is registered with a SQL Data Sync client agent becomes unreachable, the client agent can't be deleted.

- **Cause**. The local agent can't be deleted because the unreachable database is still registered with the agent. When you try to delete the agent, the deletion process tries to reach the database, which fails.

- **Resolution**. Use "force delete" to delete the unreachable database.

> [!NOTE]
> If sync metadata tables remain after a "force delete", use `deprovisioningutil.exe` to clean them up.

### <a name="agent-connect"></a> Local Sync Agent app can't connect to the local sync service

- **Resolution**. Try the following steps:

  1. Exit the app.  
  1. Open the Component Services Panel.  
    a. In the search box on the taskbar, enter **services.msc**.  
    b. In the search results, double-click **Services**.  
  1. Stop the **SQL Data Sync** service.
  1. Restart the **SQL Data Sync** service.  
  1. Reopen the app.

## Run the Data Sync Agent from the command prompt

You can run the following Data Sync Agent commands from the command prompt:

### Ping the service

#### Usage

```cmd
SqlDataSyncAgentCommand.exe -action pingsyncservice
```

#### Example

```cmd
SqlDataSyncAgentCommand.exe -action "pingsyncservice"
```

### Display registered databases

#### Usage

```cmd
SqlDataSyncAgentCommand.exe -action displayregistereddatabases
```

#### Example

```cmd
SqlDataSyncAgentCommand.exe -action "displayregistereddatabases"
```

### Submit the agent key

#### Usage

```cmd
Usage: SqlDataSyncAgentCommand.exe -action submitagentkey -agentkey [agent key]  -username [user name] -password [password]
```

#### Example

```cmd
SqlDataSyncAgentCommand.exe -action submitagentkey -agentkey [agent key generated from portal, PowerShell, or API] -username [user name to sync metadata database] -password [user name to sync metadata database]
```

### Register a database

#### Usage

```cmd
SqlDataSyncAgentCommand.exe -action registerdatabase -servername [on-premisesdatabase server name] -databasename [on-premisesdatabase name]  -username [domain\\username] -password [password] -authentication [sql or windows] -encryption [true or false]
```

#### Examples

```cmd
SqlDataSyncAgentCommand.exe -action "registerdatabase" -serverName localhost -databaseName testdb -authentication sql -username <user name> -password <password> -encryption true

SqlDataSyncAgentCommand.exe -action "registerdatabase" -serverName localhost -databaseName testdb -authentication windows -encryption true

```

### Unregister a database

When you use this command to unregister a database, it deprovisions the database completely. If the database participates in other sync groups, this operation breaks the other sync groups.

#### Usage

```cmd
SqlDataSyncAgentCommand.exe -action unregisterdatabase -servername [on-premisesdatabase server name] -databasename [on-premisesdatabase name]
```

#### Example

```cmd
SqlDataSyncAgentCommand.exe -action "unregisterdatabase" -serverName localhost -databaseName testdb
```

### Update credentials

#### Usage

```cmd
SqlDataSyncAgentCommand.exe -action updatecredential -servername [on-premisesdatabase server name] -databasename [on-premisesdatabase name]  -username [domain\\username] -password [password] -authentication [sql or windows] -encryption [true or false]
```

#### Examples

```cmd
SqlDataSyncAgentCommand.exe -action "updatecredential" -serverName localhost -databaseName testdb -authentication sql -username <user name> -password <password> -encryption true

SqlDataSyncAgentCommand.exe -action "updatecredential" -serverName localhost -databaseName testdb -authentication windows -encryption true
```

## Next steps

For more info about SQL Data Sync, see the following articles:

-   Overview - [Sync data across multiple cloud and on-premises databases with SQL Data Sync in Azure](sql-data-sync-data-sql-server-sql-database.md)
-   Set up Data Sync
    - In the portal - [Tutorial: Set up SQL Data Sync to sync data between Azure SQL Database and SQL Server](sql-data-sync-sql-server-configure.md)
    - With PowerShell
        -  [Use PowerShell to sync between multiple databases in Azure SQL Database](scripts/sql-data-sync-sync-data-between-sql-databases.md)
        -  [Use PowerShell to sync between a database in Azure SQL Database and a database in a SQL Server instance](scripts/sql-data-sync-sync-data-between-azure-onprem.md)
-   Best practices - [Best practices for Azure SQL Data Sync](sql-data-sync-best-practices.md)
-   Monitor - [Monitor SQL Data Sync with Azure Monitor logs](./monitor-tune-overview.md)
-   Troubleshoot - [Troubleshoot issues with Azure SQL Data Sync](sql-data-sync-troubleshoot.md)
-   Update the sync schema
    -   With Transact-SQL - [Automate replication of schema changes with SQL Data Sync in Azure](sql-data-sync-update-sync-schema.md)
    -   With PowerShell - [Use PowerShell to update the sync schema in an existing sync group](scripts/update-sync-schema-in-sync-group.md)
