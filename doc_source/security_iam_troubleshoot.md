# Troubleshooting Hyperledger Fabric on Amazon Managed Blockchain identity and access<a name="security_iam_troubleshoot"></a>

Use the following information to help you diagnose and fix common issues that you might encounter when working with Hyperledger Fabric on Managed Blockchain and IAM\.

**Topics**
+ [I Am Not Authorized to Perform an Action in Hyperledger Fabric on Managed Blockchain](#security_iam_troubleshoot-no-permissions)
+ [I'm an administrator and want to allow others to access Hyperledger Fabric on Managed Blockchain](#security_iam_troubleshoot-admin-delegate)

## I Am Not Authorized to Perform an Action in Hyperledger Fabric on Managed Blockchain<a name="security_iam_troubleshoot-no-permissions"></a>

If the AWS Management Console tells you that you're not authorized to perform an action, then you must contact your administrator for assistance\. Your administrator is the person that provided you with your user name and password\.

The following example error occurs when the `mateojackson` IAM user tries to use the console to create a member but does not have `managedblockchain:CreateMember` permissions\.

```
User: arn:aws:iam::123456789012:user/mateojackson is not authorized to perform: managedblockchain:CreateMember on resource: n-MWY63ZJZU5HGNCMBQER7IN6OIU
```

In this case, Mateo asks his administrator to update his policies to allow him to access the `n-MWY63ZJZU5HGNCMBQER7IN6OIU` resource using the `managedblockchain:CreateMember` action\.

## I'm an administrator and want to allow others to access Hyperledger Fabric on Managed Blockchain<a name="security_iam_troubleshoot-admin-delegate"></a>

To allow others to access Managed Blockchain, you must create an IAM entity \(user or role\) for the person or application that needs access\. They will use the credentials for that entity to access AWS\. You must then attach a policy to the entity that grants them the correct permissions in Managed Blockchain\.

To get started right away, see [Creating your first IAM delegated user and group](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-delegated-user.html) in the *IAM User Guide*\.