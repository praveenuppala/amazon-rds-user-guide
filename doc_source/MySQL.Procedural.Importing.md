# Importing Data into an Amazon RDS MySQL DB Instance<a name="MySQL.Procedural.Importing"></a>

Amazon Relational Database Service \(Amazon RDS\) supports importing MySQL databases by using backup files\. You can create a backup of your on\-premises database, store it on Amazon Simple Storage Service \(Amazon S3\), and then restore the backup file onto a new Amazon RDS DB instance running MySQL\. 

The following diagram shows the supported scenario\.

![\[MySQL importing backup files from S3 architecture\]](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/images/MySQL-bak-file.png)

Importing backup files from Amazon S3  is supported for MySQL version 5\.6\. Importing backup files from Amazon S3 is available in all AWS Regions\. 

We recommend that you import your database to Amazon RDS by using backup files if your database can be offline while the backup file is created, copied, and restored\. If your on\-premises database can't be offline, you can use binlog replication to update your database after you have migrated to Amazon RDS through Amazon S3 as explained in this topic\. For more information, see [Replication with a MySQL or MariaDB Instance Running External to Amazon RDS](MySQL.Procedural.Importing.External.Repl.md)\. You can also use the AWS Database Migration Service to migrate your database to Amazon RDS\. For more information, see [What Is AWS Database Migration Service? ](http://docs.aws.amazon.com/dms/latest/userguide/Welcome.html) 

## Limitations and Recommendations for Importing Backup Files from Amazon S3 to Amazon RDS<a name="MySQL.Procedural.Importing.Limitations"></a>

The following are some limitations and recommendations for importing backup files from Amazon S3: 

+ You can only import your data to a new DB instance, not an existing DB instance\. 

+ You must use Percona XtraBackup to create the backup of your on\-premises database\.

+ You can't migrate from a source database that has tables defined outside of the default MySQL data directory\. 

+ You can't import a MySQL 5\.5 database\. 

+ You can't import an on\-premises MySQL 5\.6 database to an Amazon RDS MySQL 5\.7 database\. You can upgrade your DB instance after you complete the import\. 

+ You can't restore databases larger than 6 TB in size\. 

+ You can't restore from an encrypted source database, but you can restore to an encrypted Amazon RDS DB instance\. 

+ Your Amazon S3 bucket can't be encrypted\. 

+ You can't restore from an Amazon S3 bucket in a different AWS Region than your Amazon RDS DB instance\. 

+ Importing from Amazon S3 is not supported on the db\.t2\.micro DB instance class\. However, you can restore to a different DB instance class, and then change the instance class later\. For more information about instance classes, see [Specifications for All Available DB Instance Classes](Concepts.DBInstanceClass.md#Concepts.DBInstanceClass.Summary)\. 

+ Amazon S3 limits the size of a file uploaded to an Amazon S3 bucket to 5 TB\. If a backup file exceeds 5 TB, then you must split the backup file into smaller files\. 

+ Amazon RDS limits the number of files uploaded to an Amazon S3 bucket to 1 million\. If the backup data for your database, including all full and incremental backups, exceeds 1 million files, use a tarball \(\.tar\.gz\) file to store full and incremental backup files in the Amazon S3 bucket\. 

+ User accounts are not imported automatically\. Save your user accounts from your source database and add them to your new DB instance later\. 

+ Functions are not imported automatically\. Save your functions from your source database and add them to your new DB instance later\. 

+ Stored procedures are not imported automatically\. Save your stored procedures from your source database and add them to your new DB instance later\. 

+ Time zone information is not imported automatically\. Record the time zone information for your source database, and set the time zone of your new DB instance later\. For more information, see [Local Time Zone for MySQL DB Instances](CHAP_MySQL.md#MySQL.Concepts.LocalTimeZone)\. 

## Overview of Setting Up to Import Backup Files from Amazon S3 to Amazon RDS<a name="MySQL.Procedural.Importing.Enabling"></a>

These are the components you need to set up to import backup files from Amazon S3 to Amazon RDS: 

+ An Amazon S3 bucket to store your backup files\.

+ A backup of your on\-premises database created by Percona XtraBackup\.

+ An AWS Identity and Access Management \(IAM\) role to allow Amazon RDS to access the bucket\.

If you already have an Amazon S3 bucket, you can use that\. If you don't have an Amazon S3 bucket, you can create a new one\. Your Amazon S3 bucket can't be encrypted\. If you want to create a new bucket, see [Creating a Bucket](http://docs.aws.amazon.com/AmazonS3/latest/user-guide/CreatingaBucket.html)\. 

Use the Percona XtraBackup tool to create your backup\. For more information, see [Creating Your Database Backup](#MySQL.Procedural.Importing.Backup)\. 

If you already have an IAM role, you can use that\. If you don't have an IAM role, you can create a new one manually\. Alternatively, you can choose to have a new IAM role created for you in your account by the wizard when you restore the database by using the AWS Management Console\. If you want to create a new IAM role manually, or attach trust and permissions policies to an existing IAM role, see [Creating an IAM Role Manually](#MySQL.Procedural.Importing.Enabling.IAM)\. If you want to have a new IAM role created for you, follow the procedure in [AWS Management Console](#MySQL.Procedural.Importing.Console) 

## Creating Your Database Backup<a name="MySQL.Procedural.Importing.Backup"></a>

Use the Percona XtraBackup software to create your backup\. Amazon RDS supports backup files created with the following versions of the Percona XtraBackup software: 

+ For MySQL 5\.6, use Percona XtraBackup version 2\.3\.

We recommend that if you don't already have Percona XtraBackup installed, you use the latest version of the software available\. You can download Percona XtraBackup from [the Percona website](https://www.percona.com/downloads/XtraBackup/LATEST/)\. 

You can create a full backup of your MySQL database files using Percona XtraBackup\. Alternatively, if you already use Percona XtraBackup to back up your MySQL database files, you can upload your existing full and incremental backup directories and files\. 

For more information about backing up your database with Percona XtraBackup, see [Percona XtraBackup \- Documentation](https://www.percona.com/doc/percona-xtrabackup/LATEST/index.html) and [The innobackupex Script](https://www.percona.com/doc/percona-xtrabackup/2.1/innobackupex/innobackupex_script.html) on the Percona website\. 

### Creating a Full Backup With Percona XtraBackup<a name="AuroraMySQL.Migrating.ExtMySQL.S3.Backup.Full"></a>

To create a full backup of your MySQL database files that can be restored from Amazon S3, use the Percona XtraBackup utility \(`innobackupex`\) to back up your database\. 

For example, the following command creates a backup of a MySQL database and stores the files in the folder `/on-premises/s3-restore/backup` folder\. 

```
innobackupex --user=<myuser> --password=<password> --no-timestamp /on-premises/s3-restore/backup
```

If you want to compress your backup into a single file \(which can be split later, if needed\), you can save your backup in one of the following formats: 

+ Gzip \(\.gz\)

+ tar \(\.tar\)

+ Percona xbstream \(\.xbstream\)

The following command creates a backup of your MySQL database split into multiple Gzip files\. 

```
innobackupex --user=<myuser> --password=<password> --stream=tar \
   /on-premises/s3-restore/backup | gzip - | split -d --bytes=500MB \
   - /on-premises/s3-restore/backup/backup.tar.gz
```

The following command creates a backup of your MySQL database split into multiple tar files\. 

```
innobackupex --user=<myuser> --password=<password> --stream=tar \
   /on-premises/s3-restore/backup | split -d --bytes=500MB \
   - /on-premises/s3-restore/backup/backup.tar
```

The following command creates a backup of your MySQL database split into multiple xbstream files\. 

```
innobackupex --stream=xbstream  \
   /on-premises/s3-restore/backup | split -d --bytes=500MB \
   - /on-premises/s3-restore/backup/backup.xbstream
```

### Using Incremental Backups With Percona XtraBackup<a name="AuroraMySQL.Migrating.ExtMySQL.S3.Backup.Incr"></a>

If you already use Percona XtraBackup to perform full and incremental backups of your MySQL database files, you don't need to create a full backup and upload the backup files to Amazon S3\. Instead, you can save a significant amount of time by copying your existing backup directories and files to your Amazon S3 bucket\. For more information about creating incremental backups using Percona XtraBackup, see [Incremental Backups with innobackupex](https://www.percona.com/doc/percona-xtrabackup/2.1/innobackupex/incremental_backups_innobackupex.html)\. 

When copying your existing full and incremental backup files to an Amazon S3 bucket, you must recursively copy the contents of the base directory\. Those contents include the full backup and also all incremental backup directories and files\. This copy must preserve the directory structure in the Amazon S3 bucket\. Amazon RDS iterates through all files and directories\. Amazon RDS uses the `xtrabackup-checkpoints` file that is included with each incremental backup to identify the base directory, and to order incremental backups by log sequence number \(LSN\) range\. 

### Backup Considerations for Percona XtraBackup<a name="AuroraMySQL.Migrating.ExtMySQL.S3.Backup.Considerations"></a>

Amazon RDS consumes your backup files based on the file name\. Name your backup files with the appropriate file extension based on the file format—for example, `.xbstream` for files stored using the Percona xbstream format\. 

Amazon RDS consumes your backup files in alphabetical order and also in natural number order\. Use the `split` option when you issue the `innobackupex` command to ensure that your backup files are written and named in the proper order\. 

Amazon RDS doesn't support partial backups created using Percona XtraBackup\. You can't use the `--include`, `--tables-file`, or `--databases` options to create a partial backup when you backup the source files for your database\. 

Amazon RDS supports incremental backups created using Percona XtraBackup with or without the `--no-timestamp` option\. We recommend that you use the `--no-timestamp` option to reduce the depth of the directory structure for your incremental backup\. 

## Creating an IAM Role Manually<a name="MySQL.Procedural.Importing.Enabling.IAM"></a>

If you don't have an IAM role, you can create a new one manually\. Alternatively, you can choose to have a new IAM role created for you by the wizard when you restore the database by using the AWS Management Console\. If you want to have a new IAM role created for you, follow the procedure in [AWS Management Console](#MySQL.Procedural.Importing.Console) 

If you want to manually create a new IAM role to use to import your database from Amazon S3, you create a role to delegate permissions from the Amazon RDS service to your Amazon S3 bucket\. When you create an IAM role, you attach trust and permissions policies\. To import your backup files from Amazon S3, use trust and permissions policies similar to the examples following\. For more information about creating the role, see [Creating a Role to Delegate Permissions to an AWS Service](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html)\. 

Alternatively, you can choose to have a new IAM role created for you by the wizard when you restore the database by using the AWS Management Console\. If you want to have a new IAM role created for you, follow the procedure in [AWS Management Console](#MySQL.Procedural.Importing.Console) 

The trust and permissions policies require that you provide an Amazon Resource Name \(ARN\)\. For more information about ARN formatting, see [Amazon Resource Names \(ARNs\) and AWS Service Namespaces](http://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html)\. 

**Example Trust Policy for Importing from Amazon S3**  

```
1. {
2.     "Version": "2012-10-17",
3.     "Statement":
4.     [{
5.         "Effect": "Allow",
6.         "Principal": {"Service":  "rds.amazonaws.com"},
7.         "Action": "sts:AssumeRole"
8.     }]
9. }
```

**Example Permissions Policy for Importing from Amazon S3 — IAM User Permissions**  

```
 1. {
 2.     "Version":"2012-10-17",
 3.     "Statement":
 4.     [
 5.         {
 6.             "Sid":"AllowS3AccessRole",
 7.             "Effect":"Allow",
 8.             "Action":"iam:PassRole",
 9.             "Resource":"arn:aws:iam::IAM User ID:role/S3Access"
10.         }
11.     ]
12. }
```

**Example Permissions Policy for Importing from Amazon S3 — Role Permissions**  

```
 1. {
 2.     "Version": "2012-10-17",
 3.     "Statement":
 4.     [
 5.         {
 6.         "Effect": "Allow",
 7.         "Action":
 8.             [
 9.                 "s3:ListBucket",
10.                 "s3:GetBucketLocation"
11.             ],
12.         "Resource": "arn:aws:s3:::bucket_name"
13.         },
14.         {
15.         "Effect": "Allow",
16.         "Action":
17.             [
18.                 "s3:GetObject"
19.             ],
20.         "Resource": "arn:aws:s3:::bucket_name/prefix*"
21.         }
22.     ]
23. }
```
If you include a file name prefix, include the asterisk \(\*\) after the prefix\. If you don't want to specify a prefix, specify only an asterisk\. 

## AWS Management Console<a name="MySQL.Procedural.Importing.Console"></a>

**To import data from Amazon S3 to a new MySQL DB instance**

1. Sign in to the AWS Management Console and open the Amazon RDS console at [https://console\.aws\.amazon\.com/rds/](https://console.aws.amazon.com/rds/)\.

1. In the top right corner of the Amazon RDS console, choose the AWS Region in which to create your DB instance\. Choose the same AWS Region as the Amazon S3 bucket that contains your database backup\. 

1. In the navigation pane, choose **Instances**\. 

1. Choose **Restore from S3** to launch the wizard\. 

   The wizard opens on the **Select Engine** page\. 

1. On the **Select Engine** page, choose the MySQL icon, and then choose the **Select** button\. 

   The **Specify source backup details** page appears\.   
![\[The page where you specify the details for your source database backup\]](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/images/mys-s3-ingestion.png)

1. On the **Specify source backup details** page, specify your backup information\. 

   1. For **Source engine**, choose **mysql**\. 

   1. For **Source engine version**, choose the MySQL version of your source database\. 

   1. For **S3 bucket**, choose your Amazon S3 bucket\. 

   1. \(Optional\) For **S3 bucket prefix**, type a file path prefix for the files stored in your Amazon S3 bucket\. If you don't specify a prefix, then RDS creates your DB instance using all of the files and folders in the root folder of the S3 bucket\. If you do specify a prefix, then RDS creates your DB instance using the files and folders in the S3 bucket where the path for the file begins with the specified prefix\. For example, suppose that you store your backup files on S3 in a sub\-folder named backups, and you have multiple sets of backup files, each in its own directory \(gzip\_backup1, gzip\_backup2, and so on\.\) In this case, you specify a prefix of backups/gzip\_backup1 to restore from the files in the gzip\_backup1 folder\. 

   1. For **Create a new role**, choose **Yes** to have a new IAM role created for you in your account, or choose **No** to select an existing IAM role\. 

   1. For **IAM Role**, select an existing IAM role, or specify the name for a new IAM Role\. You can choose to have a new IAM role created for you by choosing **Yes** for **Create a New Role**\. 

1. Choose **Next** to continue\. The **Specify DB Details** page appears\. 

   On the **Specify DB Details** page, specify your DB instance information\. For information about each setting, see [Settings for MySQL DB Instances](USER_CreateInstance.md#USER_CreateInstance.Settings)\. 
**Note**  
Be sure to allocate enough memory for your new DB instance so that the restore can succeed\. You can also allocate additional memory for future growth\. 

1. Choose **Next** to continue\. The **Configure Advanced Settings** page appears\. 

   On the **Configure Advanced Settings** page, provide additional information that Amazon RDS needs to launch the DB instance\. For information about each setting, see [Settings for MySQL DB Instances](USER_CreateInstance.md#USER_CreateInstance.Settings)\. 

1. Choose **Launch DB Instance**\. 

## CLI<a name="MySQL.Procedural.Importing.CLI"></a>

To import data from Amazon S3 to a new MySQL DB instance by using the AWS CLI, call the [restore\-db\-instance\-from\-s3](http://docs.aws.amazon.com/cli/latest/reference/rds/restore-db-instance-from-s3.html) command with the parameters following\. For information about each setting, see [Settings for MySQL DB Instances](USER_CreateInstance.md#USER_CreateInstance.Settings)\. 

**Note**  
Be sure to allocate enough memory for your new DB instance so that the restore can succeed\. You can also allocate additional memory for future growth\. 

+ `--allocated-storage`

+ `--db-instance-identifier`

+ `--db-instance-class`

+ `--engine`

+ `--master-user-name`

+ `--master-user-password`

+ `--s3-bucket-name`

+ `--s3-ingestion-role-arn`

+ `--s3-prefix`

+ `--source-engine`

+ `--source-engine-version`

**Example**  
For Linux, OS X, or Unix:  

```
 1. aws rds restore-db-instance-from-s3 \  
 2. --allocated-storage 250 \ 
 3. --db-instance-identifier myidentifier \
 4. --db-instance-class db.m4.large \
 5. --engine mysql \
 6. --master-user-name masterawsuser \
 7. --master-user-password masteruserpassword \
 8. --s3-bucket-name mybucket \
 9. --s3-ingestion-role-arn arn:aws:iam::account-number:role/rolename \
10. --s3-prefix bucketprefix \
11. --source-engine mysql \
12. --source-engine-version 5.6.27
```
For Windows:  

```
 1. aws rds restore-db-instance-from-s3 ^
 2. --allocated-storage 250 ^ 
 3. --db-instance-identifier myidentifier ^
 4. --db-instance-class db.m4.large ^
 5. --engine mysql ^
 6. --master-user-name masterawsuser ^
 7. --master-user-password masteruserpassword ^
 8. --s3-bucket-name mybucket ^
 9. --s3-ingestion-role-arn arn:aws:iam::account-number:role/rolename ^
10. --s3-prefix bucketprefix ^
11. --source-engine mysql ^
12. --source-engine-version 5.6.27
```

## API<a name="MySQL.Procedural.Importing.API"></a>

To import data from Amazon S3 to a new MySQL DB instance by using the Amazon RDS API, call the [RestoreDBInstanceFromS3](http://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_RestoreDBInstanceFromS3.html) action\. 

## Related Topics<a name="MySQL.Procedural.Importing.Related"></a>

+ [Importing Data into a MySQL DB Instance by Using Other MethodsImporting Data by Using Other Methods](MySQL.Procedural.Importing.Other.md)

+ [Backing Up and Restoring Amazon RDS DB Instances](CHAP_CommonTasks.BackupRestore.md)