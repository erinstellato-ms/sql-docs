---
title: "setBytes Method (long, byte)"
description: "setBytes Method (long, byte)"
author: David-Engel
ms.author: davidengel
ms.date: "01/19/2017"
ms.service: sql
ms.subservice: connectivity
ms.topic: reference
apilocation: "sqljdbc.jar"
apiname: "SQLServerBlob.setBytes (long.byte[])"
apitype: "Assembly"
---
# setBytes Method (long, byte)
[!INCLUDE[Driver_JDBC_Download](../../../includes/driver_jdbc_download.md)]

  Writes the given array of bytes into the BLOB starting at the given position, and then returns the number of bytes written.  
  
## Syntax  
  
```  
  
public int setBytes(long pos,  
                    byte[] bytes)  
```  
  
#### Parameters  
 *pos*  
  
 The position (1-based) in the BLOB at which to start writing the data.  
  
 *bytes*  
  
 The array of bytes to be written into the BLOB.  
  
## Return Value  
 A **long** value that specifies the number of bytes written.  
  
## Exceptions  
 java.sql.SQLException  
  
## Remarks  
 This setBytes method is specified by the setBytes method in the java.sql.Blob interface.  
  
 Data is overwritten starting at the specified position and can overrun the initial length of the BLOB. Specifying a position+1 values will append bytes. Passing a position+2 or greater (or zero or less) value will cause a position error to be thrown. Passing a zero-length **byte** array will return zero because no bytes were written.  
  
## See Also  
 [setBytes Method &#40;SQLServerBlob&#41;](../../../connect/jdbc/reference/setbytes-method-sqlserverblob.md)   
 [SQLServerBlob Methods](../../../connect/jdbc/reference/sqlserverblob-methods.md)   
 [SQLServerBlob Members](../../../connect/jdbc/reference/sqlserverblob-members.md)   
 [SQLServerBlob Class](../../../connect/jdbc/reference/sqlserverblob-class.md)  
  
  
