# Monitoring AWS Systems Manager<a name="monitoring"></a>

SSM Agent writes information about executions, scheduled actions, errors, and health statuses to log files on each instance\. Manually connecting to an instance to view log files and troubleshoot an issue with SSM Agent is time\-consuming\. For more efficient instance monitoring, you can configure either SSM Agent itself or the CloudWatch agent to send this log data to Amazon CloudWatch Logs\. 

**Important**  
The unified CloudWatch agent has replaced SSM Agent as the tool for sending log data to Amazon CloudWatch Logs\. Support for using SSM Agent to send log data will be deprecated in the near future\. We recommend using only the unified CloudWatch agent for your log collection processes\. For more information, see the following topics:  
[Sending Logs to CloudWatch Logs \(CloudWatch agent\)](monitoring-cloudwatch-agent.md)
[Migrate Windows Server Instance Log Collection to the CloudWatch agent](monitoring-cloudwatch-agent.md#monitoring-cloudwatch-agent-migrate)
[Collect Metrics from Amazon Elastic Compute Cloud Instances and On\-Premises Servers with the CloudWatch agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html) in the *Amazon CloudWatch User Guide*

Using CloudWatch Logs, you can monitor log data in real\-time, search and filter log data by creating one or more metric filters, and archive and retrieve historical data when you need it\. For more information about CloudWatch Logs, see the *[Amazon CloudWatch Logs User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/)*\.

Configuring an agent to send log data to Amazon CloudWatch Logs provides the following benefits:
+ Centralized log file storage for all of your SSM Agent log files\.
+ Quicker access to files to investigate errors\.
+ Indefinite log file retention \(configurable\)\.
+ Logs can be maintained and accessed regardless of the status of the instance\.
+ Access to other CloudWatch features such as metrics and alarms\.

For information about monitoring Session Manager activity, see [Auditing and Logging Session Activity](session-manager-logging-auditing.md)\.

**Topics**
+ [Sending Logs to CloudWatch Logs \(SSM Agent\)](monitoring-ssm-agent.md)
+ [Sending Logs to CloudWatch Logs \(CloudWatch agent\)](monitoring-cloudwatch-agent.md)
+ [Logging AWS Systems Manager API Calls with AWS CloudTrail](monitoring-cloudtrail-logs.md)
+ [Monitoring Systems Manager Events with Amazon CloudWatch Events](monitoring-cloudwatch-events.md)
+ [Configuring Amazon SNS Notifications for AWS Systems Manager](monitoring-sns-notifications.md)