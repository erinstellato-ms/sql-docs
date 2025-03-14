---
title: "usesLocalFilePerTable Method (SQLServerDatabaseMetaData)"
description: "usesLocalFilePerTable Method (SQLServerDatabaseMetaData)"
author: David-Engel
ms.author: davidengel
ms.date: "01/19/2017"
ms.service: sql
ms.subservice: connectivity
ms.topic: reference
apilocation: "sqljdbc.jar"
apiname: "SQLServerDatabaseMetaData.usesLocalFilePerTable"
apitype: "Assembly"
---
# usesLocalFilePerTable Method (SQLServerDatabaseMetaData)
[!INCLUDE[Driver_JDBC_Download](../../../includes/driver_jdbc_download.md)]

  Retrieves whether this database uses a file for each table.  
  
## Syntax  
  
```  
  
public boolean usesLocalFilePerTable()  
```  
  
## Return Value  
 **true** uses a file for each table. Otherwise, **false**.  
  
## Exceptions  
 [SQLServerException](../../../connect/jdbc/reference/sqlserverexception-class.md)  
  
## Remarks  
 This usesLocalFilePerTable method is specified by the usesLocalFilePerTable method in the java.sql.DatabaseMetaData interface.  
  
## See Also  
 [SQLServerDatabaseMetaData Methods](../../../connect/jdbc/reference/sqlserverdatabasemetadata-methods.md)   
 [SQLServerDatabaseMetaData Members](../../../connect/jdbc/reference/sqlserverdatabasemetadata-members.md)   
 [SQLServerDatabaseMetaData Class](../../../connect/jdbc/reference/sqlserverdatabasemetadata-class.md)  
  
  
