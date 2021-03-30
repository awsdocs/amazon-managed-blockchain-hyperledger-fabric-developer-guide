# Work with Channels<a name="hyperledger-work-with-channels"></a>

A *channel* is a private communication pathway between two or more members of a Hyperledger Fabric network on Amazon Managed Blockchain\. Each transaction on a Hyperledger Fabric network occurs on a channel\. A network must contain at least one channel and a network on Managed Blockchain can have up to eight channels\.

Each channel has its own transaction ledger\. The ordering service on the network manages transactions according to a channel configuration that a member creates\. The channel configuration specifies which members participate in the channel, the peer nodes that can transact on the channel, and the policies that govern channel updates\.

To participate on a channel, a member must join at least one peer node to the channel\. A peer node can belong to multiple channels, but ledger data from one channel can't pass to ledger data in another channel\. A member shares their admin certificates and root CA with the channel creator when they join the channel\. These artifacts allow the ordering service to authenticate and authorize members of the channel when they join their peer nodes to the channel\.

Optionally, a channel also can contain private data collections\. A private data collection allows a subset of channel members to share ledger data independently of the ordering service\. Private data collections are supported only with Hyperledger Fabric 1\.4 or later\. Private data collections require that at least one peer node for a member is set up as an anchor peer\. For more information, see [Add an Anchor Peer to a Channel](hyperledger-anchor-peers.md) and [Work with Private Data Collections in Hyperledger Fabric on Managed Blockchain](managed-blockchain-hyperledger-create-pdc.md)\.

## Create a Channel<a name="create-a-channel"></a>

A channel creator is a member that uses their Hyperledger Fabric client to create a channel configuration file\. The channel creator then runs a command that outputs a channel transaction file based on that configuration file\. Finally, the channel creator uses the transaction file to run a command that creates the channel with the Hyperledger Fabric ordering service on Managed Blockchain\. The channel creator must get security artifacts from channel members to be able to add the members to the channel\.

After the channel is created, channel members, including the channel creator, download the channel genesis block to their respective Hyperledger Fabric client machines and join peer nodes to the channel\.

**To create and join a Hyperledger Fabric channel on Managed Blockchain**

The following steps specify file locations in terms of both the Hyperledger Fabric client machine's file system and the Hyperledger Fabric CLI Docker container file system, as appropriate\. When the CLI is installed, the Docker Compose configuration file maps these locations to one another\. For more information, see [step 3\.4](get-started-create-client.md#get-started-client-configure-peer-cli) in the [Getting Started](managed-blockchain-get-started-tutorial.md) tutorial of this guide\.

In the following examples, the client machine directory `/home/ec2-user` is mapped to the container directory `/opt/home`\. In steps where you save artifacts, the steps use the client machine file system\. In steps where you reference artifacts in CLI commands and configuration files, the steps use the container file system\.

1. Each member that joins a channel shares its security artifacts with the channel creator\. The channel creator saves the artifacts on the Hyperledger Fabric client machine\. The channel creator then references the location of these artifacts in the next step\. For more information about sharing artifacts, see [Step 8\.4](get-started-joint-channel.md#get-started-joint-channel-artifact-exchange) and 8\.5 in the getting started tutorial of this guide\. Sharing artifacts is a prerequisite for the steps below\.

1. On your Hyperledger Fabric client machine, create a configuration file in yaml format named `configtx.yaml`\. Save this file to a file location on your Hyperledger Fabric client machine that is mapped to the CLI container file system\. The following steps use `/home/ec2-user`, which is mapped to `/opt/home`\.

   Use the examples below as a starting point\. For more information about configuration files, see [Channel configuration \(configtx\)](https://hyperledger-fabric.readthedocs.io/en/release-1.4/configtx.html) in the Hyperledger Fabric documentation\. Replace the following configuration parameters as appropriate for your channel, and add sections as required for additional members\.
   + *Org*<n>*MemberID* is the respective member ID for each channel participant, for example, `m-K46ICRRXJRCGRNNS4ES4XUUS5A`\.
   + *Org*<n>*AdminMSPDir* is the Docker container file system directory where security artifacts for each corresponding member are saved\. For example, *opt/home/Org2AdminMSP* references artifacts saved to `/home/ec2-user/Org2AdminMSP` on the channel creator's client machine\.

   The configtx file for a Hyperledger Fabric 1\.4 network includes application settings to enable expanded features\. These settings are not supported in Hyperledger Fabric 1\.2 channels\. Examples for version 1\.2 and 1\.4 networks are provided below\.

------
#### [ Version 1\.4 ]

   ```
   ################################################################################
   #
   # Section: Organizations
   #
   # - This section defines the different organizational identities which will
   # be referenced later in the configuration.
   #
   ################################################################################
   Organizations:
       - &Org1
           # member id defines the organization
           Name: Org1MemberID
           # ID to load the MSP definition as
           ID: Org1MemberID
           #msp dir of org1 in the docker container
           MSPDir: /opt/home/Org1AdminMSP
           # AnchorPeers defines the location of peers which can be used
           # for cross org gossip communication. Note, this value is only
           # encoded in the genesis block in the Application section context
           AnchorPeers:
               - Host:
                 Port:
       - &Org2
           Name: Org2MemberID
           ID: Org2MemberID
           #msp dir of org2 in the docker container
           MSPDir: /opt/home/Org2AdminMSP
           AnchorPeers:
               - Host: 
                 Port:
   ################################################################################
   #
   #   CAPABILITIES
   #
   #   This section defines the capabilities of fabric network. This is a new
   #   concept as of v1.1.0 and should not be utilized in mixed networks with
   #   v1.0.x peers and orderers.  Capabilities define features which must be
   #   present in a fabric binary for that binary to safely participate in the
   #   fabric network.  For instance, if a new MSP type is added, newer binaries
   #   might recognize and validate the signatures from this type, while older
   #   binaries without this support would be unable to validate those
   #   transactions.  This could lead to different versions of the fabric binaries
   #   having different world states.  Instead, defining a capability for a channel
   #   informs those binaries without this capability that they must cease
   #   processing transactions until they have been upgraded.  For v1.0.x if any
   #   capabilities are defined (including a map with all capabilities turned off)
   #   then the v1.0.x peer will deliberately crash.
   #
   ################################################################################
   Capabilities:
       # Channel capabilities apply to both the orderers and the peers and must be
       # supported by both.
       # Set the value of the capability to true to require it.
       # Note that setting a later Channel version capability to true will also
       # implicitly set prior Channel version capabilities to true. There is no need
       # to set each version capability to true (prior version capabilities remain
       # in this sample only to provide the list of valid values).
       Channel: &ChannelCapabilities
           # V1.4.3 for Channel is a catchall flag for behavior which has been
           # determined to be desired for all orderers and peers running at the v1.4.3
           # level, but which would be incompatible with orderers and peers from
           # prior releases.
           # Prior to enabling V1.4.3 channel capabilities, ensure that all
           # orderers and peers on a channel are at v1.4.3 or later.
           V1_4_3: true
           # V1.3 for Channel enables the new non-backwards compatible
           # features and fixes of fabric v1.3
           V1_3: false
           # V1.1 for Channel enables the new non-backwards compatible
           # features and fixes of fabric v1.1
           V1_1: false
       # Application capabilities apply only to the peer network, and may be safely
       # used with prior release orderers.
       # Set the value of the capability to true to require it.
       # Note that setting a later Application version capability to true will also
       # implicitly set prior Application version capabilities to true. There is no need
       # to set each version capability to true (prior version capabilities remain
       # in this sample only to provide the list of valid values).
       Application: &ApplicationCapabilities
           # V1.4.2 for Application enables the new non-backwards compatible
           # features and fixes of fabric v1.4.2
           V1_4_2: true
           # V1.3 for Application enables the new non-backwards compatible
           # features and fixes of fabric v1.3.
           V1_3: false
           # V1.2 for Application enables the new non-backwards compatible
           # features and fixes of fabric v1.2 (note, this need not be set if
           # later version capabilities are set)
           V1_2: false
           # V1.1 for Application enables the new non-backwards compatible
           # features and fixes of fabric v1.1 (note, this need not be set if
           # later version capabilities are set).
           V1_1: false
   ################################################################################
   #
   # SECTION: Application
   #
   # - This section defines the values to encode into a config transaction or
   # genesis block for application related parameters
   #
   ################################################################################
   Application: &ApplicationDefaults
       # Organizations is the list of orgs which are defined as participants on
       # the application side of the network
       Organizations:
       Capabilities:
           <<: *ApplicationCapabilities
   ################################################################################
   #
   # Profile
   #
   # - Different configuration profiles may be encoded here to be specified
   # as parameters to the configtxgen tool
   #
   ################################################################################
   Profiles:
       TwoOrgChannel:
           Consortium: AWSSystemConsortium
           Application:
               <<: *ApplicationDefaults
               Organizations:
                   - *Org1
                   - *Org2
   ```

------
#### [ Version 1\.2 ]

   ```
   ################################################################################
   #
   #   Section: Organizations
   #
   #   - This section defines the different organizational identities which will
   #   be referenced later in the configuration.
   #
   ################################################################################
   Organizations:
       - &Org1
               # member id defines the organization
           Name: Member1ID
               # ID to load the MSP definition as
           ID: Member1ID
               #msp dir of org1 in the docker container
           MSPDir: /opt/home/Org1AdminMSP
               # AnchorPeers defines the location of peers which can be used
               # for cross org gossip communication.  Note, this value is only
               # encoded in the genesis block in the Application section context
           AnchorPeers:
               - Host:
                 Port:
       - &Org2
           Name: Member2ID
           ID: Member2ID
           MSPDir: /opt/home/Org2AdminMSP
           AnchorPeers:
               - Host:
                 Port:
   
   ################################################################################
   #
   #   SECTION: Application
   #
   #   - This section defines the values to encode into a config transaction or
   #   genesis block for application related parameters
   #
   ################################################################################
   Application: &ApplicationDefaults
           # Organizations is the list of orgs which are defined as participants on
           # the application side of the network
        Organizations:
   
   ################################################################################
   #
   #   Profile
   #
   #   - Different configuration profiles may be encoded here to be specified
   #   as parameters to the configtxgen tool
   #
   ################################################################################
   Profiles:
       TwoOrgChannel:
           Consortium: AWSSystemConsortium
           Application:
               <<: *ApplicationDefaults
               Organizations:
                   - *Org1
                   - *Org2
   ```

------

1. The channel creator uses the Hyperledger Fabric CLI [configtxgen](https://hyperledger-fabric.readthedocs.io/en/release-1.4/commands/configtxgen.html) command as shown in the following example to write a channel creation transaction\.

   Replace the following configuration parameters as appropriate for your channel\.
   + */opt/home/ourchannel\.pb* is the channel creation transaction file that the command creates\. Specify the file location using the container file system\.
   + *TwoOrgChannel* is the profile specified in the `configtx.yaml` from the previous step\.
   + *ourchannel* is the channel ID that will be used for the channel\.
   + */opt/home/* for the `--configPath` option is the container location where the `configtx.yaml` file is saved\.

   ```
   docker exec cli configtxgen \
   -outputCreateChannelTx /opt/home/ourchannel.pb \
   -profile TwoOrgChannel -channelID ourchannel \
   --configPath /opt/home/
   ```

1. The channel creator uses the Hyperledger Fabric CLI [channel create](https://hyperledger-fabric.readthedocs.io/en/release-1.4/commands/peerchannel.html#peer-channel-create) command to create a channel and write the genesis block to the orderer\. The `-f` option specifies the transaction file that you created in the previous step, and the `$ORDERER` environment variable in the example has been set to the endpoint of the network orderer for simplicity—for example, `orderer.n-MWY63ZJZU5HGNCMBQER7IN6OIU.managedblockchain.amazonaws.com:30001`\.

   ```
   docker exec cli peer channel create -c ourchannel \
   -f /opt/home/ourchannel.pb -o $ORDERER \ 
   --cafile /opt/home/managedblockchain-tls-chain.pem --tls
   ```

1. To join a peer node to the channel, each member must fetch the channel genesis block from the orderer, write it to a file, and then reference that genesis block when they join their peer or peers\.

   1. To get the channel genesis block, each member uses the Hyperledger Fabric CLI [peer channel fetch oldest](https://hyperledger-fabric.readthedocs.io/en/release-1.4/commands/peerchannel.html#peer-channel-fetch) command\. In the following example, the genesis block is written to a file in the container file system, */opt/home/ourchannel\.block*\. The `$ORDERER` variable is used in the same ways the previous step, and the `ourchannel` channel ID is specified\.

      ```
      docker exec cli peer channel fetch oldest /opt/home/ourchannel.block \
      -c ourchannel -o $ORDERER \
      --cafile /opt/home/managedblockchain-tls-chain.pem --tls
      ```

   1. Using the channel genesis block file created above, each member uses the Hyperledger Fabric [peer channel join](https://hyperledger-fabric.readthedocs.io/en/release-1.4/commands/peerchannel.html#peer-channel-join) command to join their member's peer node to the channel\. 

      ```
      docker exec cli peer channel join -b /opt/home/ourchannel.block \
      -o $ORDERER --cafile /opt/home/managedblockchain-tls-chain.pem --tls
      ```