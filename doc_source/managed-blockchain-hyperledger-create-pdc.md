# Work with Private Data Collections in Hyperledger Fabric on Managed Blockchain<a name="managed-blockchain-hyperledger-create-pdc"></a>

Network members that participate on a Hyperledger Fabric channel on Managed Blockchain may want to keep some data private from other members on the same channel\. Hyperledger Fabric *private data collections* allow you to accomplish this\. Private data collections allow a subset of members on a channel to endorse, commit, and query private data without having to create a separate channel to exclude other members\. Private data reduces the overhead associated with channel creation and maintenance\. In addition, private data is disseminated only between authorized peers using the Hyperledger Fabric *gossip protocol*, so the data is kept confidential from the ordering service\.

To create a private data collection, a peer creates a *private data definition* during chaincode instantiation\. The definition establishes the members who have access to the private data collection\. It also defines a policy that governs how transactions are endorsed and committed\.

Private data is stored on each peer node in a way that is accessible to the peer chaincode and logically distinct from other channel transaction data\. This helps ensure that channel members who are not specified in the private data definition cannot access the private data\.

For more information about private data collections, see [Private Data](https://hyperledger-fabric.readthedocs.io/en/release-1.4/private-data/private-data.html#private-data) and the [Private Data architecture](https://hyperledger-fabric.readthedocs.io/en/release-1.4/private-data-arch.html) topic in Hyperledger Fabric documentation\.

Private data collections with Managed Blockchainlong require the following:
+ A network created using Hyperledger Fabric version 1\.4 or later\.
+ A channel configuration that supports the latest features of your version\. The [multi\-member channel configuration](get-started-joint-channel.md#get-started-joint-channel-channel-configtx) featured in the [Getting Started](managed-blockchain-get-started-tutorial.md) tutorial of this guide enables the latest features\.
+ At least one anchor peer for each member that participates in the private data collection\. This is because Hyperledger Fabric uses the gossip protocol to distribute private data to authorized peers\. For more information, see [Add an Anchor Peer to a Channel](hyperledger-anchor-peers.md)\.