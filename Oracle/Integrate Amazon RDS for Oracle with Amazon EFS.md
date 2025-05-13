#oracle #rds 

by Devinder Singh, Manash Kalita, and Arnab Saha on 23 JAN 2023 in [Advanced (300)](https://aws.amazon.com/blogs/database/category/learning-levels/advanced-300/ "View all posts in Advanced (300)"), [Amazon Elastic File System (EFS)](https://aws.amazon.com/blogs/database/category/storage/amazon-elastic-file-system-efs/ "View all posts in Amazon Elastic File System (EFS)"), [Amazon RDS](https://aws.amazon.com/blogs/database/category/database/amazon-rds/ "View all posts in Amazon RDS"), [RDS for Oracle](https://aws.amazon.com/blogs/database/category/database/amazon-rds/rds-for-oracle/ "View all posts in RDS for Oracle"), [Technical How-to](https://aws.amazon.com/blogs/database/category/post-types/technical-how-to/ "View all posts in Technical How-to") [Permalink](https://aws.amazon.com/blogs/database/integrate-amazon-rds-for-oracle-with-amazon-efs/)  [Comments](https://aws.amazon.com/blogs/database/integrate-amazon-rds-for-oracle-with-amazon-efs/#Comments)  [Share](https://aws.amazon.com/blogs/database/integrate-amazon-rds-for-oracle-with-amazon-efs/#)

As customers migrate their Oracle databases to the [Amazon Relational Database Service for Oracle](https://aws.amazon.com/rds/oracle), they may often benefit from a shared file system to be available on their Oracle database systems. This is either to share files between the database and application servers or to act as a staging location to keep backups, data loads, and more. Amazon RDS for Oracle now supports [integration](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/oracle-efs-integration.html) with [Amazon Elastic File System](https://aws.amazon.com/about-aws/whats-new/2022/11/amazon-rds-oracle-amazon-elastic-file-system-integration/) (Amazon EFS), which provides a simple, serverless, set-and-forget elastic file system that lets you share file data without provisioning or managing storage. It’s built to scale on demand to petabytes without disrupting applications.

Amazon RDS for Oracle with Amazon EFS is well-suited to support a broad spectrum of use cases, such as the following:

- Share a file system between applications and multiple database servers
- Use it as an upload location for the native dumps and backups required for migration
- Store and share RMAN backup and recovery logs without the allocation of additional storage space on the server
- Use Oracle utilities such as `UTL_FILE` to read and write files

## Benefits of integrating an Amazon RDS for Oracle instance with Amazon EFS

Once Amazon RDS for Oracle has been integrated with Amazon EFS, you can transfer files between your Amazon RDS for Oracle DB instance and an Amazon EFS file system. This integration provides the following benefits:

- You can export/import Oracle Data Pump files to and from Amazon EFS to your Amazon RDS for Oracle DB instance. You don’t need to copy the dump files onto Amazon RDS for Oracle storage. These operations are performed directly from the Amazon EFS file system.
- Faster migration of data as compared to migration over database link. You can use Amazon EFS file system mounted on Amazon RDS for Oracle DB instances as a landing zone for various Oracle files required for migration or data transfer.
- Using it as a landing zone helps to save the allocation of extra storage space on the Amazon RDS instance to hold the files.
- The Amazon EFS file systems can automatically scale from gigabytes to petabytes of data without needing to provision storage.
- There are no minimum fees or setup costs, and you pay only for what you use.

In this post, we walk through a step-by-step configuration to set up Amazon EFS on an Amazon RDS for Oracle database instance. We also talk about the benefits of this integration, and the best practices to consider while doing so. Before starting, review the [requirements and restrictions](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/oracle-efs-integration.html) of Amazon EFS integration with Oracle Database.

## [Create an Amazon EFS file system](https://docs.aws.amazon.com/efs/latest/ug/gs-step-two-create-efs-resources.html)

Your first step is to create an Amazon EFS file system:

1. On the Amazon EFS console, choose **Create file system**.
2. For **Name**, enter a name for your file system.
3. For Amazon [**Virtual Private Cloud**](https://aws.amazon.com/vpc/) **(Amazon VPC)**, choose the VPC in which you have an Amazon RDS for Oracle instance deployed.
4. For [**Storage class**](https://docs.aws.amazon.com/efs/latest/ug/storage-classes.html), select **Standard**.
5. Choose **Create**. (by default, it will inherit the default security group, so make sure to select appropriate security group under customize while creating)

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661createfs.png)

## [Configure Amazon EFS file system permissions](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/oracle-efs-integration.html#oracle-efs-integration.file-system)

After you create a new Amazon EFS file system, by default only the root user (UID 0) has read, write, and run permissions. For other users to modify the file system, the root user must explicitly grant them access. You must mount the Amazon EFS file system locally on your [Amazon Elastic Compute Cloud](http://aws.amazon.com/ec2) (Amazon EC2) instance and set up fine-grained permissions to allow Amazon RDS instances to read and write files from and to the Amazon EFS file system. For example, you can run `chmod 777` against the Amazon EFS file system root to grant other users the permission to write to this directory. Complete the following steps:

1. Select the security group of the Amazon EC2 instance where Amazon EFS file system will be mounted.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661fsprops.png)

2. Edit the inbound rules and add port 2049 (default port for NFS) and click Save rules

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661inboundrules.png)

3. Now attach this security group to the Amazon EFS mount point in each AZ. On the **Network** tab for the Amazon EFS, select your mount point and choose **Manage**.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661mounttargets.png)

4. Connect to the Amazon EC2 instance and mount your file system on the Amazon EC2 instance:

```bash
$sudo mount -t efs -o tls fs-05cef2152acda9175:/ /efsdir
```

Please ensure [NFS Client](https://docs.aws.amazon.com/efs/latest/ug/wt1-test.html#wt1-connect-install-nfs-client) is installed on your Amazon EC2 instance (please refer to Step 3.2).

Add an entry to `/etc/fstab` to make it persistent across reboots.

5. Create a directory on the mounted EFS file system

```bash
$sudo mkdir /efsdir/datapump
```

6. Change permissions so that Amazon RDS for Oracle can write to this directory:

```bash
$sudo chmod -R 777 /efsdir
```

## [Create an option group](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithOptionGroups.html#USER_WorkingWithOptionGroups.Create)

Now you create an Amazon RDS option group:

1. On the Amazon RDS console, choose [**Option groups**](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithOptionGroups.html) in the navigation pane.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661optiongroups.png)

2. Choose **Create group**.
3. For **Name**, enter a name for your group (for example, `efs-integration-option-grp`).
4. For **Description**, enter a brief description (for example, EFS integration with RDS Oracle).
5. For **Engine**, choose **oracle-ee**.
6. For **Major Engine Version**, choose **19**.
7. Choose **Create**.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661createOG.png)

8. On the **Option groups** page, select the option group that you created and choose **Add option**.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661verifyOG.png)

9. For **Option name**, choose **EFS_INTEGRATION**.
10. Under **Option settings**, for **EFS_ID**, enter the ID of the file system that you created earlier.
11. Choose **Add option**.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661addoption.png)

**Add the option group to your Amazon RDS for Oracle instance**

To add the option group to your Amazon RDS for Oracle instance, complete the following steps:

1. On the Amazon RDS console, choose **Databases** in the navigation pane.
2. Select your instance and choose **Modify**.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661fsdetails.png)

3. For **Option group**, choose the option group that you created

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661dboptions.png)

4. Choose **Continue** and check the summary of modifications

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661delete.png)

5. Choose **Modify DB instance** and select **Apply immediately**.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661moddb.png)

## Configure network and security group permissions for Amazon RDS for Oracle integration with Amazon EFS

For Amazon RDS for Oracle to integrate with Amazon EFS, your DB instance must have [network access to an Amazon EFS file system](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/oracle-efs-integration.html#oracle-efs-integration.network). Your VPC should have the `enableDnsSupport` attribute enabled. To allow access to the Amazon EFS file system for your DB instance, make sure the Amazon EFS file system has a mount target in each Availability Zone (AZ) and that the security group attached to the mount targets has an inbound rule allowing the Amazon RDS instance to make a TCP connection to the NFS port 2049 and receive return traffic.

1. On the Amazon RDS console, select your Amazon RDS for Oracle instance. Under Connectivity & Security click on the security group.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661endpoint.png)

2. Select the security group for your Amazon RDS for Oracle instance and on the **Actions** menu, choose **Edit inbound rules**.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661editinboundrules.png)

3. Add a rule allowing access on port 2049. ( default NFS port).
4. Choose **Save**.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661port2049.png)

5. Now attach this security group to the Amazon EFS mount point in each AZ. On the **Network** tab for the Amazon EFS, select your mount point and choose **Manage**.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661network.png)

6. Add the security group for Amazon RDS for Oracle to the mount point in each AZ.
7. Choose **Save**.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661vpcadd.png)

## Transfer files between Amazon RDS for Oracle and an Amazon EFS file system

[To transfer files between an Amazon RDS for Oracle instance and an Amazon EFS file system](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/oracle-efs-integration.html#oracle-efs-integration.transferring), you must create an Oracle directory on Amazon RDS for Oracle. See the following code.

Example commands are run by admin user, other users may need proper privileges to run these commands. Note that file system path must begin with `/rdsefs-`

```bash
BEGIN
rdsadmin.rdsadmin_util.create_directory_efs(
p_directory_name => 'DATA_PUMP_DIR_EFS',
p_path_on_efs => '/rdsefs-fs-05cef2152acda9175/datapump');
END;
/
```

### 1. Use DATAPUMP to export a table

The following PL/SQL code example shows how to export the table `EMP` in the `HR_USER` schema, and the dump file is created in the Oracle directory `DATA_PUMP_DIR_EFS`, which is on Amazon EFS:  
declare

```bash
l_dp_handle       number;
begin
  -- Open a table export job.
  l_dp_handle := dbms_datapump.open(
    operation   => 'EXPORT',
    job_mode    => 'TABLE',
    remote_link => NULL,
    job_name    => 'HR_USER_TBL_EXP',
    version     => 'LATEST');
  -- Specify the dump file name and directory object name.
  dbms_datapump.add_file(
    handle    => l_dp_handle,
    filename  => 'hr_user.dmp',
    directory => 'DATA_PUMP_DIR_EFS');
  -- Specify the log file name and directory object name.
  dbms_datapump.add_file(
    handle    => l_dp_handle,
    filename  => 'h_user.log',
    directory => 'DATA_PUMP_DIR_EFS',
    filetype  => DBMS_DATAPUMP.KU$_FILE_TYPE_LOG_FILE);
  -- Specify the table to be exported, filtering the schema and table.
  dbms_datapump.metadata_filter(
    handle => l_dp_handle,
    name   => 'SCHEMA_EXPR',
    value  => '= ''HR_USER''');
  dbms_datapump.metadata_filter(
    handle => l_dp_handle,
    name   => 'NAME_EXPR',
    value  => '= ''EMP''');
  dbms_datapump.start_job(l_dp_handle);
  dbms_datapump.detach(l_dp_handle);
end;
/
```

This creates the file `hr_user.dmp` in the Oracle directory ‘`DATA_PUMP_DIR_EFS`‘ and you can check from Amazon EC2 where Amazon EFS is mounted.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661efsdatapump.png)

### 2. Use UTL_FILE to write and read from files

To write and read from files, use the following code:

```bash
declare
  fhandle  utl_file.file_type;
begin
  fhandle := utl_file.fopen(
                'DATA_PUMP_DIR_EFS'     -- File location
              , 'test_file.txt' -- File name
              , 'w' -- Open mode: w = write.
 );
  utl_file.put(fhandle, 'Hello world!'  || CHR(10));
  utl_file.put(fhandle, 'Hello again!');
  utl_file.fclose(fhandle);
exception
  when others then
    dbms_output.put_line('ERROR: ' || SQLCODE || ' - ' || SQLERRM);
    raise;
end;
/
```

You can check from the Amazon EC2 instance that the file was created.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661efs.png)

###  3. Use Oracle [RMAN](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.Oracle.CommonDBATasks.RMAN.html) to back up your Amazon RDS for Oracle database to the shared EFS file system

The following section lists steps to perform the backup of your Amazon RDS for Oracle database using Oracle Recovery Manager (RMAN) and store the backup pieces in the Amazon EFS file system

1. Create a directory on the OS and change permissions from the Amazon EC2 server where you have mounted the Amazon EFS

```bash
sudo mkdir /efsdir/rman
sudo chmod -R 777 /efsdir
```

2. Create Oracle Directory.

```sql
BEGIN
rdsadmin.rdsadmin_util.create_directory_efs(
p_directory_name => 'RMAN_DIR_EFS',
p_path_on_efs => '/rdsefs-fs-05cef2152acda175/rman');
END;
/
```

This will create the database directory name `RMAN_DIR_EFS` to store the RMAN Backups. The file system path value for the `_p_path_on_efs_` parameter is prefixed with the string `“/rdsefs- <EFS FILE SYSTEM ID>”.`

3. Make sure the archive logs are retained on the RDS database server as long as the Oracle RMAN tool requires them. In our example we have used 2 hours as retention of archive logs.

```sql
begin
rdsadmin.rdsadmin_util.set_configuration(
name => 'archivelog retention hours',
value => '2');
end;
/
commit;
```

4. Run the RMAN Backup for the Amazon RDS for Oracle instance. Note that the backup location is specified as the Oracle Directory pointing to Amazon EFS File system. Refer to this [document](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.Oracle.CommonDBATasks.RMAN.html) for common RMAN tasks related to the backup.

```sql
BEGIN
rdsadmin.rdsadmin_rman_util.backup_database_full(
p_owner => 'SYS',
p_directory_name => 'RMAN_DIR_EFS',
p_parallel => 4,
p_section_size_mb => 10,
p_tag => 'FULL_DB_BACKUP',
p_rman_to_dbms_output => FALSE);
END;
/
```

5. Check the RMAN backup logs.

RMAN log files are stored in the `bdump` directory. Use the following query to get the logfile name:

```sql
SELECT * FROM table(rdsadmin.rds_file_util.listdir('BDUMP')) order by mtime;
```

Run the following query to open the log file and check for any errors. Replace the file name obtained from the previous query and add it in the next query.

```sql
SELECT text FROM table(rdsadmin.rds_file_util.read_text_file('BDUMP', 'rds-rman-backup-database-2023-01-04.14-01-58.933233000.txt'));
```

6. Log in to the EC2 instance where the same Amazon EFS file system is mounted. The RMAN Backup pieces are stored in the Amazon EFS file system.

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2023/01/17/DBBLOG-2661rman.png)

## Conclusion

In this post, we showed how to integrate and configure Amazon EFS with Amazon RDS for Oracle. You can use this feature to share data between databases and application servers and among database servers. We demonstrated use cases where you can also use it when performing RMAN backups and native Data Pump from Amazon RDS for Oracle without consuming database storage on the RDS instance.