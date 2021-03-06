# Getting Started with Automation<a name="automation-setup"></a>

Setting up Automation requires that you verify user access to the Automation service and configure roles so that the service can perform actions on your instances\. To configure access to Systems Manager Automation, complete the following tasks\.

1. **Verify user access**: Verify that you have permission to run Automation workflows\. If your AWS Identity and Access Management \(IAM\) user account, group, or role is assigned administrator permissions, then you have access to Systems Manager Automation\. If you don't have administrator permissions, then an administrator must give you permission by assigning the `AmazonSSMFullAccess` managed policy, or a policy that provides comparable permissions, to your IAM account, group, or role\.

1. **Configure instance access by creating and assigning an instance profile role \(Required\)**: Each instance that runs an Automation workflow requires an IAM instance profile role\. This role gives Automation permission to perform actions on your instances, such as running commands or starting and stopping services\. If you previously created an instance profile role for Systems Manager, as described in [Create an IAM Instance Profile for Systems Manager](setup-instance-profile.md), then you can use this same instance profile role for Automation\. For information about how to attach this role to an existing instance, see [Attaching an IAM Role to an Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html#attach-iam-role) in the *Amazon EC2 User Guide for Linux Instances*\.

**Note**  
Automation previously required that you specify a service role \(or *assume role*\) so that the service had permission to perform actions on your behalf\. Automation no longer requires this role because the service now operates by using the context of the user who invoked the execution\.   
However, the following situations still require that you specify a service role for Automation:  
When you want to restrict a user's privileges on a resource, but you want the user to run an Automation workflow that requires elevated privileges\. In this scenario, you can create a service role with elevated privileges and allow the user to run the workflow\.
Operations that you expect to run longer than 12 hours require a service role\.

If you need to create a service role and an instance profile role for Automation, you can use one of the following methods\.

**Topics**
+ [Method 1: Use AWS CloudFormation to Configure Roles for Automation](automation-cf.md)
+ [Method 2: Use IAM to Configure Roles for Automation](automation-permissions.md)