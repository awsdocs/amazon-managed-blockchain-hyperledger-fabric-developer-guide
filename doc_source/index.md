# Amazon Managed Blockchain Hyperledger Fabric Developer Guide

-----
*****Copyright &copy; 2020 Amazon Web Services, Inc. and/or its affiliates. All rights reserved.*****

-----
Amazon's trademarks and trade dress may not be used in 
     connection with any product or service that is not Amazon's, 
     in any manner that is likely to cause confusion among customers, 
     or in any manner that disparages or discredits Amazon. All other 
     trademarks not owned by Amazon are the property of their respective
     owners, who may or may not be affiliated with, connected to, or 
     sponsored by Amazon.

-----
## Contents
+ [What Is Amazon Managed Blockchain?](what-is-managed-blockchain.md)
+ [Key Concepts: Amazon Managed Blockchain Networks, Members, and Peer Nodes](network-components.md)
+ [Get Started Creating a Hyperledger Fabric Blockchain Network Using Amazon Managed Blockchain](managed-blockchain-get-started-tutorial.md)
   + [Prerequisites and Considerations](get-started-prerequisites.md)
   + [Step 1: Create the Network and First Member](get-started-create-network.md)
   + [Step 2: Create and Configure the Interface VPC Endpoint](get-started-create-endpoint.md)
   + [Step 3: Create an Amazon EC2 Instance and Set Up the Hyperledger Fabric Client](get-started-create-client.md)
   + [Step 4: Enroll an Administrative User](get-started-enroll-admin.md)
   + [Step 5: Create a Peer Node in Your Membership](get-started-create-peer-node.md)
   + [Step 6: Create a Hyperledger Fabric Channel](get-started-create-channel.md)
   + [Step 7: Install and Run Chaincode](get-started-chaincode.md)
   + [Step 8: Invite Another AWS Account to be a Member and Create a Multi-Member Channel](get-started-joint-channel.md)
+ [Create a Hyperledger Fabric Blockchain Network on Amazon Managed Blockchain](create-network.md)
+ [Delete a Hyperledger Fabric Network on Amazon Managed Blockchain](delete-network.md)
+ [Invite or Remove Hyperledger Fabric Network Members on Amazon Managed Blockchain](managed-blockchain-members.md)
+ [Accept an Invitation to Create a Member and Join a Hyperledger Fabric Network on Amazon Managed Blockchain](managed-blockchain-hyperledger-member.md)
   + [Work with Invitations](accept-invitation.md)
   + [Create a Member and Join a Network](managed-blockchain-hyperledger-create-member.md)
+ [Create an Interface VPC Endpoint for Hyperledger Fabric on Amazon Managed Blockchain](managed-blockchain-endpoints.md)
+ [Work with Hyperledger Fabric Peer Nodes on Managed Blockchain](managed-blockchain-hyperledger-peer-nodes.md)
   + [Create a Hyperledger Fabric Peer Node on Amazon Managed Blockchain](managed-blockchain-create-peer-node.md)
   + [View Hyperledger Fabric Peer Node Properties on Amazon Managed Blockchain](managed-blockchain-view-peer-node.md)
   + [Use Hyperledger Fabric Peer Node Metrics on Amazon Managed Blockchain](managed-blockchain-peer-node-metrics.md)
+ [Work with Proposals for a Hyperledger Fabric Network on Amazon Managed Blockchain](managed-blockchain-proposals.md)
   + [Automating Managed Blockchain Proposal Notifications with CloudWatch Events](automating-proposals-with-cloudwatch-events.md)
+ [Work with Hyperledger Fabric Components and Chaincode](framework-client.md)
   + [Register and Enroll a Hyperledger Fabric Admin](managed-blockchain-hyperledger-create-admin.md)
   + [Work with Channels](hyperledger-work-with-channels.md)
   + [Add an Anchor Peer to a Channel](hyperledger-anchor-peers.md)
   + [Develop Hyperledger Fabric Chaincode](managed-blockchain-hyperledger-develop-chaincode.md)
      + [Work with Private Data Collections in Hyperledger Fabric on Managed Blockchain](managed-blockchain-hyperledger-create-pdc.md)
      + [Develop Hyperledger Fabric Chaincode Using Java on Amazon Managed Blockchain](java-chaincode.md)
   + [Query Chaincode Data in the State Database](hyperledger-couchdb.md)
+ [Hyperledger Fabric on Amazon Managed Blockchain Security](managed-blockchain-security.md)
   + [Data Protection for Hyperledger Fabric on Amazon Managed Blockchain](managed-blockchain-data-protection.md)
   + [Authentication and Access Control for Hyperledger Fabric on Managed Blockchain](managed-blockchain-auth-and-access-control.md)
   + [Identity and Access Management for Hyperledger Fabric on Amazon Managed Blockchain](security-iam.md)
      + [How Hyperledger Fabric on Amazon Managed Blockchain works with IAM](security_iam_service-with-iam.md)
      + [Hyperledger Fabric on Amazon Managed Blockchain Identity-Based Policy Examples](security_iam_id-based-policy-examples.md)
      + [Example IAM Role Permissions Policy for Hyperledger Fabric Client EC2 Instance](security_iam_hyperledger_ec2_client.md)
      + [Using Service-Linked Roles for Managed Blockchain](using-service-linked-roles.md)
      + [Troubleshooting Hyperledger Fabric on Amazon Managed Blockchain identity and access](security_iam_troubleshoot.md)
   + [Configuring Security Groups for Hyperledger Fabric on Amazon Managed Blockchain](managed-blockchain-security-sgs.md)
+ [Tagging Amazon Managed Blockchain resources](tagging.md)
+ [Monitoring Hyperledger Fabric on Managed Blockchain Using CloudWatch Logs](monitoring-cloudwatch-logs.md)
+ [Logging Amazon Managed Blockchain API calls using AWS CloudTrail](logging-using-cloudtrail.md)
+ [Document History for Hyperledger Fabric on Amazon Managed Blockchain Developer Guide](managed-blockchain-management-guide-doc-history.md)
+ [AWS glossary](glossary.md)