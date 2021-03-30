# Query Chaincode Data in the State Database<a name="hyperledger-couchdb"></a>

The [state database](https://hyperledger-fabric.readthedocs.io/en/release-1.4/ledger/ledger.html#world-state) in Hyperledger Fabric peer nodes on Managed Blockchain networks created using Hyperledger Fabric 1\.4 and later supports two types of peer state databases:
+ **CouchDB** is a state database in Managed Blockchain that models ledger data as JSON\. It is the default peer state database for Hyperledger Fabric 1\.4 or later network on Managed Blockchain because CouchDB supports rich queries and indexing for more efficient queries over large datasets\.
+ **LevelDB** is a state database that stores ledger data as simple key\-value pairs\. It is the only peer state database available for Hyperledger Fabric 1\.2 networks on Managed Blockchain\. LevelDB supports only key, key range, and composite key queries\.

For detailed information about CouchDB implementation and capabilities in Hyperledger Fabric, see [CouchDB as the State Database](https://hyperledger-fabric.readthedocs.io/en/release-1.4/couchdb_as_state_database.html) in Hyperledger Fabric documentation\.

## Specifying and Viewing the State Database Type<a name="hyperledger-couchdb-specify"></a>

You specify the peer database type when you create a peer node for a member using the AWS Management Console, the AWS CLI, or the Managed Blockchain API\. For more information, see [Create a Hyperledger Fabric Peer Node on Amazon Managed Blockchain](managed-blockchain-create-peer-node.md) and [CreateNode](https://docs.aws.amazon.com/managed-blockchain/latest/APIReference/API_CreateNode.html) in the *Amazon Managed Blockchain API Reference*\. You can't change the state database type after a node has been created\.

The type of state database that a peer node uses is available on the peer node properties page in the AWS Management Console\. It is also returned by the AWS CLI `managedblockchain get-node` command\. For more information, see [View Hyperledger Fabric Peer Node Properties on Amazon Managed Blockchain](managed-blockchain-view-peer-node.md)\.

## Rich Queries With CouchDB<a name="hyperledger-couchdb-examples"></a>

The following examples demonstrate rich queries against a CouchDB state database using the `peer chaincode query` command of the Hyperledger Fabric CLI in Managed Blockchain\. The examples are based on the [marbles02 chaincode sample on GitHub](https://github.com/hyperledger/fabric-samples/tree/master/chaincode/marbles02/go)\. This example includes [implementations of rich query functionality within chaincode](https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02/go/marbles_chaincode.go#L496), including the code that supports the [queryMarbles](https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02/go/marbles_chaincode.go#L534) and the [queryMarblesWithPagination](https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02/go/marbles_chaincode.go#L634) functions passed as arguments in the examples\.

The marbles02 sample chaincode is available when you [set up a Hyperledger Fabric client](get-started-create-client.md) according to the instructions in the getting started tutorial of this guide\. Those instructions include a [step to clone the samples repository](get-started-create-client.md#get-started-client-clone-samples) that contains marbles02\.

For examples of basic queries that can be run when using either LevelDB or CouchDB, see the [query chaincode](get-started-chaincode.md#get-started-create-channel-query-chaincode) example in the getting started tutorial of this guide\.

Before querying the chaincode, `install` the chaincode, `instantiate` it, and `invoke` the chaincode to add three ledger records as shown in the following example commands\. For more information, see the [peer chaincode](https://hyperledger-fabric.readthedocs.io/en/release-1.4/commands/peerchaincode.html) command reference in Hyperledger Fabric documentation\.

```
docker exec cli peer chaincode install \
-n myrichquerycc -v v0 -p github.com/marbles02/go
```

```
docker exec cli peer chaincode instantiate \
-o $ORDERER -C mychannel -n myrichquerycc \
-v v0 -c '{"Args":["init"]}' \
--cafile /opt/home/managedblockchain-tls-chain.pem --tls
```

```
docker exec cli peer chaincode invoke \
-o $ORDERER -C mychannel -n myrichquerycc \
-c '{"Args":["initMarble","marble1","blue","10","tom"]}' \
--cafile /opt/home/managedblockchain-tls-chain.pem --tls
```

```
docker exec cli peer chaincode invoke \
-o $ORDERER -C mychannel -n myrichquerycc \
-c '{"Args":["initMarble","marble2","red","20","tom"]}' \
--cafile /opt/home/managedblockchain-tls-chain.pem --tls
```

```
docker exec cli peer chaincode invoke \
-o $ORDERER -C mychannel -n myrichquerycc \
-c '{"Args":["initMarble","marble3","green","30","tom"]}' \
--cafile /opt/home/managedblockchain-tls-chain.pem --tls
```

Now that the state database is populated with three records, use the rich query function `queryMarbles` defined within the marbles02 chaincode sample as shown in the following example\. The query returns only those marbles owned by `tom`\.

```
docker exec cli peer chaincode query -o $ORDERER \
-C mychannel -n myrichquerycc \
-c '{"Args":["queryMarbles","{\"selector\":{\"owner\":\"tom\"}}"]}' \
--cafile /opt/home/managedblockchain-tls-chain.pem --tls
```

The query example below uses the rich query function `queryMarblesWithPagination` defined within the marbles02 chaincode sample\. The query is similar to the previous rich query\. However, this function allows you to specify the number of records to return \(the page size\) and an optional bookmark\. The page size specified in the example is `1` and the bookmark argument is empty\.

```
docker exec cli peer chaincode query \
-o $ORDERER -C mychannel -n myrichquerycc \
-c '{"Args":["queryMarblesWithPagination","{\"selector\":{\"owner\":\"tom\"}}","1",""]}' \
--cafile /opt/home/managedblockchain-tls-chain.pem --tls
```