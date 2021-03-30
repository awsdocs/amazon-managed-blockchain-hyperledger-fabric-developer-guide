# Hyperledger Fabric on Amazon Managed Blockchain Identity\-Based Policy Examples<a name="security_iam_id-based-policy-examples"></a>

By default, IAM users and roles don't have permission to create or modify Managed Blockchain resources\. They also can't perform tasks using the AWS Management Console, AWS CLI, or AWS API\. An IAM administrator must create IAM policies that grant users and roles permission to perform specific API operations on the specified resources they need\. The administrator must then attach those policies to the IAM users or groups that require those permissions\.

To learn how to create an IAM identity\-based policy using these example JSON policy documents, see [Creating Policies on the JSON Tab](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html#access_policies_create-json-editor) in the *IAM User Guide*\.

**Topics**
+ [Policy Best Practices](#security_iam_service-with-iam-policy-best-practices)
+ [Allow Users to View Their Own Permissions](#security_iam_id-based-policy-examples-view-own-permissions)
+ [Using the Managed Blockchain Console](#security_iam_id-based-policy-examples-console)
+ [Performing All Managed Blockchain Actions on All Accessible Networks for an AWS Account](#security_iam_id-based-policy-examples-access-one-bucket)
+ [Controlling access using tags](#security_iam_id-based-policy-examples-tags)

## Policy Best Practices<a name="security_iam_service-with-iam-policy-best-practices"></a>

Identity\-based policies are very powerful\. They determine whether someone can create, access, or delete Managed Blockchain resources in your account\. These actions can incur costs for your AWS account\. When you create or edit identity\-based policies, follow these guidelines and recommendations:
+ **Get started using AWS managed policies** – To start using Managed Blockchain quickly, use AWS managed policies to give your employees the permissions they need\. These policies are already available in your account and are maintained and updated by AWS\. For more information, see [Get started using permissions with AWS managed policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#bp-use-aws-defined-policies) in the *IAM User Guide*\.
+ **Grant least privilege** – When you create custom policies, grant only the permissions required to perform a task\. Start with a minimum set of permissions and grant additional permissions as necessary\. Doing so is more secure than starting with permissions that are too lenient and then trying to tighten them later\. For more information, see [Grant least privilege](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege) in the *IAM User Guide*\.
+ **Enable MFA for sensitive operations** – For extra security, require IAM users to use multi\-factor authentication \(MFA\) to access sensitive resources or API operations\. For more information, see [Using multi\-factor authentication \(MFA\) in AWS](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html) in the *IAM User Guide*\.
+ **Use policy conditions for extra security** – To the extent that it's practical, define the conditions under which your identity\-based policies allow access to a resource\. For example, you can write conditions to specify a range of allowable IP addresses that a request must come from\. You can also write conditions to allow requests only within a specified date or time range, or to require the use of SSL or MFA\. For more information, see [IAM JSON policy elements: Condition](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition.html) in the *IAM User Guide*\.

## Allow Users to View Their Own Permissions<a name="security_iam_id-based-policy-examples-view-own-permissions"></a>

This example shows how you might create a policy that allows IAM users to view the inline and managed policies that are attached to their user identity\. This policy includes permissions to complete this action on the console or programmatically using the AWS CLI or AWS API\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ViewOwnUserInfo",
            "Effect": "Allow",
            "Action": [
                "iam:GetUserPolicy",
                "iam:ListGroupsForUser",
                "iam:ListAttachedUserPolicies",
                "iam:ListUserPolicies",
                "iam:GetUser"
            ],
            "Resource": ["arn:aws:iam::*:user/${aws:username}"]
        },
        {
            "Sid": "NavigateInConsole",
            "Effect": "Allow",
            "Action": [
                "iam:GetGroupPolicy",
                "iam:GetPolicyVersion",
                "iam:GetPolicy",
                "iam:ListAttachedGroupPolicies",
                "iam:ListGroupPolicies",
                "iam:ListPolicyVersions",
                "iam:ListPolicies",
                "iam:ListUsers"
            ],
            "Resource": "*"
        }
    ]
}
```

## Using the Managed Blockchain Console<a name="security_iam_id-based-policy-examples-console"></a>

To access the Amazon Managed Blockchain console, you must have a minimum set of permissions\. These permissions must allow you to list and view details about the Managed Blockchain resources in your AWS account\. If you create an identity\-based policy that is more restrictive than the minimum required permissions, the console won't function as intended for entities \(IAM users or roles\) with that policy\.

To ensure that those entities can still use the Managed Blockchain console, also attach the following AWS managed policy to the entities\.

```
AmazonManagedBlockchainConsoleFullAccess
```

For more information, see [Adding Permissions to a User](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_change-permissions.html#users_change_permissions-add-console) in the *IAM User Guide*\.

You don't need to allow minimum console permissions for users that are making calls only to the AWS CLI or the AWS API\. Instead, allow access to only the actions that match the API operation that you're trying to perform\.

## Performing All Managed Blockchain Actions on All Accessible Networks for an AWS Account<a name="security_iam_id-based-policy-examples-access-one-bucket"></a>

This example grants an IAM user in your AWS account access to all network and member resources in your account in the `us-east-1` Region for the AWS account `123456789012`\. This includes the ability to create new networks, reject invitations to join other networks, and join other networks by creating a member\.

```
{
	"Version": "2012-10-17",
	"Statement": [{
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
				"arn:aws:managedblockchain:us-east-1::networks/*",
				"arn:aws:managedblockchain:us-east-1::proposals/*",
				"arn:aws:managedblockchain:us-east-1:123456789012:members/*",
				"arn:aws:managedblockchain:us-east-1:123456789012:invitations/*",
				"arn:aws:managedblockchain:us-east-1:123456789012:nodes/*"
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

## Controlling access using tags<a name="security_iam_id-based-policy-examples-tags"></a>

The following example policy statements demonstrate how you can use tags to limit access to Hyperledger Fabric on Managed Blockchain resources and actions performed on those resources\.

To tag resources during creation, policy statements that allow the create action for the resource as well as the `TagResource` action must be attached to the IAM principal performing the operation\.

**Note**  
This topic includes examples of policy statements with a `Deny` effect\. Each policy statement assumes that a statement with a broader `Allow` effect for the same actions exists; the `Deny` policy statement restricts that otherwise overly\-permissive allow statement\.

**Example – Require a tag key with either of two values to be added during network creation**  
The following identity\-based policy statements allow the IAM principal to create a network only if the network is created with the tags specified\.  
The statement with the `Sid` set to `RequireTag` specifies that network creation is allowed only if the network is created with a tag that has a tag key of `department` and a value of either `sales` or `marketing`; otherwise, the create network operation fails\.  
The statement with `Sid` set to `AllowTagging` allows the IAM principal to tag networks, which are not associated with an AWS account ID\. It also allows tagging of members in the specified AWS account, `111122223333`\.  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "RequireTag",
            "Effect": "Allow",
            "Action": [
                "managedblockchain:CreateNetwork"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringLike": {
                    "aws:RequestTag/department": [
                        "sales",
                        "marketing"
                    ]
                }
            }
        },
        {
            "Sid": "AllowTagging",
            "Effect": "Allow",
            "Action": [
                "managedblockchain:TagResource"
            ],
            "Resource": [
                "arn:aws:managedblockchain:us-east-1::networks/*",
                "arn:aws:managedblockchain:us-east-1:111122223333:members/*"
            ]
        }
    ]
}
```

**Example – Deny access to networks that have a specific tag key**  
The following identity\-based policy statement denies the IAM principal the ability to retrieve or view information for networks that have a tag with a tag key of `restricted`\.  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyRestrictedNetwork",
            "Effect": "Deny",
            "Action": [
                "managedblockchain:GetNetwork"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringLike": {
                    "aws:ResourceTag/restricted": [
                        "*"
                    ]
                }
            }
        }
    ]
}
```

**Example – Deny member creation for networks with a specific tag key and value**  
The following identity\-based policy statement denies the IAM principal from creating a member on any network that has a tag with a tag key of `department` with the value of `accounting`\.  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyMemberCreation",
            "Effect": "Deny",
            "Action": [
                "managedblockchain:CreateMember"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:ResourceTag/department": "accounting"
                }
            }
        }
    ]
}
```

**Example – Allow member creation only if a specific tag key and value are added**  
The following identity\-based policy statements allow the IAM principal to create members in the account `111122223333` only if the member is created with a tag that has the tag key of `department` and a tag value of `accounting`; otherwise, the create member operation fails\.  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "RestrictMemberCreation",
            "Effect": "Allow",
            "Action": [
                "managedblockchain:CreateMember"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:RequestTag/department": "accounting"
                }
            }
        },
        {
            "Sid": "AllowTaggingOfMembers",
            "Effect": "Allow",
            "Action": [
                "managedblockchain:TagResource"
            ],
            "Resource": [
                "arn:aws:managedblockchain:us-east-1:111122223333:members/*"
            ]
        }
    ]
}
```

**Example – Allow access only to members that have a specified tag key**  
The following identity\-based policy statement allows the IAM principal to retrieve or view information about members only if the member has a tag with a tag key of `unrestricted`\.  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "",
            "Effect": "Allow",
            "Action": [
                "managedblockchain:GetMember"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringLike": {
                    "aws:ResourceTag/unrestricted": [
                        "*"
                    ]
                }
            }
        }
    ]
}
```

**Example – Deny node creation if a specific tag key and value are added during creation**  
The following identity\-based policy statement denies the IAM principal the ability to create a node if a tag with the tag key of `department` and a value of `analytics` is added during creation\.  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyCreateNodeWithTag",
            "Effect": "Deny",
            "Action": [
                "managedblockchain:CreateNode"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:RequestTag/department": [
                        "analytics"
                    ]
                }
            }
        }
    ]
}
```

**Example – Deny node creation for members with a specific tag key and value**  
The following identity\-based policy statement denies the IAM principal the ability to create a node if the member to which the node will belong has a tag with a tag key of `department` and a tag value of `analytics`\.  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyCreateNodeForTaggedMember",
            "Effect": "Deny",
            "Action": [
                "managedblockchain:CreateNode"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:ResourceTag/department": [
                        "analytics"
                    ]
                }
            }
        }
    ]
}
```

**Example – Allow access only to nodes with a specific tag key and value**  
The following identity\-based policy statement allows the IAM principal to retrieve or view information about nodes only if the node has a tag with a tag key of `department` and a tag value of `sales`\.  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowTaggedNodeAccess",
            "Effect": "Allow",
            "Action": [
                "managedblockchain:GetNode"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:ResourceTag/department": [
                        "sales"
                    ]
                }
            }
        }
    ]
}
```

**Example – Deny access to nodes with a specific tag key**  
The following identity\-based policy statement denies the IAM principal the ability to retrieve or view information about a node if the node has a tag with a tag key of `restricted`\.  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyAccessToTaggedNodes",
            "Effect": "Deny",
            "Action": [
                "managedblockchain:GetNode"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringLike": {
                    "aws:ResourceTag/restricted": [
                        "*"
                    ]
                }
            }
        }
    ]
}
```

**Example – Allow proposal creation only for networks with a specific tag key and value**  
The following identity\-based policy statement allows the IAM principal to create a proposal only if the network for which the proposal is made has a tag with a tag key of `department` and a value of `accounting`\.  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowCreateProposalOnlyForTaggedNetwork",
            "Effect": "Allow",
            "Action": [
                "managedblockchain:CreateProposal"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:ResourceTag/department": [
                        "accounting"
                    ]
                }
            }
        }
    ]
}
```

**Example – Allow proposal creation only if a specific tag key and value are added**  
The following identity\-based policy statements allow the IAM principal to create a proposal for inviting the specified account `123456789012` to the network only if a tag with a tag key of `consortium` and a tag value of `exampleconsortium` is added during proposal creation\.  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "RequireTagWithProposal",
            "Effect": "Allow",
            "Action": [
                "managedblockchain:CreateProposal"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:RequestTag/consortium": [
                        "exampleconsortium"
                    ]
                }
            }
        },
        {
            "Sid": "AllowProposalAndInvitationTagging",
            "Effect": "Allow",
            "Action": [
                "managedblockchain:TagResource"
            ],
            "Resource": [
                "arn:aws:managedblockchain:us-east-1::proposals/*",
                "arn:aws:managedblockchain:us-east-1:123456789012:invitations/*"
            ]
        }
    ]
}
```

**Example – Allow access only to proposals with a specific tag key and value**  
The following identity\-based policy statement allows the IAM principal to retrieve and view proposals only if the proposal has a tag with a tag key of `consortium` and a tag value of `exampleconsortium`\.  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowOnlyTaggedProposalAccess",
            "Effect": "Allow",
            "Action": [
                "managedblockchain:GetProposal"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:ResourceTag/consortium": [
                        "exampleconsortium"
                    ]
                }
            }
        }
    ]
}
```

**Example – Deny the ability to vote on proposals based on a specific tag key and value**  
The following identity\-based policy statement denies the IAM principal the ability to vote on proposals that have a tag with the tag key of `consortium` and a tag value of `exampleconsortium`\.  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyVoteOnTaggedProposal",
            "Effect": "Deny",
            "Action": [
                "managedblockchain:VoteOnProposal"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:ResourceTag/consortium": [
                        "exampleconsortium"
                    ]
                }
            }
        }
    ]
}
```

**Example – Deny the ability to reject an invitation with a specific tag and value**  
The following identity\-based policy statement denies the IAM principal the ability to reject on invitation that has a tag with the tag key of `consortium` and a tag value of `exampleconsortium`\.  

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "1",
            "Effect": "Deny",
            "Action": [
                "managedblockchain:RejectInvitation"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEquals": {
                    "aws:ResourceTag/consortium": [
                        "exampleconsortium"
                    ]
                }
            }
        }
    ]
}
```