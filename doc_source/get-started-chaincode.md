# Step 7: Install and Run Chaincode<a name="get-started-chaincode"></a>

In this section, you create and install a package for [golang sample chaincode](https://github.com/hyperledger/fabric-samples/blob/release-2.2/chaincode/abstore/go/abstore.go) on your peer node\. You also approve, commit, and verify the chaincode\.

You then use the chaincode's `init` command to initialize values attributed to entities `a` and `b` in the ledger, followed by the `query` command to confirm that initialization was successful\. Next, you use the chaincode's `invoke` command to transfer 10 units from `a` to `b` in the ledger\. Finally, you use the chaincode's `query` command again to confirm that the value attributed to `a` was decremented by 10 units in the ledger\.

## Step 7\.1: Install Vendor Dependencies<a name="get-started-chaincode-install-vendor-dependencies"></a>

Run the following commands to enable *vendoring* for the Go module dependencies of your example chaincode\.

```
sudo chown -R ec2-user:ec2-user fabric-samples/
cd fabric-samples/chaincode/abstore/go/
GO111MODULE=on go mod vendor
cd -
```

## Step 7\.2: Create the Chaincode Package<a name="get-started-chaincode-create-package"></a>

Run the following command to create the example chaincode package\.

```
docker exec cli peer lifecycle chaincode package ./abstore.tar.gz \
--path fabric-samples/chaincode/abstore/go/ \
--label abstore_1
```

## Step 7\.3: Install the Package<a name="get-started-chaincode-install-package"></a>

Run the following command to install the chaincode package on the peer node\.

```
docker exec cli peer lifecycle chaincode install abstore.tar.gz
```

## Step 7\.4: Verify the Package<a name="get-started-chaincode-verify-package"></a>

Run the following command to verify that the chaincode package is installed on the peer node\.

```
docker exec cli peer lifecycle chaincode queryinstalled
```

The command returns the following if the package is installed successfully\.

```
Installed chaincodes on peer:
Package ID: MyPackageID, Label: abstore_1
```

## Step 7\.5: Approve the Chaincode<a name="get-started-chaincode-approve"></a>

Run the following commands to approve the chaincode definition for your organization\. Replace *MyPackageID* with the *Package ID* value returned in the previous step [Step 7\.4: Verify the Package](#get-started-chaincode-verify-package)\.

```
export CC_PACKAGE_ID=MyPackageID
docker exec cli peer lifecycle chaincode approveformyorg \
--orderer $ORDERER --tls --cafile /opt/home/managedblockchain-tls-chain.pem \
--channelID mychannel --name mycc --version v0 --sequence 1 --package-id $CC_PACKAGE_ID
```

## Step 7\.6: Check Commit Readiness<a name="get-started-chaincode-check-commit-readiness"></a>

Run the following command to check whether the chaincode definition is ready to be committed on the channel\.

```
docker exec cli peer lifecycle chaincode checkcommitreadiness \
--orderer $ORDERER --tls --cafile /opt/home/managedblockchain-tls-chain.pem \
--channelID mychannel --name mycc --version v0 --sequence 1
```

The command returns `true` if the chaincode is ready to be committed\.

```
Chaincode definition for chaincode 'mycc', version 'v0', sequence '1' on channel 'mychannel' approval status by org:
m-LVQMIJ75CNCUZATGHLDP24HUHM: true
```

## Step 7\.7: Commit the Chaincode<a name="get-started-chaincode-commit"></a>

Run the following command to commit the chaincode definition on the channel\.

```
docker exec cli peer lifecycle chaincode commit \
--orderer $ORDERER --tls --cafile /opt/home/managedblockchain-tls-chain.pem \
--channelID mychannel --name mycc --version v0 --sequence 1
```

## Step 7\.8: Verify the Chaincode<a name="get-started-chaincode-verify-committed"></a>

You might have to wait a minute or two for the commit to propagate to the peer node\. Run the following command to verify that the chaincode is committed\.

```
docker exec cli peer lifecycle chaincode querycommitted \
--channelID mychannel
```

The command returns the following if the chaincode is committed successfully\.

```
Committed chaincode definitions on channel 'mychannel':
Name: mycc, Version: v0, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc
```

## Step 7\.9: Initialize the Chaincode<a name="get-started-chaincode-initialize"></a>

Run the following command to initialize the chaincode\.

```
docker exec cli peer chaincode invoke \
--tls --cafile /opt/home/managedblockchain-tls-chain.pem \
--channelID mychannel \
--name mycc -c '{"Args":["init", "a", "100", "b", "200"]}'
```

The command returns the following when the chaincode is initialized\.

```
2021-12-20 19:23:05.434 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 0ad Chaincode invoke successful. result: status:200
```

## Step 7\.10: Query the Chaincode<a name="get-started-chaincode-query"></a>

You might need to wait a brief moment for the initialization from the previous command to complete before you run the following command to query a value\.

```
docker exec cli peer chaincode query \
--tls --cafile /opt/home/managedblockchain-tls-chain.pem \
--channelID mychannel \
--name mycc -c '{"Args":["query", "a"]}'
```

The command should return the value of `a`, which you initialized with a value of `100`\.

## Step 7\.11: Invoke the Chaincode<a name="get-started-chaincode-invoke"></a>

In the previous steps, you initialized the key `a` with a value of `100` and queried to verify\. Using the `invoke` command in the following example, you subtract `10` from that initial value\.

```
docker exec cli peer chaincode invoke \
--tls --cafile /opt/home/managedblockchain-tls-chain.pem \
--channelID mychannel \
--name mycc -c '{"Args":["invoke", "a", "b", "10"]}'
```

The command returns the following when the chaincode is invoked\.

```
2021-12-20 19:23:22.977 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 0ad Chaincode invoke successful. result: status:200
```

Lastly, you again query the value of `a` using the following command\.

```
docker exec cli peer chaincode query \
--tls --cafile /opt/home/managedblockchain-tls-chain.pem \
--channelID mychannel \
--name mycc -c '{"Args":["query", "a"]}'
```

The command should return the new value `90`\.