# Encryption at Rest for Hyperledger Fabric on Managed Blockchain<a name="managed-blockchain-encryption-at-rest"></a>

Amazon Managed Blockchain offers fully managed encryption at rest\. Managed Blockchain *encryption at rest* provides enhanced security by encrypting all data at rest on networks, members, and peer nodes using keys in AWS Key Management Service \(AWS KMS\)\. This functionality helps reduce the operational burden and complexity involved in protecting sensitive data\. With encryption at rest, you can build security\-sensitive blockchain applications that meet strict encryption compliance and regulatory requirements\.

Encryption at rest integrates with AWS KMS for managing the encryption key that is used to protect your Hyperledger Fabric data\. For more information about AWS KMS, see [AWS Key Management Service concepts](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html) in the *AWS Key Management Service Developer Guide*\.

In Hyperledger Fabric on Managed Blockchain, you can specify the type of AWS KMS key at the member level\. This encryption key is then inherited by any peer nodes that the member creates\. Data at the network level is encrypted using AWS owned keys by default at no additional cost\.

When you create a new member, you can choose one of the following types of KMS keys to protect your member and its peer nodes:
+ *AWS owned key* – The default encryption type, if no KMS key ARN is specified\. The key is owned by Managed Blockchain \(no additional charge, and no configuration required\)\.
+ *Customer managed key* – The key is stored in your AWS account and is created, owned, and managed by you\. You have full control over the key \(AWS KMS charges apply\)\.

When you access networks, members, and peer nodes, Managed Blockchain decrypts the data transparently\. You don't have to change any code or applications to use or manage encrypted data\. All Managed Blockchain operations work seamlessly on your encrypted data\.

You can specify a KMS key when you create a new member by using the AWS Management Console, the Managed Blockchain API, or the AWS Command Line Interface \(AWS CLI\)\. For more information, see the [MemberConfiguration](https://docs.aws.amazon.com/managed-blockchain/latest/APIReference/API_MemberConfiguration.html) data type in the *Amazon Managed Blockchain API Reference*\. You can pass this data type as an input parameter to the [CreateMember](https://docs.aws.amazon.com/managed-blockchain/latest/APIReference/API_CreateMember.html) or [CreateNetwork](https://docs.aws.amazon.com/managed-blockchain/latest/APIReference/API_CreateNetwork.html) API operations\.

**Note**  
By default, Amazon Managed Blockchain automatically enables encryption at rest using AWS owned keys at no additional charge\. However, AWS KMS charges apply for using a customer managed key\. For information about pricing, see [AWS Key Management Service pricing](https://aws.amazon.com/kms/pricing)\.

**Topics**
+ [Encryption at Rest: How It Works](#managed-blockchain-encryption-at-rest.how-it-works)
+ [How Managed Blockchain Uses Grants in AWS KMS](#managed-blockchain-encryption-at-rest.grants)
+ [Encryption at Rest Considerations](#managed-blockchain-encryption-at-rest.considerations)
+ [Using Customer Managed Keys in Managed Blockchain](#managed-blockchain-encryption-at-rest.using-cust-keys)

## Encryption at Rest: How It Works<a name="managed-blockchain-encryption-at-rest.how-it-works"></a>

Hyperledger Fabric on Managed Blockchain encryption at rest encrypts your data using 256\-bit Advanced Encryption Standard \(AES\-256\), which helps secure your data from unauthorized access to the underlying storage\.

Encryption at rest integrates with AWS Key Management Service \(AWS KMS\) for managing the encryption key that is used to protect your Hyperledger Fabric data\. When creating a new member, you can choose one of the following types of AWS KMS keys:
+ [AWS owned key](#managed-blockchain-encryption-at-rest.aws-owned)
+ [Customer managed key](#managed-blockchain-encryption-at-rest.customer-managed)

### AWS owned key<a name="managed-blockchain-encryption-at-rest.aws-owned"></a>

AWS owned keys aren't stored in your AWS account\. They are part of a collection of KMS keys that AWS owns and manages for use in multiple AWS accounts\. AWS services can use AWS owned keys to protect your data\.

You don't need to create or manage AWS owned keys\. However, you can't view, use, track, or audit them\. You aren't charged a monthly fee or a usage fee for AWS owned keys, and they don't count against the AWS KMS quotas for your account\.

For more information, see [AWS owned keys](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk) in the *AWS Key Management Service Developer Guide*\.

### Customer managed key<a name="managed-blockchain-encryption-at-rest.customer-managed"></a>

Customer managed keys are KMS keys in your AWS account that you create, own, and manage\. You have full control over these KMS keys\. Managed Blockchain supports symmetric customer managed keys only\.

Use a customer managed key to get the following features:
+ Setting and maintaining key policies, IAM policies, and grants to control access to the key
+ Enabling and disabling the key
+ Rotating cryptographic material for the key
+ Creating key tags and aliases
+ Scheduling the key for deletion
+ Importing your own key material or using a custom key store that you own and manage
+ Using AWS CloudTrail and Amazon CloudWatch Logs to track the requests that Managed Blockchain sends to AWS KMS on your behalf

For more information, see [Customer managed keys](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk) in the *AWS Key Management Service Developer Guide*\.

Customer managed keys [incur a charge](https://aws.amazon.com/kms/pricing/) for each API call, and AWS KMS quotas apply to these KMS keys\. For more information, see [AWS KMS resource or request quotas](https://docs.aws.amazon.com/kms/latest/developerguide/limits.html)\.

When you specify a customer managed key as the member\-level KMS key, the member and its peer nodes are protected with the same customer managed key\. Network\-level data is encrypted using AWS owned keys by default at no additional cost\.

**Inaccessible Customer Managed Keys**

If you disable your customer managed key, schedule the key for deletion, or revoke the grants on the key, the status of your member and its peer nodes becomes `INACCESSIBLE_ENCRYPTION_KEY`\. In this state, the member and its peer nodes are impaired and might not function as expected\. An inaccessible key prevents all users and the Managed Blockchain service from encrypting or decrypting data—and from performing read and write operations on the nodes\. Managed Blockchain must have access to your KMS key to ensure that you can continue to access your nodes and to prevent data loss\.

The effect of disabling or deleting a key or of revoking a grant is not immediate\. It might take some time for the member and peer node resources to discover that the key is inaccessible\. When a resource is in this state, we recommend deleting and recreating the resource, and reconfiguring the KMS key\. To check the encryption status of a member or peer node resource, use the [GetMember](https://docs.aws.amazon.com/managed-blockchain/latest/APIReference/API_GetMember.html) or [GetNode](https://docs.aws.amazon.com/managed-blockchain/latest/APIReference/API_GetNode.html) API operation respectively\.

## How Managed Blockchain Uses Grants in AWS KMS<a name="managed-blockchain-encryption-at-rest.grants"></a>

Managed Blockchain requires *grants* to use your customer managed key\. When you create a member that is protected with a customer managed key, Managed Blockchain creates grants on your behalf by sending [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) requests to AWS KMS\. Grants in AWS KMS are used to give Managed Blockchain access to a KMS key in a customer AWS account\. For more information, see [Using Grants](https://docs.aws.amazon.com/kms/latest/developerguide/grants.html) in the *AWS Key Management Service Developer Guide*\.

Managed Blockchain requires the grants to use your customer managed key for the following API operations:
+ [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)
+ [DescribeKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_DescribeKey.html)
+ [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html)
+ [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html)
+ [GenerateDataKeyWithoutPlaintext](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKeyWithoutPlaintext.html)
+ [ReEncryptFrom](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)
+ [ReEncryptTo](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html)

Managed Blockchain uses Amazon Elastic Compute Cloud \(Amazon EC2\), Amazon Elastic Block Store \(Amazon EBS\), and AWS Secrets Manager to call these AWS KMS operations on its behalf\.

You can revoke a grant to remove the service's access to the customer managed key at any time\. If you do, the key becomes inaccessible, and Managed Blockchain loses access to any of the data protected by the customer managed key\. In this state, the member and its peer node resources are impaired and might not function as expected\. When a resource is in this state, we recommend deleting and recreating the resource, and reconfiguring the KMS key\.

## Encryption at Rest Considerations<a name="managed-blockchain-encryption-at-rest.considerations"></a>

Consider the following when you are using encryption at rest in Hyperledger Fabric on Managed Blockchain\.
+ Server\-side encryption at rest is enabled on all Managed Blockchain network, member, and peer node data and can't be disabled\. You can't encrypt only a subset of data in these resource types\.
+ Encryption at rest only encrypts data while it is static \(at rest\) on a persistent storage media\. If data security is a concern for data in transit or data in use, you might need to take additional measures as follows:
  + *Data in transit*: All your data in Managed Blockchain is encrypted in transit\. By default, communications to and from Managed Blockchain use the HTTPS protocol, which protects network traffic by using Secure Sockets Layer \(SSL\)/Transport Layer Security \(TLS\) encryption\.
  + *Data in use*: Use client\-side encryption to protect your data before sending it to Managed Blockchain\.
+ Managed Blockchain currently doesn't support encryption context for AWS KMS cryptographic operations\.

## Using Customer Managed Keys in Managed Blockchain<a name="managed-blockchain-encryption-at-rest.using-cust-keys"></a>

You can use the Managed Blockchain console, the Managed Blockchain API, or the AWS CLI to specify the AWS KMS key for new members in Managed Blockchain\. The following topics describe how to manage and monitor the usage of your customer managed keys in Managed Blockchain\.

**Topics**
+ [Prerequisites](#managed-blockchain-encryption-at-rest.using-cust-keys.prereqs)
+ [Specifying a Customer Managed Key](#managed-blockchain-encryption-at-rest.using-cust-keys.specify)
+ [Monitoring Your Customer Managed Keys](#managed-blockchain-encryption-at-rest.using-cust-keys.monitor)

### Prerequisites<a name="managed-blockchain-encryption-at-rest.using-cust-keys.prereqs"></a>

Before you can protect Managed Blockchain resources with a customer managed key, you must first create the key in AWS KMS\. You must also specify a key policy that allows Managed Blockchain to create grants on that AWS KMS key on your behalf\.

**Creating a Customer Managed Key**  
To create a customer managed key, follow the steps in [Creating symmetric customer managed keys](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html#create-symmetric-cmk) in the *AWS Key Management Service Developer Guide*\. Managed Blockchain doesn't support [asymmetric keys](https://docs.aws.amazon.com/kms/latest/developerguide/symmetric-asymmetric.html)\.

**Setting a Key Policy**  
Key policies are the primary way to control access to customer managed keys in AWS KMS\. Every customer managed key must have exactly one key policy\. The statements in the key policy document determine who has permission to use the AWS KMS key and how they can use it\. For more information, see [Using key policies in AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html) in the *AWS Key Management Service Developer Guide*\.  
You can specify a key policy when you create your customer managed key\. To change a key policy for an existing customer managed key, see [Changing a key policy](https://docs.aws.amazon.com/kms/latest/developerguide/key-policy-modifying.html)\.  
To allow Managed Blockchain to use your customer managed key, the key policy must include permissions for the [CreateGrant](https://docs.aws.amazon.com/kms/latest/APIReference/API_CreateGrant.html) API operation\. This operation adds a [grant](https://docs.aws.amazon.com/kms/latest/developerguide/grants.html) to a customer managed key\. Grants control access to a specified KMS key\. Managed Blockchain creates grants that allow access to the [grant operations](https://docs.aws.amazon.com/kms/latest/developerguide/grants.html#terms-grant-operations) that it requires, which are listed in [How Managed Blockchain Uses Grants in AWS KMS](#managed-blockchain-encryption-at-rest.grants)\.  
**Key Policy Example**  
The following is a key policy example that you can use for Managed Blockchain\. This policy allows principals that are authorized to use Managed Blockchain from the account `111122223333` to call the `CreateGrant` operation on all resources\.  
To use this policy, replace *aws\-region* and *111122223333* in the example with your own information\.  

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid" : "Allow access to principals authorized to use Amazon Managed Blockchain",
      "Effect" : "Allow",
      "Principal" : {
        "AWS" : "*"
      },
      "Action" : [
        "kms:CreateGrant"
      ],
      "Resource" : "*",
      "Condition" : {
        "StringEquals" : {
          "kms:ViaService" : "managedblockchain.aws-region.amazonaws.com",
          "kms:CallerAccount" : "111122223333"
        }
      }
    }
  ]
}
```

### Specifying a Customer Managed Key<a name="managed-blockchain-encryption-at-rest.using-cust-keys.specify"></a>

You can specify the Amazon Resource Name \(ARN\) of a customer managed KMS key when you create a new member by using the Managed Blockchain console, the Managed Blockchain API, or the AWS CLI\. The following is an example of a KMS key ARN\.

```
arn:aws:kms:us-east-1:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab
```

For more information, see the [MemberConfiguration](https://docs.aws.amazon.com/managed-blockchain/latest/APIReference/API_MemberConfiguration.html) data type in the *Amazon Managed Blockchain API Reference*\. You can pass this data type as an input parameter to the [CreateMember](https://docs.aws.amazon.com/managed-blockchain/latest/APIReference/API_CreateMember.html) or [CreateNetwork](https://docs.aws.amazon.com/managed-blockchain/latest/APIReference/API_CreateNetwork.html) API operations\.

### Monitoring Your Customer Managed Keys<a name="managed-blockchain-encryption-at-rest.using-cust-keys.monitor"></a>

If you use a customer managed key to protect your Amazon Managed Blockchain resources, you can use [AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html) or [Amazon CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html) to track the requests that Managed Blockchain sends to AWS KMS on your behalf\. For more information, see [Monitoring customer master keys](https://docs.aws.amazon.com/kms/latest/developerguide/monitoring-overview.html) in the *AWS Key Management Service Developer Guide*\.

The following example is a CloudTrail log entry for the `CreateGrant` API operation\. In addition to this event from Managed Blockchain, you can expect AWS KMS calls for your customer managed key from Amazon EC2, Amazon EBS, and AWS Secrets Manager\. These AWS services call other AWS KMS operations such as `GenerateDataKey`, `Decrypt`, and `Encrypt` on behalf of Managed Blockchain\.

**CreateGrant**  
When you specify a customer managed key to protect your member, Managed Blockchain sends `CreateGrant` requests to AWS KMS on your behalf to allow access to your KMS key\.  
The grants that Managed Blockchain creates are specific to a member and its peer nodes\. The principal in the `CreateGrant` request is the user who created the member or peer node\.  
The event that records the `CreateGrant` operation is similar to the following example event\. The parameters include the Amazon Resource Name \(ARN\) of the customer managed key, the grantee principal and retiring principal \(the Managed Blockchain service\), and the operations that the grant covers\.  

```
{
    "eventVersion": "1.08",
    "userIdentity": {
        "type": "IAMUser",
        "principalId": "AKIAIOSFODNN7EXAMPLE",
        "arn": "arn:aws:iam::111122223333:user/sample-user",
        "accountId": "111122223333",
        "accessKeyId": "AKIAI44QH8DHBEXAMPLE",
        "userName": "sample-user",
        "sessionContext": {
            "sessionIssuer": {},
            "webIdFederationData": {},
            "attributes": {
                "mfaAuthenticated": "false",
                "creationDate": "2021-06-08T16:36:03Z"
            }
        },
        "invokedBy": "managedblockchain.amazonaws.com"
    },
    "eventTime": "2021-06-08T16:36:04Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "CreateGrant",
    "awsRegion": "us-east-1",
    "sourceIPAddress": "managedblockchain.amazonaws.com",
    "userAgent": "managedblockchain.amazonaws.com",
    "requestParameters": {
        "keyId": "arn:aws:kms:us-east-1:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab",
        "granteePrincipal": "managedblockchain.us-east-1.amazonaws.com",
        "retiringPrincipal": "managedblockchain.us-east-1.amazonaws.com",
        "operations": [
            "Decrypt"
        ],
        "constraints": {
            "encryptionContextSubset": {}
        }
    },
    "responseElements": {
        "grantId": "64690d7d6ad23d39edd0c8fdea6400f297fae673ad84ce3563b6ca3a1e685ba2"
    },
    "requestID": "55b9f158-f537-43c2-acec-f336f41866b1",
    "eventID": "944aa59c-3bf9-4092-9ca2-435f585b41f6",
    "readOnly": false,
    "resources": [
        {
            "accountId": "111122223333",
            "type": "AWS::KMS::Key",
            "ARN": "arn:aws:kms:us-east-1:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
        }
    ],
    "eventType": "AwsApiCall",
    "managementEvent": true,
    "eventCategory": "Management",
    "recipientAccountId": "111122223333"
}
```