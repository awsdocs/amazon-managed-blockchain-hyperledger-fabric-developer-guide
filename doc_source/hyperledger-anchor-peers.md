# Add an Anchor Peer to a Channel<a name="hyperledger-anchor-peers"></a>

Anchor peers within an organization enable cross\-organization communication among peer nodes using the Hyperledger Fabric gossip protocol\. In general, the gossip protocol identifies other available member peers, disseminates ledger data across peers on a channel, and brings new peers up to date quickly\.

Although anchor peers are not strictly required for an organization on a channel, at least one anchor peer is required for an organization to participate in features that use the gossip protocol, such as [private data collections](https://hyperledger-fabric.readthedocs.io/en/release-2.2/private-data-arch.html) and service discovery\. In addition, we recommend that an organization creates redundant anchor peers on a channel for resiliency\. For more information, see [Anchor Peer](https://hyperledger-fabric.readthedocs.io/en/release-2.2/glossary.html#anchor-peer) and [Gossip data dissemination protocol](https://hyperledger-fabric.readthedocs.io/en/release-2.2/gossip.html) in the Hyperledger Fabric documentation\.

The steps in this section demonstrate how you can update a channel configuration to add a peer as an anchor peer for your organization in Amazon Managed Blockchain\. The peer that you want to designate as an anchor peer for your organization must first be joined to the channel\.

## Prerequisites and Assumptions<a name="hyperledger-anchor-peer-prerequisites"></a>

The steps in this section are based on the assumption that you are using a network, client machine, member, peer, and joint channel similar to those that you create during the [getting started tutorial](managed-blockchain-get-started-tutorial.md) in this guide\.

The example commands shown presume that the environment variables, the directory locations on the client machine, the locations in the Docker container, the properties of members and peer nodes, and the properties of the joint channel from that tutorial are being used\. You might find it useful to review that information before running these commands\.

For more information, see the following resources:
+  [Create a client](get-started-create-client.md) 
+  [Enroll an administrator](get-started-enroll-admin.md) 
+  [Create the peer node](get-started-create-peer-node.md) 
+  [Create a multi\-member channel](get-started-joint-channel.md) 
+ [Join the peer node to the channel](get-started-joint-channel.md#get-started-joint-channel-invite-join-peer)

## Adding a Peer as an Anchor Peer<a name="hyperledger-anchor-peer-adding"></a>

A peer node must already be joined to a channel to be set up as an anchor peer\. To update a peer to be an anchor peer, you download a configuration file from the orderer on the Managed Blockchain network into a directory that you create on the Hyperledger Fabric client\. Next, you use the [configtxlator](https://hyperledger-fabric.readthedocs.io/en/release-2.2/commands/configtxlator.html) command and the [jq](https://stedolan.github.io/jq/manual/) tool to manipulate the original protobuf configuration file between protobuf and JSON\. Then you use your final protobuf file to update the channel configuration in the orderer\.

1. Create a directory on your Hyperledger Fabric client machine, *channel\-artifacts*, that you can use to save channel configuration artifacts that you create\.

   ```
   cd /home/ec2-user
   ```

   ```
   mkdir channel-artifacts
   ```

1. Use the Hyperledger Fabric [peer channel]() `fetch` sub\-command to get the channel configuration and save it locally, as shown in the following command\. In this example, *ourchannel* is the Channel ID\. This Channel ID is the same one established in the [step for creating the multi\-member channel](get-started-joint-channel.md#get-started-joint-channel-create-channel) in the [Getting Started](managed-blockchain-get-started-tutorial.md) tutorial of this guide\. The command saves the configuration file to the directory you created in a protobuf file named *ourchannelCfg\.pb*\.

   ```
   docker exec cli peer channel fetch config \ 
   /opt/home/channel-artifacts/ourchannelCfg.pb \
   -c ourchannel -o $ORDERER \
   --cafile /opt/home/managedblockchain-tls-chain.pem --tls
   ```

1. Decode the `ourchannelCfg.pb` configuration from protobuf into a JSON object as shown in the following command\.

   ```
   docker exec cli configtxlator proto_decode \
   --input /opt/home/channel-artifacts/ourchannelCfg.pb --type common.Block \
   --output /opt/home/channel-artifacts/config_block.json
   ```

1. Switch directories to the `channel-artifacts` directory\.

   ```
   cd channel-artifacts
   ```

1. Convert the JSON object that you created from the protobuf file into a streamlined JSON file named `config.json`, and then copy it to `config_copy.json`, as shown in the following commands\.

   ```
   sudo jq .data.data[0].payload.data.config config_block.json > config.json
   ```

   ```
   cp config.json config_copy.json
   ```

1. Use the `jq` tool to modify the `config_copy.json` configuration file by adding an anchor peer and to save the modifications to a new JSON configuration file named `modified_config.json` as shown in the following command\. Replace the items below as appropriate for your Managed Blockchain member, network, and peer configuration\.
   +  *OrgMemberID* is the ID of a Managed Blockchain member that belongs to the channel and owns the peer node that will be an anchor peer\. For example, `m-K46ICRRXJRCGRNNS4ES4XUUS5A`\.
   +  *PeerNodeDNS* is the endpoint, minus the port identifier, of the peer node\. For example, `nd-6EAJ5VA43JGGNPXOUZP7Y47E4Y.m-K46ICRRXJRCGRNNS4ES4XUUS5A.n-MWY63ZJZU5HGNCMBQER7IN6OIU.managedblockchain.us-east-1.amazonaws.com`\.
   +  *PortNumber* is the port number included in the **peer node endpoint** in the console or the `PeerEndpoint` in the Managed Blockchain API for the peer\. For example `30003`\.

   ```
   jq '.channel_group.groups.Application.groups["OrgMemberID"].values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "PeerNodeDNS","port": PortNumber}]},"version": "0"}}' config_copy.json > modified_config.json
   ```

1. Return the configuration JSON files to protobuf format, calculate the difference between them, and then generate a new `config_update.json` configuration file as shown in the following commands\.

   ```
   docker exec cli configtxlator proto_encode \
   --input /opt/home/channel-artifacts/config.json \
   --type common.Config \
   --output /opt/home/channel-artifacts/config.pb
   ```

   ```
   docker exec cli configtxlator proto_encode \
   --input /opt/home/channel-artifacts/modified_config.json \
   --type common.Config \
   --output /opt/home/channel-artifacts/modified_config.pb
   ```

   ```
   docker exec cli configtxlator compute_update \
   --channel_id ourchannel \
   --original /opt/home/channel-artifacts/config.pb \
   --updated /opt/home/channel-artifacts/modified_config.pb \
   --output /opt/home/channel-artifacts/config_update.pb
   ```

   ```
   docker exec cli configtxlator proto_decode \
   --input /opt/home/channel-artifacts/config_update.pb \
   --type common.ConfigUpdate \
   --output /opt/home/channel-artifacts/config_update.json
   ```

1. Change permissions for the `config_update.json` file you created above so that you can pipe its contents along with transaction envelope parameters to the `jq` tool and create the `config_update_in_envelope.json` file, as shown in the following commands\.

   ```
   sudo chmod 755 config_update.json
   ```

   ```
   echo '{"payload":{"header":{"channel_header":{"channel_id":"ourchannel", "type":2}},"data":{"config_update":'$(cat config_update.json)'}}}' | jq . > config_update_in_envelope.json
   ```

1. Convert the `config_update_in_envelope.json` file to a `config_update_in_envelope.pb` configuration file in protobuf format as shown in the following command\.

   ```
   docker exec cli configtxlator proto_encode --input /opt/home/channel-artifacts/config_update_in_envelope.json --type common.Envelope --output /opt/home/channel-artifacts/config_update_in_envelope.pb
   ```

1. Specify the `config_update_in_envelope.pb` configuration file to update the channel configuration on the orderer for the Managed Blockchain network as shown in the following command\.

   ```
   docker exec cli peer channel update \
   -f /opt/home/channel-artifacts/config_update_in_envelope.pb \
   -c ourchannel -o $ORDERER \
   --cafile /opt/home/managedblockchain-tls-chain.pem --tls
   ```