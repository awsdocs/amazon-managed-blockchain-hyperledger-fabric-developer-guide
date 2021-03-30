# Step 3: Create an Amazon EC2 Instance and Set Up the Hyperledger Fabric Client<a name="get-started-create-client"></a>

To complete this step, you launch an Amazon EC2 instance using the Amazon Linux AMI\. Consider the following requirements and recommendations when you create the Hyperledger Fabric client Amazon EC2 instance:
+ We recommend that you launch the client Amazon EC2 instance in the same VPC and using the same security group as the VPC Endpoint that you created in [Step 2: Create and Configure the Interface VPC Endpoint](get-started-create-endpoint.md)\. This simplifies connectivity between the Amazon EC2 instance and the Interface VPC Endpoint\.
+ We recommend that the EC2 security group shared by the VPC Endpoint and the client Amazon EC2 instance have rules that allow all inbound and outbound traffic between members of the security group\. This also simplifies connectivity\. In addition, ensure that this security group or another security group associated with the client Amazon EC2 instance has a rule that allows inbound SSH connections from a source that includes your SSH client's IP address\. For more information about security groups and required rules, see [Configuring Security Groups for Hyperledger Fabric on Amazon Managed Blockchain](managed-blockchain-security-sgs.md)\.
+ Make sure that the client Amazon EC2 instance is configured with an automatically assigned public IP address and that you can connect to it using SSH\. For more information, see [Getting Started with Amazon EC2 Linux Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html) and [Connect to your Linux instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstances.html) in the *Amazon EC2 User Guide for Linux Instances*\.
+ Make sure that the service role associated with the EC2 instance allows access to the Amazon S3 bucket where Managed Blockchain certificates are stored and that it has required permissions for working with Managed Blockchain resources\. For more information, see [Example IAM Role Permissions Policy for Hyperledger Fabric Client EC2 Instance](security_iam_hyperledger_ec2_client.md)\.

**Note**  
An AWS CloudFormation template to create a Hyperledger Fabric client is available in [amazon\-managed\-blockchain\-client\-templates repository](https://github.com/awslabs/amazon-managed-blockchain-client-templates) on Github\. For more information, see the [readme\.md](https://github.com/awslabs/amazon-managed-blockchain-client-templates/blob/master/README.md) in that repository\. For more information about using AWS CloudFormation, see [Getting Started](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/GettingStarted.Walkthrough.html) in the *AWS CloudFormation User Guide*\.

## Step 3\.1: Install Packages<a name="get-started-client-install-packages"></a>

Your Hyperledger Fabric client needs some packages and samples installed so that you can work with the Hyperledger Fabric resources\. In this step, you install Go, Docker, Docker Compose, and some other utilities\. You also create variables in the `~/.bash_profile` for your development environment\. These are prerequisites for installing and using Hyperledger tools\. 

While connected to the Hyperledger Fabric client using SSH, run the following commands to install utilities, install docker, and configure the Docker user to be the default user for the Amazon EC2 instance:

```
sudo yum update -y
```

```
sudo yum install jq telnet emacs docker libtool libtool-ltdl-devel git -y
```

```
sudo service docker start
```

```
sudo usermod -a -G docker ec2-user
```

Log out and log in again for the `usermod` command to take effect\.

Run the following commands to install Docker Compose\.

```
sudo curl -L \
https://github.com/docker/compose/releases/download/1.20.0/docker-compose-`uname \
-s`-`uname -m` -o /usr/local/bin/docker-compose
```

```
sudo chmod a+x /usr/local/bin/docker-compose
```

Run the following commands to install golang\.

```
wget https://dl.google.com/go/go1.14.4.linux-amd64.tar.gz
```

```
tar -xzf go1.14.4.linux-amd64.tar.gz
```

```
sudo mv go /usr/local
```

```
sudo yum install git -y
```

Use a text editor to set up variables such as `GOROOT` and `GOPATH` in your `~/.bashrc` or `~/.bash_profile` and save the updates\. The following example shows entries in `.bash_profile`\.

```
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi

# User specific environment and startup programs
PATH=$PATH:$HOME/.local/bin:$HOME/bin

# GOROOT is the location where Go package is installed on your system
export GOROOT=/usr/local/go

# GOPATH is the location of your work directory
export GOPATH=$HOME/go

# CASERVICEENDPOINT is the endpoint to reach your member's CA
# for example ca.m-K46ICRRXJRCGRNNS4ES4XUUS5A.n-MWY63ZJZU5HGNCMBQER7IN6OIU.managedblockchain.us-east-1.amazonaws.com:30002
export CASERVICEENDPOINT=MyMemberCaEndpoint

# ORDERER is the endpoint to reach your network's orderer
# for example orderer.n-MWY63ZJZU5HGNCMBQER7IN6OIU.managedblockchain.amazonaws.com:30001
export ORDERER=MyNetworkOrdererEndpoint

# Update PATH so that you can access the go binary system wide
export PATH=$GOROOT/bin:$PATH
export PATH=$PATH:/home/ec2-user/go/src/github.com/hyperledger/fabric-ca/bin
```

After you update `.bash_profile`, apply the changes:

```
source ~/.bash_profile 
```

After the installation, verify that you have the correct versions installed:
+ Docker–17\.06\.2\-ce or later
+ Docker\-compose–1\.14\.0 or later
+ Go–1\.14\.x

To check the Docker version, run the following command:

```
sudo docker version
```

The command returns output similar to the following:

```
Client:
 Version: 18.06.1-ce
 API version: 1.38
 Go version: go1.14.4
 Git commit: CommitHash
 Built: Tue Oct 2 18:06:45 2018
 OS/Arch: linux/amd64
 Experimental: false

Server:
 Engine:
 Version: 18.06.1-ce
 API version: 1.38 (minimum version 1.12)
 Go version: go1.14.4
 Git commit: e68fc7a/18.06.1-ce
 Built: Tue Oct 2 18:08:26 2018
 OS/Arch: linux/amd64
 Experimental: false
```

To check the version of Docker Compose, run the following command:

```
sudo /usr/local/bin/docker-compose version
```

The command returns output similar to the following:

```
docker-compose version 1.22.0, build f46880fe
docker-py version: 3.4.1
CPython version: 3.6.6
OpenSSL version: OpenSSL 1.1.0f  25 May 2017
```

To check the version of go, run the following command:

```
go version
```

The command returns output similar to the following:

```
go version go1.14.4 linux/amd64
```

## Step 3\.2: Set Up the Hyperledger Fabric CA Client<a name="get-started-client-setup-CA-client"></a>

In this step, you verify that you can connect to the Hyperledger Fabric CA using the VPC endpoint you configured in [Step 2: Create and Configure the Interface VPC Endpoint](get-started-create-endpoint.md)\. You then install the Hyperledger Fabric CA client\. The Fabric CA issues certificates to administrators and network peers\.

To verify connectivity to the Hyperledger Fabric CA, you need the `CAEndpoint`\. Use the `get-member` command to get the CA endpoint for your member, as shown in the following example\. Replace the values of `--network-id` and `--member-id` with the values returned in [Step 1: Create the Network and First Member](get-started-create-network.md)\.

```
aws managedblockchain get-member \
--network-id n-MWY63ZJZU5HGNCMBQER7IN6OIU \
--member-id m-K46ICRRXJRCGRNNS4ES4XUUS5A
```

Use `curl` or `telnet` to verify that the endpoint resolves\. In the following example, replace `CAEndpoint` with the **CAEndpoint** returned by the `get-member` command\.

```
curl https://$CASERVICEENDPOINT/cainfo -k
```

The command should return output similar to the following:

```
{"result":{"CAName":"abcd1efghijkllmn5op3q52rst","CAChain":"LongStringOfCharacters","Version":"1.4.7-snapshot-"}
,"errors":[],"messages":[],"success":true}
```

Alternatively, you can connect to the Fabric CA using Telnet as shown in the following example\. Use the same endpoint in the `curl` example, but separate the endpoint and the port as shown in the following example\.

```
telnet CaEndpoint-Without-Port CaPort
```

The command should return output similar to the following:

```
Trying 10.0.1.228...
Connected to ca.m-K46ICRRXJRCGRNNS4ES4XUUS5A.n-MWY63ZJZU5HGNCMBQER7IN6OIU.managedblockchain.us-east-1.amazonaws.com.
Escape character is '^]'.
```

If you are unable to connect to the Fabric CA, double\-check your network settings to ensure that the client Amazon EC2 instance has connectivity with the VPC Endpoint\. In particular, ensure that the security groups associated with both the VPC Endpoint and the client Amazon EC2 instance have inbound and outbound rules that allow traffic between them\.

Now that you have verified that you can connect to the Hyperledger Fabric CA, run the following commands to configure the CA client\.

**Note**  
If you are working with Hyperledger Fabric v1\.2 networks, you need to install and build the correct client version, which is available at https://github\.com/hyperledger/fabric\-ca/releases/download/v1\.2\.1/hyperledger\-fabric\-ca\-linux\-amd64\-1\.2\.1\.tar\.gz\.

```
mkdir -p /home/ec2-user/go/src/github.com/hyperledger/fabric-ca
```

```
cd /home/ec2-user/go/src/github.com/hyperledger/fabric-ca
```

```
wget https://github.com/hyperledger/fabric-ca/releases/download/v1.4.7/hyperledger-fabric-ca-linux-amd64-1.4.7.tar.gz
```

```
tar -xzf hyperledger-fabric-ca-linux-amd64-1.4.7.tar.gz
```

## Step 3\.3: Clone the Samples Repository<a name="get-started-client-clone-samples"></a>

**Note**  
If you are working with Hyperledger Fabric v1\.2 networks, use `--branch v1.2.0` instead of `--branch v1.4.7` in the following commmands\.

```
cd /home/ec2-user
```

```
git clone --branch v1.4.7 https://github.com/hyperledger/fabric-samples.git
```

## Step 3\.4: Configure and Run Docker Compose to Start the Hyperledger Fabric CLI<a name="get-started-client-configure-peer-cli"></a>

Use a text editor to create a configuration file for Docker Compose named `docker-compose-cli.yaml` in the `/home/ec2-user` directory, which you use to run the Hyperledger Fabric CLI\. You use this CLI to interact with peer nodes that your member owns\. Copy the following contents into the file and replace the `placeholder values` according to the following guidance:
+ *MyMemberID* is the `MemberID` returned by the `aws managedblockchain list-members` AWS CLI command and shown on the member details page of the Managed Blockchain console—for example, `m-K46ICRRXJRCGRNNS4ES4XUUS5A`\.
+ *MyPeerNodeEndpoint* is the `PeerEndpoint` returned by the `aws managedblockchain get-node` command and listed on the node details page of the Managed Blockchain console—for example, nd\-6EAJ5VA43JGGNPXOUZP7Y47E4Y\.m\-K46ICRRXJRCGRNNS4ES4XUUS5A\.n\-MWY63ZJZU5HGNCMBQER7IN6OIU\.managedblockchain\.*us\-east\-1*\.amazonaws\.com:*30003*\.

When you subsequently use the `cli` container to run commands—for example, `docker exec cli peer channel create`—you can use the `-e` option to override an environment variable that you establish in the `docker-compose-cli.yaml` file\.

**Note**  
If you are working with Hyperledger Fabric v1\.2 networks, use `image: hyperledger/fabric-tools:1.2` in the following example instead of `image: hyperledger/fabric-tools:1.4`\. In addition, use `CORE_LOGGING_LEVEL=info` instead of `FABRIC_LOGGING_SPEC=info`\.

```
version: '2'
services:
  cli:
    container_name: cli
    image: hyperledger/fabric-tools:1.4
    tty: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - FABRIC_LOGGING_SPEC=info # Set logging level to debug for more verbose logging
      - CORE_PEER_ID=cli
      - CORE_CHAINCODE_KEEPALIVE=10
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/home/managedblockchain-tls-chain.pem
      - CORE_PEER_LOCALMSPID=MyMemberID
      - CORE_PEER_MSPCONFIGPATH=/opt/home/admin-msp
      - CORE_PEER_ADDRESS=MyPeerNodeEndpoint
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
        - /var/run/:/host/var/run/
        - /home/ec2-user/fabric-samples/chaincode:/opt/gopath/src/github.com/
        - /home/ec2-user:/opt/home
```

Run the following command to start the Hyperledger Fabric peer CLI container:

```
docker-compose -f docker-compose-cli.yaml up -d
```

If you restarted or logged out and back in after the `usermod` command in [Step 3\.1: Install Packages](#get-started-client-install-packages), you shouldn't need to run this command with `sudo`\. If the command fails, you can log out and log back in\. Alternatively, you can run the command using `sudo`, as shown in the following example:

```
sudo /usr/local/bin/docker-compose -f docker-compose-cli.yaml up -d
```