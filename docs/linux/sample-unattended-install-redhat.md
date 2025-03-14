---
title: Unattended install for SQL Server on RHEL
titleSuffix: SQL Server
description: Use a sample bash script to install SQL Server on Red Hat Enterprise Linux (RHEL) without interactive input.
author: rwestMSFT
ms.author: randolphwest
ms.reviewer: armaha
ms.date: 11/16/2023
ms.service: sql
ms.subservice: linux
ms.topic: conceptual
ms.custom:
  - linux-related-content
---
# Sample: Unattended SQL Server installation script for Red Hat Enterprise Linux

[!INCLUDE [SQL Server - Linux](../includes/applies-to-version/sql-linux.md)]

This sample bash script installs [!INCLUDE [ssnoversion-md](../includes/ssnoversion-md.md)] on Red Hat Enterprise Linux (RHEL) without interactive input. It provides examples of installing the [!INCLUDE [ssde-md](../includes/ssde-md.md)], the SQL Server command-line tools, SQL Server Agent, and performs post-install steps. You can optionally install full-text search and create an administrative user.

> [!TIP]  
> If you don't need an unattended installation script, the fastest way to install SQL Server is to follow the [Quickstart: Install SQL Server and create a database on Red Hat](quickstart-install-connect-red-hat.md). For other setup information, see [Installation guidance for SQL Server on Linux](sql-server-linux-setup.md).

## Prerequisites

- You need at least 2 GB of memory to run SQL Server on Linux.
- The file system must be **XFS** or **EXT4**. Other file systems, such as **BTRFS**, are unsupported.
- For other system requirements, see [System requirements for SQL Server on Linux](sql-server-linux-setup.md#system).

## Sample script

This example installs [!INCLUDE [sssql19-md](../includes/sssql19-md.md)] on RHEL 8.x. If you want to install a different version of [!INCLUDE [ssnoversion-md](../includes/ssnoversion-md.md)] or RHEL, change the Microsoft repository paths accordingly.

Save the sample script to a file. To customize it, You must replace the variable values in the script. You can also set any of the scripting variables as environment variables, as long as you remove them from the script file.

> [!IMPORTANT]  
> The `SA_PASSWORD` environment variable is deprecated. Use `MSSQL_SA_PASSWORD` instead.

```bash
#!/bin/bash -e

# Use the following variables to control your install:

# Password for the SA user (required)
MSSQL_SA_PASSWORD='<YourStrong!Passw0rd>'

# Product ID of the version of SQL Server you're installing
# Must be evaluation, developer, express, web, standard, enterprise, or your 25 digit product key
# Defaults to developer
MSSQL_PID='evaluation'

# Enable SQL Server Agent (recommended)
SQL_ENABLE_AGENT='y'

# Install SQL Server Full Text Search (optional)
# SQL_INSTALL_FULLTEXT='y'

# Create an additional user with sysadmin privileges (optional)
# SQL_INSTALL_USER='<Username>'
# SQL_INSTALL_USER_PASSWORD='<YourStrong!Passw0rd>'

if [ -z $MSSQL_SA_PASSWORD ]
then
  echo Environment variable MSSQL_SA_PASSWORD must be set for unattended install
  exit 1
fi

echo Adding Microsoft repositories...
sudo curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/8/mssql-server-2019.repo
sudo curl -o /etc/yum.repos.d/msprod.repo https://packages.microsoft.com/config/rhel/8/prod.repo

echo Installing SQL Server...
sudo yum install -y mssql-server

echo Running mssql-conf setup...
sudo MSSQL_SA_PASSWORD=$MSSQL_SA_PASSWORD \
     MSSQL_PID=$MSSQL_PID \
     /opt/mssql/bin/mssql-conf -n setup accept-eula

echo Installing mssql-tools and unixODBC developer...
sudo ACCEPT_EULA=Y yum install -y mssql-tools unixODBC-devel

# Add SQL Server tools to the path by default:
echo Adding SQL Server tools to your path...
echo PATH="$PATH:/opt/mssql-tools/bin" >> ~/.bash_profile
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
source ~/.bashrc

# Optional Enable SQL Server Agent:
if [ ! -z $SQL_ENABLE_AGENT ]
then
  echo Enable SQL Server Agent...
  sudo /opt/mssql/bin/mssql-conf set sqlagent.enabled true
fi

# Optional SQL Server Full Text Search installation:
if [ ! -z $SQL_INSTALL_FULLTEXT ]
then
    echo Installing SQL Server Full-Text Search...
    sudo yum install -y mssql-server-fts
fi

# Configure firewall to allow TCP port 1433:
echo Configuring firewall to allow traffic on port 1433...
sudo firewall-cmd --zone=public --add-port=1433/tcp --permanent
sudo firewall-cmd --reload

# Example of setting post-installation configuration options
# Set trace flags 1204 and 1222 for deadlock tracing:
#echo Setting trace flags...
#sudo /opt/mssql/bin/mssql-conf traceflag 1204 1222 on

# Restart SQL Server after making configuration changes:
echo Restarting SQL Server...
sudo systemctl restart mssql-server

# Connect to server and get the version:
counter=1
errstatus=1
while [ $counter -le 5 ] && [ $errstatus = 1 ]
do
  echo Waiting for SQL Server to start...
  sleep 5s
  /opt/mssql-tools/bin/sqlcmd \
    -S localhost \
    -U SA \
    -P $MSSQL_SA_PASSWORD \
    -Q "SELECT @@VERSION" 2>/dev/null
  errstatus=$?
  ((counter++))
done

# Display error if connection failed:
if [ $errstatus = 1 ]
then
  echo Cannot connect to SQL Server, installation aborted
  exit $errstatus
fi

# Optional new user creation:
if [ ! -z $SQL_INSTALL_USER ] && [ ! -z $SQL_INSTALL_USER_PASSWORD ]
then
  echo Creating user $SQL_INSTALL_USER
  /opt/mssql-tools/bin/sqlcmd \
    -S localhost \
    -U SA \
    -P $MSSQL_SA_PASSWORD \
    -Q "CREATE LOGIN [$SQL_INSTALL_USER] WITH PASSWORD=N'$SQL_INSTALL_USER_PASSWORD', DEFAULT_DATABASE=[master], CHECK_EXPIRATION=ON, CHECK_POLICY=ON; ALTER SERVER ROLE [sysadmin] ADD MEMBER [$SQL_INSTALL_USER]"
fi

echo Done!
```

## Run the script

To run the script:

1. Paste the sample into your favorite text editor and save it with a memorable name, like `install_sql.sh`.

1. Customize `MSSQL_SA_PASSWORD`, `MSSQL_PID`, and any of the other variables you'd like to change.

1. Mark the script as executable

   ```bash
   chmod +x install_sql.sh
   ```

1. Run the script

   ```bash
   ./install_sql.sh
   ```

## Understand the script

The first thing the bash script does is set a few variables. These variables can be either scripting variables, like the sample, or environment variables. The variable `MSSQL_SA_PASSWORD` is **required** by SQL Server installation. The others are custom variables created for the script. The sample script performs the following steps:

1. Import the public Microsoft GPG keys.

1. Register the Microsoft repositories for SQL Server and the command-line tools.

1. Update the local repositories.

1. Install SQL Server.

1. Configure SQL Server with the `MSSQL_SA_PASSWORD` and automatically accept the End-User License Agreement.

1. Automatically accept the End-User License Agreement for the SQL Server command-line tools, install them, and install the `unixODBC-devel` package.

1. Add the SQL Server command-line tools to the path for ease of use.

1. Enable the SQL Server Agent if the scripting variable `SQL_ENABLE_AGENT` is set, on by default.

1. Optionally install SQL Server Full-Text search, if the variable `SQL_INSTALL_FULLTEXT` is set.

1. Unblock port 1433 for TCP on the system firewall, necessary to connect to SQL Server from another system.

1. Optionally set trace flags for deadlock tracing (requires uncommenting the lines).

1. SQL Server is now installed, to make it operational, restart the process.

1. Verify that SQL Server is installed correctly, while hiding any error messages.

1. Create a new server administrator user if `SQL_INSTALL_USER` and `SQL_INSTALL_USER_PASSWORD` are both set.

## Unattended install

Simplify multiple unattended installs and create a stand-alone bash script that sets the proper environment variables. You can remove any of the variables the sample script uses and put them in their own bash script.

```bash
#!/bin/bash
export MSSQL_SA_PASSWORD='<YourStrong!Passw0rd>'
export MSSQL_PID='evaluation'
export SQL_ENABLE_AGENT='y'
export SQL_INSTALL_USER='<Username>'
export SQL_INSTALL_USER_PASSWORD='<YourStrong!Passw0rd>'
```

Then run the bash script as follows:

```bash
. ./my_script_name.sh
```

## Related content

- [What is SQL Server on Linux?](sql-server-linux-overview.md)
