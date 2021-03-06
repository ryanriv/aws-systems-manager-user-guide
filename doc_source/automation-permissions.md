# Method 2: Use IAM to Configure Roles for Automation<a name="automation-permissions"></a>

Configuring access to Systems Manager Automation requires that you complete the following tasks\.

1. **Verify user access**: Verify that you have permission to run Automation workflows\. If your AWS Identity and Access Management \(IAM\) user account, group, or role is assigned administrator permissions, then you have access to Systems Manager Automation\. If you don't have administrator permissions, then an administrator must give you permission by assigning the `AmazonSSMFullAccess` managed policy, or a policy that provides comparable permissions, to your IAM account, group, or role\.

1. **Configure instance access by creating and assigning an instance profile role \(Required\)**: Each instance that runs an Automation workflow requires an IAM instance profile role\. This role gives Automation permission to perform actions on your instances, such as running commands or starting and stopping services\. If you previously created an instance profile role for Systems Manager, as described in [Create an IAM Instance Profile for Systems Manager](setup-instance-profile.md), then you can use this same instance profile role for Automation\. For information about how to attach this role to an existing instance, see [Attaching an IAM Role to an Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html#attach-iam-role) in the *Amazon EC2 User Guide*\.

**Note**  
Automation previously required that you specify a service role \(or an *assume* role\) so that the service had permission to perform actions on your behalf\. Automation no longer requires this role because the service now operates by using the context of the user who invoked the execution\.   
However, the following situations still require that you specify a service role for Automation:  
When you want to restrict a user's privileges on a resource, but you want the user to run an Automation workflow that requires elevated privileges\. In this scenario, you can create a service role with elevated privileges and allow the user to run the workflow\.
Operations that you expect to run longer than 12 hours require a service role\.

If you need to create an instance profile role and a service role for Systems Manager Automation, complete the following tasks\.

**Topics**
+ [Task 1: Create a Service Role for Automation](#automation-role)
+ [Task 2: Add a Trust Relationship for Automation](#automation-trust2)
+ [Task 3: Attach the iam:PassRole Policy to Your Automation Role](#automation-passpolicy)
+ [Task 4: Configure User Access to Automation](#automation-passrole)
+ [Task 5: Create an Instance Profile Role](#automation-instance-role)

## Task 1: Create a Service Role for Automation<a name="automation-role"></a>

Use the following procedure to create a service role \(or *assume role*\) for Systems Manager Automation\.

**Note**  
You can also use these roles and their Amazon Resource Names \(ARNs\) in Automation documents, such as the `AWS-UpdateLinuxAmi` document\. Using these roles or their ARNs in Automation documents enables Automation to perform actions on your managed instances, launch new instances, and perform actions on your behalf\.

**To create an IAM role and allow Automation to assume it**

1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the navigation pane, choose **Roles**, and then choose **Create role**\.

1. On the **Select type of trusted entity** page, under **AWS Service**, choose **EC2**\.

1. In the **Select your use case** section, choose **EC2**, and then choose **Next: Permissions**\.

1. On the **Attached permissions policy** page, search for the **AmazonSSMAutomationRole** policy, choose it, and then choose **Next: Review**\. 

1. On the **Review** page, type a name in the **Role name** box, and then type a description\.

1. Choose **Create role**\. The system returns you to the **Roles** page\.

1. On the **Roles** page, choose the role you just created to open the **Summary** page\. Note the **Role Name** and **Role ARN**\. You will specify the role ARN when you attach the iam:PassRole policy to your IAM account in the next procedure\. You can also specify the role name and the ARN in Automation documents\.

**Note**  
The AmazonSSMAutomationRole policy assigns the Automation role permission to a subset of AWS Lambda functions within your account\. These functions begin with "Automation"\. If you plan to use Automation with Lambda functions, the Lambda ARN must use the following format:  
`"arn:aws:lambda:*:*:function:Automation*"`  
If you have existing Lambda functions whose ARNs do not use this format, then you must also attach an additional Lambda policy to your automation role, such as the **AWSLambdaRole** policy\. The additional policy or role must provide broader access to Lambda functions within the AWS account\.

### \(Optional\) Add an Automation Inline Policy to Invoke Other AWS Services<a name="automation-role-add-inline-policy"></a>

If you run an automation that invokes other AWS services by using an IAM service role, the service role must be configured with permission to invoke those services\. This requirement applies to all AWS Automation documents \(AWS\-\* documents\) such as the AWS\-ConfigureS3BucketLogging, AWS\-CreateDynamoDBBackup, and AWS\-RestartEC2Instance documents, to name a few\. This requirement also applies to any custom Automation documents you create that invoke other AWS services by using actions that call other services\. For example, if you use the aws:executeAwsApi, aws:CreateStack, or aws:copyImage actions, to name a few, then you must configure the service role with permission to invoke those services\. You can enable permissions to other AWS services by adding an IAM inline policy to the role\. 

**To embed an inline policy for a service role \(IAM Console\)**

1. Sign in to the AWS Management Console and open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the navigation pane, choose **Roles**\.

1. In the list, choose the name of the role that you want to edit\.

1. Choose the **Permissions** tab\.

1. Choose **Add inline policy**\.
**Note**  
You cannot embed an inline policy in a [service\-linked role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html#iam-term-service-linked-role) in IAM\.

1. Choose the **JSON** tab\.

1. Enter a JSON policy document for the AWS services you want to invoke\. Here are two example JSON policy documents:

   **Amazon S3 PutObject and GetObject Example**

   ```
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "s3:PutObject",
                   "s3:GetObject"
               ],
               "Resource": "arn:aws:s3:::my-bucket-name/*"
           }
       ]
   }
   ```

   **Amazon EC2 CreateSnapshot and DescribeSnapShots Example**

   ```
   {
      "Version":"2012-10-17",
      "Statement":[
         {
            "Effect":"Allow",
            "Action":"ec2:CreateSnapshot",
            "Resource":"*"
         },
         {
            "Effect":"Allow",
            "Action":"ec2:DescribeSnapshots",
            "Resource":"*"
         }
      ]
   }
   ```

   For details about the IAM policy language, see [IAM JSON Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) in the *IAM User Guide*\.

1. When you are finished, choose **Review policy**\. The [Policy Validator](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_policy-validator.html) reports any syntax errors\.

1. On the **Review policy** page, enter a **Name** for the policy that you are creating\. Review the policy **Summary** to see the permissions that are granted by your policy\. Then choose **Create policy** to save your work\.

1. After you create an inline policy, it is automatically embedded in your role\.

## Task 2: Add a Trust Relationship for Automation<a name="automation-trust2"></a>

Use the following procedure to configure the service role policy to trust Automation\.

**To add a trust relationship for Automation**

1. In the **Summary** page for the role you just created, choose the **Trust Relationships** tab, and then choose **Edit Trust Relationship**\.

1. Add "ssm\.amazonaws\.com", as shown in the following example:

   ```
   {
      "Version":"2012-10-17",
      "Statement":[
         {
            "Sid":"",
            "Effect":"Allow",
            "Principal":{
               "Service":[
                  "ec2.amazonaws.com",
                  "ssm.amazonaws.com"
               ]
            },
            "Action":"sts:AssumeRole"
         }
      ]
   }
   ```

1. Choose **Update Trust Policy**\.

1. Leave the **Summary** page open\. 

## Task 3: Attach the iam:PassRole Policy to Your Automation Role<a name="automation-passpolicy"></a>

Use the following procedure to attach the iam:PassRole policy to your Automation service role\. This enables the Automation service to pass the role to other services or Systems Manager capabilities when running Automation workflows\.

**To attach the iam:PassRole policy to your Automation role**

1. In the **Summary** page for the role you just created, choose the **Permissions** tab\.

1. Choose **Add inline policy**\.

1. On the **Create policy** page, choose the **Visual editor** tab\.

1. Choose **Service**, and then choose **IAM**\.

1. Choose **Select actions**\.

1. In the **Filter actions** text box, type **PassRole**, and then choose the **PassRole** option\.

1. Choose **Resources**\. Verify that **Specific** is selected, and then choose **Add ARN**\.

1. In the **Specify ARN for role** field, paste the Automation role ARN that you copied at the end of Task 1\. The system populates the **Account** and **Role name with path** fields\.

1. Choose **Add**\.

1. Choose **Review policy**\.

1. On the **Review Policy** page, type a name and then choose **Create Policy**\.

## Task 4: Configure User Access to Automation<a name="automation-passrole"></a>

If your AWS Identity and Access Management \(IAM\) user account, group, or role is assigned administrator permissions, then you have access to Systems Manager Automation\. If you don't have administrator permissions, then an administrator must give you permission by assigning the `AmazonSSMFullAccess` managed policy, or a policy that provides comparable permissions, to your IAM account, group, or role\.

Use the following procedure to configure a user account to use Automation\. The user account you choose will have permission to configure and run Automation\. If you need to create a new user account, see [Creating an IAM User in Your AWS Account](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html) in the *IAM User Guide*\.

**To configure user access and attach the iam:PassRole policy to a user account**

1. In the IAM navigation pane, choose **Users**, and then choose the user account you want to configure\.

1. On the **Permissions** tab, in the policies list, verify that either the **AmazonSSMFullAccess** policy is listed or there is a comparable policy that gives the account permissions to access Systems Manager\.

1. Choose **Add inline policy**\.

1. On the **Set Permissions** page, choose **Policy Generator**, and then choose **Select**\.

1. Verify that **Effect** is set to **Allow**\.

1. From **AWS Services**, choose **AWS Identity and Access Management**\.

1. From **Actions**, choose **PassRole**\.

1. In the **Amazon Resource Name \(ARN\)** field, paste the ARN for the Automation service role you copied at the end of Task 1\.

1. Choose **Add Statement**, and then choose **Next Step**\.

1. On the **Review Policy** page, choose **Apply Policy**\.

## Task 5: Create an Instance Profile Role<a name="automation-instance-role"></a>

Each instance that runs an Automation workflow requires an IAM instance profile role\. This role gives Automation permission to perform actions on your instances, such as running commands or starting and stopping services\. You must create an instance profile role for Systems Manager, as described in [Create an IAM Instance Profile for Systems Manager](setup-instance-profile.md), to complete this procedure\. If you previously created an instance profile role for Systems Manager using the mentioned topic, then you can use this same instance profile role for Automation\. For information about how to attach this role to an existing instance, see [Attaching an IAM Role to an Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html#attach-iam-role) in the *Amazon EC2 User Guide*\.

You have finished configuring the required roles for Automation\. You can now use the instance profile role and the Automation service role ARN in your Automation documents\.