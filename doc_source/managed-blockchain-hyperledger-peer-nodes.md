# Work with Hyperledger Fabric Peer Nodes on Managed Blockchain<a name="managed-blockchain-hyperledger-peer-nodes"></a>

Hyperledger Fabric peer nodes do the work for your member on the network\. They keep a local copy of the shared ledger, let you query the ledger, and interact with clients and other peer nodes to perform transactions\. A new member has no peer nodes\. Create at least one peer node per member\.

Each peer node runs on a *Managed Blockchain instance type*\. You cannot add a custom Amazon EC2 instance to your member, nor can you connect an on\-premises machine\. The number of peer nodes and the Managed Blockchain instance type of peer nodes available to each member depends on the network edition specified when the network was created\. For more information, see [Amazon Managed Blockchain Pricing](https://aws.amazon.com/managed-blockchain/pricing)\.

When you create a peer node, you select the following characteristics:
+ **Managed Blockchain instance type**

  This determines the computational and memory capacity allocated to this node for the blockchain workload\. You can choose more CPU and RAM if you anticipate a more demanding workload for each node\. For example, your nodes may need to process a higher rate of transactions\. Different instance types are subject to different pricing\. You can monitor CPU and memory utilization to determine if your Managed Blockchain instance type is appropriate\. For more information, see [Use Hyperledger Fabric Peer Node Metrics on Amazon Managed Blockchain](managed-blockchain-peer-node-metrics.md)
+ **Availability Zone**

  You can select the Availability Zone to launch the peer node in\. The ability to distribute peer nodes in a member across different Availability Zones allows you to design your blockchain application for resiliency\. For more information, see [Regions and Availability Zones](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html) in the *Amazon EC2 User Guide for Linux Instances*\.

  
+ **State DB configuration**

  This selection is only available for peer nodes running on networks using Hyperledger Fabric 1\.4 and later\. This determines the type of state database that the peer node uses\. **LevelDB** stores chaincode data as simple key\-value pairs\. It supports only key, key range, and composite key queries\. **CouchDB** models data in the ledger as JSON\. It supports richer queries and indexing for efficiency\. CouchDB is the default for networks running Hyperledger Fabric 1\.4 and later\. Hyperledger Fabric 1\.2 uses only LevelDB\. For more information, see [Query Chaincode Data in the State Database](hyperledger-couchdb.md) and [CouchDB as the State Database](https://hyperledger-fabric.readthedocs.io/en/release-1.4/couchdb_as_state_database.html) in Hyperledger Fabric documentation\.
+ **Logging configuration**

  You can enable peer node logs, chaincode logs, or both for a member\. Peer node logs allow you to debug timeout errors associated with proposals and identify rejected proposals\. Chaincode logs help you analyze and debug business logic\. For more information, see [Monitoring Hyperledger Fabric on Managed Blockchain Using CloudWatch Logs](monitoring-cloudwatch-logs.md)\.