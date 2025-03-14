---
title: "setSendStringParametersAsUnicode Method (SQLServerDataSource)"
description: "setSendStringParametersAsUnicode Method (SQLServerDataSource)"
author: David-Engel
ms.author: davidengel
ms.date: "01/19/2017"
ms.service: sql
ms.subservice: connectivity
ms.topic: reference
apilocation: "sqljdbc.jar"
apiname: "SQLServerDataSource.setSendStringParametersAsUnicode"
apitype: "Assembly"
---
# setSendStringParametersAsUnicode Method (SQLServerDataSource)
[!INCLUDE[Driver_JDBC_Download](../../../includes/driver_jdbc_download.md)]

  Sets a **boolean** value that indicates if sending string parameters to the server in UNICODE format is enabled.  
  
## Syntax  
  
```  
  
public void setSendStringParametersAsUnicode(boolean sendStringParametersAsUnicode)  
```  
  
#### Parameters  
 *sendStringParametersAsUnicode*  
  
 **true** if string parameters are sent to the server in UNICODE format. Otherwise, **false**.  
  
## Remarks  
 If the sendStringParametersAsUnicode property is set to **true**, which is the default value, string parameters are sent to the server in UNICODE format. If sendStringParametersAsUnicode is set to **false** string parameters are sent to the server in an ASCII/MBCS format, not in UNICODE. If sendStringParametersAsUnicode is not set, [getSendStringParametersAsUnicode](../../../connect/jdbc/reference/getsendstringparametersasunicode-method-sqlserverdatasource.md) returns the default value of **true**.  

Note: Changing this value can affect sorting of results from the database. The sorting differences are due to different sorting rules for Unicode versus non-Unicode characters.

For more information about the sendStringParametersAsUnicode connection property, see [Setting the Connection Properties](../../../connect/jdbc/setting-the-connection-properties.md).  
  
## See Also  
 [SQLServerDataSource Members](../../../connect/jdbc/reference/sqlserverdatasource-members.md)   
 [SQLServerDataSource Class](../../../connect/jdbc/reference/sqlserverdatasource-class.md)  
  
  
