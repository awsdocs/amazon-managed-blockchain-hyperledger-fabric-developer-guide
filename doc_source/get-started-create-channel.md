# Step 6: Create a Hyperledger Fabric Channel<a name="get-started-create-channel"></a>

In Hyperledger Fabric, a ledger exists in the scope of a channel\. The ledger can be shared across the entire network if every member is operating on a common channel\. A channel also can be privatized to include only a specific set of participants\. Members can be in your AWS account, or they can be members that you invite from other AWS accounts\.

In this step, you set up a basic channel\. Later on in the tutorial, in [Step 8: Invite Another AWS Account to be a Member and Create a Multi\-Member Channel](get-started-joint-channel.md), you go through a similar process to set up a channel that includes another member\.

Wait a minute or two for the administrative permissions from previous steps to propagate, and then perform these tasks to create a channel\.

**Note**  
All Hyperledger Fabric networks on Managed Blockchain support a maximum of 8 channels per network, regardless of network edition\.

## Step 6\.1: Create configtx for Hyperledger Fabric Channel Creation<a name="get-started-create-channel-configtx"></a>

The `configtx.yaml` file contains details of the channel configuration\. For more information, see [Channel Configuration \(configtx\)](https://hyperledger-fabric.readthedocs.io/en/release-2.2/configtx.html) in the Hyperledger Fabric documentation\.

This `configtx.yaml` enables application features associated with Hyperledger Fabric 2\.2\. It is not compatible with Hyperledger Fabric 1\.2 or 1\.4\. For a `configtx.yaml` compatible with Hyperledger Fabric 1\.2 or 1\.4, see [Work with Channels](hyperledger-work-with-channels.md)\.

Use a text editor to create a file with the following contents and save it as `configtx.yaml` on your Hyperledger Fabric client\. Note the following placeholders and values\.
+ Replace *MemberID* with the MemberID you returned previously\. For example *m\-K46ICRRXJRCGRNNS4ES4XUUS5A*\.
+ The `MSPDir` is set to the same directory location, `/opt/home/admin-msp`, that you established using the `CORE_PEER_MSPCONFIGPATH` environment variable in the Docker container for the Hyperledger Fabric CLI in [step 4\.4](get-started-create-client.md#get-started-client-configure-peer-cli)\.

**Important**  
This file is sensitive\. Artifacts from pasting can cause the file to fail with marshalling errors\. We recommend using `emacs` to edit it\. You can also use `VI`, but before using `VI`, enter `:set paste`, press `i` to enter insert mode, paste the contents, press escape, and then enter `:set nopaste` before saving\.

```
################################################################################
#
#   ORGANIZATIONS
#
#   This section defines the organizational identities that can be referenced
#   in the configuration profiles.
#
################################################################################
Organizations:
    # Org1 defines an MSP using the sampleconfig. It should never be used
    # in production but may be used as a template for other definitions.
    - &Org1
        # Name is the key by which this org will be referenced in channel
        # configuration transactions.
        # Name can include alphanumeric characters as well as dots and dashes.
        Name: MemberID
        # ID is the key by which this org's MSP definition will be referenced.
        # ID can include alphanumeric characters as well as dots and dashes.
        ID: MemberID
        # SkipAsForeign can be set to true for org definitions which are to be
        # inherited from the orderer system channel during channel creation.  This
        # is especially useful when an admin of a single org without access to the
        # MSP directories of the other orgs wishes to create a channel.  Note
        # this property must always be set to false for orgs included in block
        # creation.
        SkipAsForeign: false
        Policies: &Org1Policies
            Readers:
                Type: Signature
                Rule: "OR('Org1.member')"
                # If your MSP is configured with the new NodeOUs, you might
                # want to use a more specific rule like the following:
                # Rule: "OR('Org1.admin', 'Org1.peer', 'Org1.client')"
            Writers:
                Type: Signature
                Rule: "OR('Org1.member')"
                # If your MSP is configured with the new NodeOUs, you might
                # want to use a more specific rule like the following:
                # Rule: "OR('Org1.admin', 'Org1.client')"
            Admins:
                Type: Signature
                Rule: "OR('Org1.admin')"
        # MSPDir is the filesystem path which contains the MSP configuration.
        MSPDir: /opt/home/admin-msp
        # AnchorPeers defines the location of peers which can be used for
        # cross-org gossip communication. Note, this value is only encoded in
        # the genesis block in the Application section context.
        AnchorPeers:
            - Host: 127.0.0.1
              Port: 7051
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
        # V2.0 for Channel is a catchall flag for behavior which has been
        # determined to be desired for all orderers and peers running at the v2.0.0
        # level, but which would be incompatible with orderers and peers from
        # prior releases.
        # Prior to enabling V2.0 channel capabilities, ensure that all
        # orderers and peers on a channel are at v2.0.0 or later.
        V2_0: true
    # Orderer capabilities apply only to the orderers, and may be safely
    # used with prior release peers.
    # Set the value of the capability to true to require it.
    Orderer: &OrdererCapabilities
        # V1.1 for Orderer is a catchall flag for behavior which has been
        # determined to be desired for all orderers running at the v1.1.x
        # level, but which would be incompatible with orderers from prior releases.
        # Prior to enabling V2.0 orderer capabilities, ensure that all
        # orderers on a channel are at v2.0.0 or later.
        V2_0: true
    # Application capabilities apply only to the peer network, and may be safely
    # used with prior release orderers.
    # Set the value of the capability to true to require it.
    # Note that setting a later Application version capability to true will also
    # implicitly set prior Application version capabilities to true. There is no need
    # to set each version capability to true (prior version capabilities remain
    # in this sample only to provide the list of valid values).
    Application: &ApplicationCapabilities
        # V2.0 for Application enables the new non-backwards compatible
        # features and fixes of fabric v2.0.
        # Prior to enabling V2.0 orderer capabilities, ensure that all
        # orderers on a channel are at v2.0.0 or later.
        V2_0: true
################################################################################
#
#   CHANNEL
#
#   This section defines the values to encode into a config transaction or
#   genesis block for channel related parameters.
#
################################################################################
Channel: &ChannelDefaults
    # Policies defines the set of policies at this level of the config tree
    # For Channel policies, their canonical path is
    #   /Channel/<PolicyName>
    Policies:
        # Who may invoke the 'Deliver' API
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        # Who may invoke the 'Broadcast' API
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        # By default, who may modify elements at this config level
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
    # Capabilities describes the channel level capabilities, see the
    # dedicated Capabilities section elsewhere in this file for a full
    # description
    Capabilities:
        <<: *ChannelCapabilities
################################################################################
#
#   APPLICATION
#
#   This section defines the values to encode into a config transaction or
#   genesis block for application-related parameters.
#
################################################################################
Application: &ApplicationDefaults
    # Organizations is the list of orgs which are defined as participants on
    # the application side of the network
    Organizations:
    # Policies defines the set of policies at this level of the config tree
    # For Application policies, their canonical path is
    #   /Channel/Application/<PolicyName>
    Policies: &ApplicationDefaultPolicies
        LifecycleEndorsement:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Endorsement:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"

    Capabilities:
        <<: *ApplicationCapabilities
################################################################################
#
#   PROFILES
#
#   Different configuration profiles may be encoded here to be specified as
#   parameters to the configtxgen tool. The profiles which specify consortiums
#   are to be used for generating the orderer genesis block. With the correct
#   consortium members defined in the orderer genesis block, channel creation
#   requests may be generated with only the org member names and a consortium
#   name.
#
################################################################################
Profiles:
    OneOrgChannel:
        <<: *ChannelDefaults
        Consortium: AWSSystemConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - <<: *Org1
```

Run the following command to generate the configtx peer block:

```
docker exec cli configtxgen \
-outputCreateChannelTx /opt/home/mychannel.pb \
-profile OneOrgChannel -channelID mychannel \
--configPath /opt/home/
```

**Important**  
Hyperledger Fabric 2\.2 requires that a channel ID contain only lowercase ASCII alphanumeric characters, dots \(\.\), and dashes \(\-\)\. It must start with a letter, and must be fewer than 250 characters\.

## Step 6\.2: Set Environment Variables for the Orderer<a name="get-started-create-channel-environment-variables"></a>

Set the `$ORDERER` environment variable for convenience\. Replace *orderer\.n\-MWY63ZJZU5HGNCMBQER7IN6OIU\.managedblockchain\.amazonaws\.com:*30001** with the `OrderingServiceEndpoint` returned by the `aws managedblockchain get-network` command and listed on the network details page of the Managed Blockchain console\.

```
export ORDERER=orderer.n-MWY63ZJZU5HGNCMBQER7IN6OIU.managedblockchain.amazonaws.com:30001
```

This variable must be exported each time you log out of the client\. To persist the variable across sessions, add the export statement to your `~/.bash_profile` as shown in the following example\.

```
# .bash_profile
...other configurations
export ORDERER=orderer.n-MWY63ZJZU5HGNCMBQER7IN6OIU.managedblockchain.amazonaws.com:30001
```

After updating `.bash_profile`, apply the changes:

```
source ~/.bash_profile
```

## Step 6\.3: Create the Channel<a name="get-started-create-channel-create-channel"></a>

Run the following command to create a channel using the variables that you established and the configtx peer block that you created:

```
docker exec cli peer channel create -c mychannel \
-f /opt/home/mychannel.pb -o $ORDERER \
--cafile /opt/home/managedblockchain-tls-chain.pem --tls
```

**Important**  
It may take a minute or two after you enroll an administrative user for you to be able to use your administrator certificate to create a channel with the ordering service\.

## Step 6\.4: Join Your Peer Node to the Channel<a name="get-started-create-channel-join-peer"></a>

Run the following command to join the peer node that you created earlier to the channel:

```
docker exec cli peer channel join -b mychannel.block \
-o $ORDERER --cafile /opt/home/managedblockchain-tls-chain.pem --tls
```