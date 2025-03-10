---
author: rwestMSFT
ms.author: randolphwest
ms.date: 08/02/2024
ms.service: sql
ms.topic: include
---
CLR uses Code Access Security (CAS) in the .NET Framework, which is no longer supported as a security boundary. A CLR assembly created with `PERMISSION_SET = SAFE` might be able to access external system resources, call unmanaged code, and acquire sysadmin privileges. In SQL Server 2017 (14.x) and later versions, the `sp_configure` option, [clr strict security](/sql/database-engine/configure-windows/clr-strict-security), enhances the security of CLR assemblies. `clr strict security` is enabled by default, and treats `SAFE` and `EXTERNAL_ACCESS` assemblies as if they were marked `UNSAFE`. The `clr strict security` option can be disabled for backward compatibility, but isn't recommended.

We recommend that you sign all assemblies by a certificate or asymmetric key, with a corresponding login that has been granted `UNSAFE ASSEMBLY` permission in the `master` database. SQL Server administrators can also add assemblies to a list of assemblies, which the Database Engine should trust. For more information, see [sys.sp_add_trusted_assembly](/sql/relational-databases/system-stored-procedures/sys-sp-add-trusted-assembly-transact-sql).
