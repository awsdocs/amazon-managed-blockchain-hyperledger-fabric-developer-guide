# What Is Amazon Managed Blockchain?<a name="what-is-managed-blockchain"></a>

Amazon Managed Blockchain is a fully managed service for creating and managing blockchain networks and network resources using open\-source frameworks\. Blockchain allows you to build applications where multiple parties can securely and transparently run transactions and share data without the need for a trusted, central authority\.

You can use Managed Blockchain to create scalable blockchain resources and networks quickly and efficiently using the AWS Management Console, the AWS CLI, or the Managed Blockchain SDK\. 

Managed Blockchain scales to meet the demands of thousands of applications running millions of transactions\. Managed Blockchain also simplifies the management of blockchain networks and resources after they are up and running\. Managed Blockchain manages your certificates, lets you easily create proposals for a vote among network members where applicable, and helps you track operational metrics related to requests, computational load, memory usage, and data storage\.

This guide covers the fundamentals of creating and working with a Hyperledger Fabric blockchain network using Managed Blockchain\. For information about working with Ethereum on Managed Blockchain, see [Ethereum on Amazon Managed Blockchain Developer Guide](https://docs.aws.amazon.com/managed-blockchain/latest/ethereum-dev/)\.

## How to Get Started with Hyperledger Fabric on Managed Blockchain<a name="how-to-start"></a>

We recommend the following resources to get started with Hyperledger Fabric networks and chaincode on Managed Blockchain:
+ [Key Concepts: Amazon Managed Blockchain Networks, Members, and Peer Nodes](network-components.md)

  This overview helps you understand the fundamental building blocks of a Hyperledger Fabric network on Managed Blockchain\. It also tells you how to identify and communicate with network resources\.
+ [Get Started Creating a Hyperledger Fabric Blockchain Network Using Amazon Managed Blockchain](managed-blockchain-get-started-tutorial.md)

  Use this tutorial to create your first Hyperledger Fabric network, set up a Hyperledger Fabric client on EC2, and use the open\-source Hyperledger Fabric peer CLI to query and update the ledger\. You then invite another member to the network\. The member can be from a different AWS account, or you can invite a new member in your own account to simulate a multi\-account network\. The new member then queries and updates the ledger\.
+ [Hyperledger Fabric Documentation \(v1\.4\)](https://hyperledger-fabric.readthedocs.io/en/release-1.4/)

  The open\-source documentation for Hyperledger Fabric is a starting point for key concepts and the architecture of the Hyperledger Fabric blockchain network that you build using Managed Blockchain\. As you develop your blockchain application, you can reference this document for key tasks and code samples\. Use the documentation version that corresponds to the version of Hyperledger Fabric that you use\.