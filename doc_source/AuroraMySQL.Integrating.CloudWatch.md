# Exporting Audit Log Data From Amazon Aurora to Amazon CloudWatch Logs<a name="AuroraMySQL.Integrating.CloudWatch"></a>

You can configure an Amazon Aurora DB cluster to export audit log events to a log group in Amazon CloudWatch Logs\. You can use this functionality to perform real\-time analysis of the audit events observed on your DB cluster, using CloudWatch to create alarms and view metrics\. You can use CloudWatch Logs to store your log data in highly durable storage\. You can change the log retention setting so that any log events older than this setting are automatically deleted\. The CloudWatch Logs agent makes it easy to quickly send both rotated and non\-rotated log data off of a host and into the log service\. You can then access the raw log data when you need it\.

**Note**  
Exporting audit log data from Aurora to CloudWatch Logs can incur data ingestion and archived storage charges on CloudWatch Logs\. You are only charged for exporting audit log data that exceeds the free tier provided by CloudWatch Logs\. For more information, see [Amazon CloudWatch Pricing](https://aws.amazon.com/cloudwatch/pricing/)\.

## Giving Aurora Access to Amazon CloudWatch Logs<a name="AuroraMySQL.Integrating.CloudWatch.Access"></a>

Before you can export audit logs to Amazon CloudWatch Logs, you must first give your Aurora DB cluster permission to access CloudWatch Logs\. To grant permission, create an AWS Identity and Access Management \(IAM\) role with the necessary permissions, and then associate the role with your DB cluster\. For details and instructions on how to permit your Aurora DB cluster to communicate with CloudWatch Logs on your behalf, see [Authorizing Amazon Aurora MySQL to Access Other AWS Services on Your Behalf](AuroraMySQL.Integrating.Authorizing.md)\.

**Note**  
You must set the `aws_default_logs_role` DB cluster parameter to the Amazon Resource Name \(ARN\) of the new IAM role\. If an IAM role isn't specified for the `aws_default_logs_role` the logs will not be streamed to CloudWatch\. If the specified role does not have the required permissions, we will report this situation through events\.  
For more information about DB cluster parameters, see [Amazon Aurora MySQL Parameters](AuroraMySQL.Reference.md#AuroraMySQL.Reference.ParameterGroups)\.

## Enabling Audit Log Exporting to Amazon CloudWatch Logs in an Aurora DB Cluster<a name="AuroraMySQL.Integrating.CloudWatch.Enable"></a>

To start exporting audit log to Amazon CloudWatch Logs, you need to first enable Advanced Auditing\. For more information about enabling Advanced Auditing, see [Enabling Advanced Auditing](AuroraMySQL.Auditing.md#AuroraMySQL.Auditing.Enable)\.

After you enabled Advanced Auditing and defined the events to be audited, you can enable audit log exporting to CloudWatch Logs\. To do so, set the `server_audit_logs_upload` parameter, which is a cluster\-level DB parameter\. This parameter enables or disables audit log exporting\. This parameter defaults to 0; to enable audit log exporting, set it to 1\.

You can configure audit log exporting to CloudWatch Logs the same way that you configure Advanced Auditing, by setting these parameters in the parameter group used by your DB cluster\. You can use the procedure shown in [Modifying Parameters in a DB Parameter Group](USER_WorkingWithParamGroups.md#USER_WorkingWithParamGroups.Modifying) to modify DB cluster parameters using the AWS Management Console\. You can use the `modify-db-cluster-parameter-group` AWS CLI command or the `ModifyDBClusterParameterGroup` Amazon RDS API command to modify DB cluster parameters programmatically\. Modifying these parameters doesn't require a DB cluster restart\.

**Note**  
You can identify the DB instance for each event by using the `serverhost` field included in the audit log details\. The full content of the event is exported to CloudWatch Logs\. For more information about the `serverhost` field, see [Audit Log Details](AuroraMySQL.Auditing.md#AuroraMySQL.Auditing.Logs)\.  
CloudWatch Logs events will persist after your DB instances are terminated\. The log retention period for each log group can be configured using the CloudWatch Logs management console, the AWS CLI, or the CloudWatch Logs API\.  
For more information about CloudWatch Logs, see [What is Amazon CloudWatch Logs?](http://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html) 

### Encryption Support<a name="AuroraMySQL.Integrating.CloudWatch.Enable.Encryption"></a>

Audit log data is automatically encrypted in transit and at rest by CloudWatch Logs, using its default AWS Key Management Service \(AWS KMS\) encryption key\. Currently, you cannot specify a custom AWS KMS key for encrypting audit log data in CloudWatch Logs\.

## Monitoring Audit Log Events in Amazon CloudWatch<a name="AuroraMySQL.Integrating.CloudWatch.Stream"></a>

After enabling the feature, you can monitor the audit log events in Amazon CloudWatch Logs\. A new log group is automatically created for the Aurora DB cluster under the following prefix, in which *cluster\-name* represents the DB cluster name:

```
/aws/rds/cluster/cluster-name
```

If a log group with the specified name exists, Aurora uses that log group to export audit log data for the Aurora DB cluster\. You can use automated configuration, such as AWS CloudFormation, to create log groups with predefined log retention periods, metric filters, and customer access\. Otherwise, a new log group is automatically created is created using the default log retention period, **Never Expire**, on CloudWatch Logs\. You can use the CloudWatch Logs management console, the AWS CLI, or the CloudWatch Logs API to change the log retention period\. For more information about changing log retention periods in CloudWatch Logs, see [Change Log Data Retention in CloudWatch Logs](http://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/SettingLogRetention.html)\.

All the audit events from all of the DB instances in a DB cluster are pushed to this log group using different log streams\. You can use the CloudWatch Logs console, the AWS CLI, or the CloudWatch Logs API to search information within the log events for a DB cluster\. For more information about searching and filtering log data, see [Searching and Filtering Log Data](http://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/MonitoringLogData.html)\.

**Note**  
Aurora does not delete existing log groups or log streams if exporting audit log data is disabled\. Existing audit log data remains available in CloudWatch Logs, depending on log retention, and you still incur charges for stored audit log data\. You can delete log streams and log groups using the CloudWatch Logs management console, the AWS CLI, or the CloudWatch Logs API\.