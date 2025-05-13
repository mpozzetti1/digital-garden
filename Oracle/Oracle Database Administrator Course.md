#oracle 

![[Pasted image 20250508105950.png]]
# Introduction

> Oracle Database architecture is designed to efficiently manage data storage, processing, and retrieval in enterprise environments. It consists of two main components: the Oracle Instance and the Database Storage.

---

# Architecture

## PGA

### Oracle Program Global Area

> The Program Global Area (PGA) is a memory region that is allocated by the Oracle instance for each server process or user session. It is used to store data and control information for the corresponding session or process. The PGA is a non-shared memory area, meaning that each server process or user session has its own separate PGA.

The PGA consists of the following components:

1. **Private SQL Area**: This area holds the execution plans and other private SQL data for the session. It stores information such as parsed SQL statements, execution plans, and other SQL-related data structures.
2. **SQL Work Areas**: SQL work areas are used for operations like sorting, hash joins, and other memory-intensive operations. These areas are allocated from the PGA and are used to store temporary data during query processing.
3. **Session Memory**: Session memory is used to store session-specific data, such as logon information, cursor states, and other session-related data structures.
4. **Other Memory Structures**: The PGA may also contain other memory structures, such as those used for parallel query execution, large object (LOB) operations, and other session-specific data.

The size of the PGA is controlled by various initialization parameters, such as `SORT_AREA_SIZE`, `HASH_AREA_SIZE`, and `PGA_AGGREGATE_TARGET`. These parameters determine the maximum amount of memory that can be allocated to the PGA for each session or process.

The Oracle database automatically manages the allocation and deallocation of memory within the PGA for each session or process. However, database administrators can monitor and tune the PGA settings based on the workload characteristics and memory requirements of the database.

## Physical Files

### Data Files (.dbf)

Data files are the physical files that store the actual data of a database, including table data, indexes, and other database objects. Each Oracle database consists of one or more data files, which are organized into logical storage units called tablespaces. Data files are read and written by the Oracle instance during database operations.

### Control Files (.ctl)

Control files are crucial components of an Oracle database that store the physical structure of the database, including information about data files, redo log files, and other important metadata. The control file is used by the Oracle instance during startup and recovery operations. It is recommended to have multiple copies of the control file for redundancy and protection against data loss.

### Redo Log Files (.log)

Redo log files are a set of files that record all changes made to the database data files. These files are used for recovery purposes in case of system failures or instances of data corruption. The redo log files are written sequentially, and once a log file is full, the Oracle instance switches to the next available redo log file. Redo log files are crucial for maintaining the integrity and recoverability of the database.

### Parameter File (.pfile)

The parameter file (or PFILE/SPFILE) is a file that stores the initialization parameters for an Oracle instance. These parameters control various aspects of the database, such as memory allocation, backup and recovery settings, and other configuration options. The parameter file is read by the Oracle instance during startup and can be modified to adjust the database behavior.

### Archived Log Files (.arc)

Archived log files are copies of the redo log files that have been filled and are no longer required for instance recovery. These files are created by the Oracle instance during log switching operations and are typically stored in a designated archive log destination. Archived log files are essential for performing database recovery operations, such as media recovery or point-in-time recovery.

### Password File (.pwd)

The password file is an optional file that stores user authentication credentials, including the system administrator (SYS) account password. It is used in environments where the database needs to be started automatically without manual intervention or when the database control file is lost or corrupted. The password file is created and managed by the Oracle Database Configuration Assistant (DBCA) or manually using the `orapwd`utility.

## SGA

### Oracle System Global Area

> The System Global Area (SGA) is a group of shared memory areas that are allocated by the Oracle instance when the database is started. It is a critical component of the Oracle database architecture, as it stores various data structures and control information required for the efficient operation of the database.

The SGA consists of the following major components:

1. **Database Buffer Cache**: The database buffer cache is the primary component of the SGA. It is used to cache data blocks read from the database data files, reducing the need for physical I/O operations. The buffer cache improves database performance by keeping frequently accessed data in memory for faster retrieval.
2. **Redo Log Buffer**: The redo log buffer is a circular buffer that temporarily stores redo entries (records of changes made to the database) before they are written to the redo log files on disk. This buffer helps improve performance by reducing the number of disk writes required for redo log entries.
3. **Shared Pool**: The shared pool is a memory area that stores shared data structures, such as shared SQL areas, shared PL/SQL areas, and dictionary caches. The shared SQL areas hold execution plans and other SQL-related data that can be shared among multiple sessions, reducing the need for repeated parsing and optimization.
4. **Large Pool**: The large pool is an optional memory area used for allocating large memory chunks for operations like backup and recovery, shared server operations, and parallel query execution.
5. **Java Pool**: The Java pool is a memory area used for Java-related operations, such as storing Java Virtual Machine (JVM) code and data.
6. **Streams Pool**: The Streams pool is a memory area used by the Oracle Streams feature, which is a data replication and integration mechanism.
7. **Other Memory Structures**: The SGA may also contain other memory structures, such as those used for locking, result caching, and other database-specific data structures.

The size of the SGA is controlled by various initialization parameters, such as `DB_CACHE_SIZE`, `SHARED_POOL_SIZE`, `LARGE_POOL_SIZE`, and `JAVA_POOL_SIZE`. These parameters determine the amount of memory allocated to each component of the SGA.

## Shared Pool

> The Shared Pool is a memory area within the Oracle System Global Area (SGA) that stores shared data structures and caches used by multiple sessions or processes. Its main components include:

1. **Shared SQL Areas**: Store execution plans and SQL-related data structures that can be shared across sessions, reducing parsing and optimization overhead.
2. **Shared PL/SQL Areas**: Store compiled PL/SQL code and metadata for PL/SQL objects, allowing reuse of compiled code across sessions.
3. **Dictionary Cache**: Caches database object metadata, reducing the need for repeated access to data dictionary tables.
4. **Library Cache**: Manages and shares objects like shared cursors, SQL areas, and PL/SQL areas among sessions.
5. **Result Cache**: (Optional) Stores results of SQL queries and PL/SQL function calls for reuse, improving performance.

The size of the Shared Pool is controlled by the `SHARED_POOL_SIZE` initialization parameter. Proper tuning of the Shared Pool is crucial for optimal database performance, as it affects SQL and PL/SQL execution, memory usage, and overall system resource utilization.

## Services

### **PMON (Process Monitor)**

PMON is a background process that monitors the other Oracle processes and performs process recovery operations when a user process fails. It is responsible for cleaning up the resources used by failed processes and releasing any locks held by those processes.

### **SMON (System Monitor)**

SMON is a background process that performs various recovery and maintenance tasks for the database instance. Its primary responsibilities include recovering instances after an instance failure, coalescing free space in the database, and cleaning up temporary segments.

### **DBWR (Database Writer)**

DBWR is a background process that writes the modified database buffers from the database buffer cache to the data files on disk. It ensures that the changes made to the database are persistently stored on the physical storage media.

### **LGWR (Log Writer)**

LGWR is a background process that writes the redo log entries from the redo log buffer to the online redo log files. It plays a crucial role in ensuring the recoverability of the database by persistently storing the changes made to the database.

### **CKPT (Checkpoint)**

CKPT is a background process that updates the control file and data file headers with the latest checkpoint information. It records the position in the redo log files where the database was most recently made consistent, enabling faster instance recovery in case of a failure.

### **Checkpoint Queue**

The Checkpoint Queue is a mechanism used by the CKPT process to coordinate the checkpoint operation with other processes, such as DBWR and LGWR. It ensures that all modified database buffers are written to the data files and all redo log entries are written to the redo log files before a checkpoint is completed.

---

# Installation and Creation

## Oracle Environment Variables

- **ORACLE_BASE**: Base directory for Oracle structure. Recommended to set before installation.
- **ORACLE_HOME**: Directory for Oracle binaries. Not needed before installation if `ORACLE_BASE` is set.
- **ORACLE_SID**: Oracle System Identifier for a database instance. Useful post-installation for easier interaction.
- **NLS_LANG**: Optional. Controls language, territory, and client character set settings.

## Manual Database Creation

```sql
CREATE DATABASE user01
  USER SYS IDENTIFIED BY ORACLE
  USER SYSTEM IDENTIFIED BY MANAGER
  CONTROLFILE REUSE
  LOGFILE
     GROUP 1 (‘HOME/ORADATA/u01/redo01.log’) SIZE 100M,
     GROUP 2 (‘HOME/ORADATA/u02/redo02.log’) SIZE 100M,
     GROUP 3 (‘HOME/ORADATA/u03/redo03.log’) SIZE 100M,
  MAXLOGFILES 5
  MAXLOGMEMBERS 5
  MAXLOGHISTORY 1
  MAXDATAFILES 100
  MAXINSTANCES 1
  ARCHIVELOG
  FORCE LOGGING
  CHARACTER SET US7ASCII
  NATIONAL CHARACTER SET AL16UTF16
  DATAFILE ‘/HOME/ORADATA/u01/system01.dbf’ SIZE 325M
  DEFAULT TEMPORARY TABLESPACE temp
  UNDO TABLESPACE undotbs
  SET TIME_ZONE = ‘America/New York’
```

## Oracle Initialization Files

### init.ora

- **Description**: Text-based file for static parameters.
- **Usage**: Changes require a database restart.

### spfile.ora

- **Description**: Binary file for static and dynamic parameters.
- **Advantages**: Persistent changes across startups and shutdowns.
- **Usage**: Dynamic parameters can be changed without a restart using `ALTER SYSTEM`.

### Parameter Types

- **Static Parameters**
    - **Require restart** (e.g., `DB_NAME`, `CONTROL_FILES`).
- **Dynamic Parameters**
    - **No restart needed** (e.g., `MEMORY_TARGET`, `LOG_ARCHIVE_DEST`).

---

# What happens first?

## Oracle Database Startup Sequence

1. **Startup Command Execution**
    - Invoked via the `STARTUP` command.
2. **Parameter File Reading**
    - **SPFILE**: Oracle attempts to read the `SPFILE` first.
    - **Fallback**: If `SPFILE` is not available, it reads the `init.ora` file instead.
3. **Instance Startup (NOMOUNT State)**
    - Allocates memory (SGA) and starts background processes.
    - No access to control files or datafiles at this stage.
4. **Database Mount (MOUNT State)**
    - **Control Files**: Oracle locates and opens the control files.
    - Information about physical database structure is read.
    - Used for recovery operations but data isn’t accessible to users.
5. **Database Open (OPEN State)**
    - **Datafiles and Redo Logs**: Oracle opens the datafiles and redo log files.
    - Database is now available for normal operations, allowing user access and transactions.

## Oracle Database Shutdown Sequence

1. **Shutdown Command Execution**
    - Various options include `SHUTDOWN NORMAL`, `IMMEDIATE`, `TRANSACTIONAL`, or `ABORT`.
2. **Database Close (OPEN → MOUNT)**
    - All datafiles and redo log files are closed.
    - No new user sessions are allowed.
3. **Database Dismount (MOUNT → NOMOUNT)**
    - Control files are closed.
    - Instance is still active with SGA and background processes running.
4. **Instance Shutdown (NOMOUNT → Shutdown)**
    - Memory (SGA) is deallocated.
    - Background processes are terminated.
    - Database is fully shut down.

---

# Types of Shutdown in Oracle

## Shutdown Normal

- **Description**: Default shutdown method.
- **Behavior**:
    - Waits for all connected users to disconnect before shutting down.
    - No new connections are permitted during the shutdown process.
- **Use Case**: Preferred for routine maintenance when you can afford to wait for user disconnections.

## Shutdown Immediate

- **Description**: Forces an immediate shutdown.
- **Behavior**:
    - Disconnects all users and rolls back active transactions.
    - Commits pending transactions.
    - Doesn’t wait for users to disconnect voluntarily.
- **Use Case**: Useful for most administrative tasks when you need rapid shutdown without data loss.

## Shutdown Transactional

- **Description**: Shuts down the database after active transactions complete.
- **Behavior**:
    - Prevents new transactions but allows current transactions to finish.
    - Disconnects users once their active transactions are complete.
- **Use Case**: Appropriate for situations where you want to ensure no active transactions are interrupted, but need to shutdown soon after.

## Shutdown Abort

- **Description**: Performs an immediate and forceful shutdown.
- **Behavior**:
    - Terminates all user sessions immediately.
    - No transaction rollbacks or commits; database must perform instance recovery upon next startup.
- **Use Case**: Emergency shutdown due to critical failure or unresponsive database.

---

# Networking

In Oracle databases, networking is handled by Oracle Net Services, which facilitates communication between clients and database servers. The listener process plays a crucial role in this communication. It listens for incoming connection requests from clients and forwards them to the appropriate database instance.

## listener.ora

The listener.ora file is used to configure the listener process. It specifies the protocol addresses on which the listener listens for incoming connections, as well as the database services (SIDs) that it should handle.

```bash
LISTENER =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = myhost.example.com)(PORT = 1521))
  )

SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = mydb.example.com)
      (SID_NAME = mydb)
      (ORACLE_HOME = /u01/app/oracle/product/12.2.0/dbhome_1)
    )
  )
```

## tnsnames.ora

The tnsnames.ora file, on the other hand, contains mappings between service names (aliases) and the corresponding connection descriptors. These connection descriptors provide the necessary information for clients to connect to a specific database instance, such as the protocol, host, port, and service name.

```bash
MYDB =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = myhost.example.com)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = mydb)
    )
  )
```

When a client wants to connect to a database, it looks up the service name in the tnsnames.ora file to retrieve the connection descriptor. The client then sends a connection request to the listener using the protocol address specified in the listener.ora file. The listener receives the request, checks if it can handle the requested service (SID), and forwards the connection to the appropriate database instance.

These files work together to enable clients to connect to the desired Oracle database instances over the network. The listener.ora file configures the listener process, while the tnsnames.ora file provides the necessary connection information for clients to establish connections.

---

# Dynamic Performance Views

**Dynamic Performance Views, commonly known as V$ views, are a set of read-only views in Oracle Database.** These views provide real-time information about the database's operational environment, making them invaluable tools for database administrators (DBAs) for performance tuning, troubleshooting, and monitoring. They aggregate data from various internal structures within the database and offer insights into numerous components, such as memory utilization, session activity, instance status, and more.

## Examples of Dynamic Performance Views

- **V$CONTROLFILE**: Lists information about all control files associated with the database.
- **V$DATABASE**: Contains database-wide information, such as the database name and creation date.
- **V$DATAFILE**: Provides details about datafiles within the database, such as status and physical locations.
- **V$INSTANCE**: Shows information about the Oracle instance, including instance name and status.
- **V$PARAMETER**: Displays the current values of initialization parameters used by the instance.
- **V$SESSION**: Offers information on current user sessions, including user IDs and session status.
- **V$SGA**: Reflects the size and usage of various components of the System Global Area (SGA).
- **V$SPPARAMETER**: Lists initialization parameters that have been set explicitly in the spfile.
- **V$TABLESPACE**: Provides details about tablespaces, including their names and space usage.
- **V$THREAD**: Shows information related to the redo threads, used mainly in Oracle Real Application Clusters (RAC).
- **V$VERSION**: Displays version information about Oracle software components.