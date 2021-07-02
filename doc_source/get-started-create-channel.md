# Step 6: Create a Hyperledger Fabric Channel<a name="get-started-create-channel"></a>

In Hyperledger Fabric, a ledger exists in the scope of a channel\. The ledger can be shared across the entire network if every member is operating on a common channel\. A channel also can be privatized to include only a specific set of participants\. Members can be in your AWS account, or they can be members that you invite from other AWS accounts\.

In this step, you set up a basic channel\. Later on in the tutorial, in [Step 8: Invite Another AWS Account to be a Member and Create a Multi\-Member Channel](get-started-joint-channel.md), you go through a similar process to set up a channel that includes another member\.

Wait a minute or two for the administrative permissions from previous steps to propagate, and then perform these tasks to create a channel\.

**Note**  
All Hyperledger Fabric networks on Managed Blockchain support a maximum of 8 channels per network, regardless of network edition\.

## Step 6\.1: Create configtx for Hyperledger Fabric Channel Creation<a name="get-started-create-channel-configtx"></a>

The `configtx.yaml` file contains details of the channel configuration\. For more information, see [Channel Configuration \(configtx\)](https://hyperledger-fabric.readthedocs.io/en/release-1.4/configtx.html) in the Hyperledger Fabric documentation\.

This `configtx.yaml` enables application features associated with Hyperledger Fabric 1\.4\. It is not compatible with Hyperledger Fabric 1\.2\. For a `configtx.yaml` compatible with Hyperledger Fabric 1\.2, see [Work with Channels](hyperledger-work-with-channels.md)\.

Use a text editor to create a file with the following contents and save it as `configtx.yaml` on your Hyperledger Fabric client\. Note the following placeholders and values\.
+ Replace *MemberID* with the MemberID you returned previously\. For example *m\-K46ICRRXJRCGRNNS4ES4XUUS5A*\.
+ The `MSPDir` is set to the same directory location, `/opt/home/admin-msp`, that you established using the `CORE_PEER_MSPCONFIGPATH` environment variable in the Docker container for the Hyperledger Fabric CLI in [step 4\.4](get-started-create-client.md#get-started-client-configure-peer-cli)\.

**Important**  
This file is sensitive\. Artifacts from pasting can cause the file to fail with marshalling errors\. We recommend using `emacs` to edit it\. You can also use `VI`, but before using `VI`, enter `:set paste`, press `i` to enter insert mode, paste the contents, press escape, and then enter `:set nopaste` before saving\.

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
        Name: MemberID
        # ID to load the MSP definition as
        ID: MemberID
        #msp dir of org1 in the docker container
        MSPDir: /opt/home/admin-msp
        # AnchorPeers defines the location of peers which can be used
        # for cross org gossip communication. Note, this value is only
        # encoded in the genesis block in the Application section context
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
    OneOrgChannel:
        Consortium: AWSSystemConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1
```

Run the following command to generate the configtx peer block:

```
docker exec cli configtxgen \
-outputCreateChannelTx /opt/home/mychannel.pb \
-profile OneOrgChannel -channelID mychannel \
--configPath /opt/home/
```

**Important**  
Hyperledger Fabric requires that a channel ID contain only lowercase ASCII alphanumeric characters, dots \(\.\), and dashes \(\-\)\. It must start with a letter, and must be fewer than 250 characters\.

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