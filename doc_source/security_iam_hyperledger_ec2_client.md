# Example IAM Role Permissions Policy for Hyperledger Fabric Client EC2 Instance<a name="security_iam_hyperledger_ec2_client"></a>

When you administer, develop, and deploy chaincode using an EC2 instance as a Hyperledger Fabric client, the permissions policies attached to the AWS Identity and Access Management *instance profile* and *instance role* associated with the instance determine its permissions to interact with other AWS resources, including Managed Blockchain\. The permissions policy shown in the following procedure provides sufficient privileges when it is attached to the IAM role of the instance\. 

The procedure demonstrates how to create a role with only this permissions policy attached and then attach that role to an EC2 instance\. If you have an existing service role and instance profile attached to your EC2 instance, you can create an additional policy using the following example and then attach it to the existing role\. Only one role can be attached to an EC2 instance\.

For more information, see [IAM roles for Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) in the *Amazon EC2 User Guide for Linux Instances*\.

**To create a permissions policy, attach it to an IAM role, and attach the role to a Hyperledger Fabric client EC2 instance**

1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. In the navigation pane, choose **Policies**, and then **Create policy**\.

1. Choose the `JSON` tab, and then copy and paste the following policy statement\. 

   ```
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "ListNetworkMembers",
         "Effect": "Allow",
         "Action": [
             "managedblockchain:GetNetwork",
             "managedblockchain:ListMembers"
         ],
         "Resource": [
             "arn:aws:managedblockchain:*:123456789012:networks/*"
         ]
       },
       {
         "Sid": "AccessManagedBlockchainBucket",
         "Effect": "Allow",
         "Action": [
           "s3:GetObject"
         ],
         "Resource": "arn:aws:s3:::us-east-1.managedblockchain/*"
       },
       {
         "Sid": "ManageNetworkResources",
         "Effect": "Allow",
         "Action": [
           "managedblockchain:CreateProposal",
           "managedblockchain:GetProposal",
           "managedblockchain:DeleteMember",
           "managedblockchain:VoteOnProposal",
           "managedblockchain:ListProposals",
           "managedblockchain:GetNetwork",
           "managedblockchain:ListMembers",
           "managedblockchain:ListProposalVotes",
           "managedblockchain:RejectInvitation",
           "managedblockchain:GetNode",
           "managedblockchain:GetMember",
           "managedblockchain:DeleteNode",
           "managedblockchain:CreateNode",
           "managedblockchain:CreateMember",
           "managedblockchain:ListNodes"
         ],
         "Resource": [
           "arn:aws:managedblockchain:*::networks/*",
           "arn:aws:managedblockchain:*::proposals/*",
           "arn:aws:managedblockchain:*:123456789012:members/*",
           "arn:aws:managedblockchain:*:123456789012:invitations/*",
           "arn:aws:managedblockchain:*:123456789012:nodes/*"
         ]
       },
       {
         "Sid": "WorkWithNetworksForAcct",
         "Effect": "Allow",
         "Action": [
           "managedblockchain:ListNetworks",
           "managedblockchain:ListInvitations",
           "managedblockchain:CreateNetwork"
         ],
         "Resource": "*"
       }
     ]
   }
   ```

1. Replace *123456789012* with your AWS Account ID\. If you are working in a different Region, replace *us\-east\-1* with the appropriate Region, and then choose **Review policy**\.

1. Enter a **Name** for the policy, for example, `HyperledgerFabricClientAccess`\. Enter an optional **Description**, and then choose **Create policy**\.

   You now have a permissions policy that you can attach to an IAM role for an EC2 instance\.

1. From the navigation pane, choose **Roles**, **Create role**\.

1. Under **Select type of trusted entity**, leave **AWS service** selected\. Under **Common use cases**, choose **EC2 \- Allows EC2 instances to call AWS services on your behalf**, and then choose **Next: Permissions**\.

1. Under **Attach permissions policies**, start typing the name of the permissions policy you created in the previous steps\. Select that policy from the list, and then choose **Next: Tags**\.

1. Leave the key\-value fields blank or type key\-value pairs for any tags that you want to apply to this role, and then choose **Next: Review**\.

1. Enter a **Role name** that helps you identify this role, for example, **ServiceRoleForHyperledgerFabricClient**\. Enter an optional **Description**, and then choose **Create role**\.

   You now have a service role with appropriate permissions that you can associate with your Hyperledger Fabric EC2 instance\.

1. Open the EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. From the navigation pane, choose **Instances**\.

1. From the list of instances, select the instance that you are using as a Hyperledger Fabric client\.

1. Choose **Actions**, **Instance Settings**, **Modify IAM role**\.

1. For **IAM role** begin typing the name of the role you created, for example, **ServiceRoleForHyperledgerFabricClient**\. Select it from the list, and then choose **Apply**\.