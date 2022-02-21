# Step 3: Create a Peer Node in Your Membership<a name="get-started-create-peer-node"></a>

Now that your network and the first member are up and running, you can use the Managed Blockchain console or the AWS CLI to create a peer node\. Your member's peer nodes interact with other members' peer nodes on the blockchain to query and update the ledger, and store a local copy of the ledger\.

Use one of the following procedures to create a peer node\.

## To create a peer node using the AWS Management Console<a name="w79aab9c15b7b1"></a>

1. Open the Managed Blockchain console at [https://console\.aws\.amazon\.com/managedblockchain/](https://console.aws.amazon.com/managedblockchain/)\.

1. Choose **Networks**, select the network from the list, and then choose **View details**\.

1. Select a **Member** from the list, and then choose **Create peer node**\.

1. Choose configuration parameters for your peer node according to the guidelines in [Work with Hyperledger Fabric Peer Nodes on Managed Blockchain](managed-blockchain-hyperledger-peer-nodes.md), and then choose **Create peer node**\.

## To create a peer node using the AWS CLI<a name="w79aab9c15b7b3"></a>
+ Use the `create-node` command, as shown in the following example\. Replace the value of `--network-id`, `--member-id`, and `AvailabilityZone` as appropriate\.

  ```
  [ec2-user@ip-192-0-2-17 ~]$ aws managedblockchain create-node \
  --node-configuration '{"InstanceType":"bc.t3.small","AvailabilityZone":"us-east-1a"}' \
  --network-id n-MWY63ZJZU5HGNCMBQER7IN6OIU \
  --member-id m-K46ICRRXJRCGRNNS4ES4XUUS5A
  ```

  The command returns output that includes the peer node's `NodeID`, as shown in the following example:

  ```
  {
       "NodeId": "nd-6EAJ5VA43JGGNPXOUZP7Y47E4Y"
  }
  ```