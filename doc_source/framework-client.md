# Work with Hyperledger Fabric Components and Chaincode<a name="framework-client"></a>

You access services and applications in Hyperledger Fabric on Managed Blockchain from a framework client machine\. The client runs tools and applications that you install for the version of Hyperledger Fabric on the network\.

The client accesses Managed Blockchain endpoints for components such as the CA, the orderer, and peer nodes through an interface VPC endpoint that you set up in your AWS account\. The client machine must have access to the interface VPC endpoint to access resource endpoints\. For more information, see [Create an Interface VPC Endpoint for Hyperledger Fabric on Amazon Managed Blockchain](managed-blockchain-endpoints.md)\.

You can get the endpoints that networks, members, and peer nodes make available using the AWS Management Console, or using `get` commands and actions with the Managed Blockchain AWS CLI or SDK\.

An AWS CloudFormation template to create a Hyperledger Fabric client is available in [amazon\-managed\-blockchain\-client\-templates repository](https://github.com/awslabs/amazon-managed-blockchain-client-templates) on Github\. For more information, see the [readme\.md](https://github.com/awslabs/amazon-managed-blockchain-client-templates/blob/master/README.md) in that repository\. For more information about using AWS CloudFormation, see [Getting Started](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/GettingStarted.Walkthrough.html) in the *AWS CloudFormation User Guide*\.

**Topics**
+ [Register and Enroll a Hyperledger Fabric Admin](managed-blockchain-hyperledger-create-admin.md)
+ [Work with Channels](hyperledger-work-with-channels.md)
+ [Add an Anchor Peer to a Channel](hyperledger-anchor-peers.md)
+ [Develop Hyperledger Fabric Chaincode](managed-blockchain-hyperledger-develop-chaincode.md)
+ [Query Chaincode Data in the State Database](hyperledger-couchdb.md)