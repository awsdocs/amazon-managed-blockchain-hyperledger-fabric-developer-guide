# Register and Enroll a Hyperledger Fabric Admin<a name="managed-blockchain-hyperledger-create-admin"></a>

Only identities who are admins within a Hyperledger Fabric member can install, instantiate, and query chaincode\. 

Creating an admin in Hyperledger Fabric is a three\-step process\.

1. You *register* the identity with the Hyperledger Fabric CA\. Registering stores the user name and password in the CA database as an admin\.

1. After you register, you *enroll* the identity\. This sends the CA a Certificate Signing Request \(CSR\)\. The CA validates that the identity is registered and otherwise valid, and returns a signed certificate\. The certificate is stored in the Hyperledger Fabric client machine's local Membership Service Provider \(MSP\)\.

1. You then copy the certificate to the `admincerts` subdirectory, and the certificate validates the role of the identity as an admin\. Similarly, the CA updates the local MSP for the member's peer nodes and the ordering service so that the admin is recognized\.

 For more information, see [Fabric CA User's Guide](https://hyperledger-fabric-ca.readthedocs.io/en/release-1.4/users-guide.html) and [Membership](https://hyperledger-fabric.readthedocs.io/en/release-2.2/membership/membership.html) in Hyperledger Fabric documentation\.

When you first create a member in a Hyperledger Fabric network on Managed Blockchain, you specify the member's first user\. Managed Blockchain registers the identity of this user automatically with the Hyperledger Fabric CA\. This is called a *bootstrap identity*\. Even though the identity is registered, the remaining steps need to be performed\. The identity must then enroll itself as an admin and certificates must be updated\. After the steps are complete, the identity can install and instantiate chaincode and can be used to enroll additional identities as admins\.

After you enroll an identity as an admin, it may take a minute or two until the identity is able to use the admin certificate to perform tasks\.

**Important**  
Managed Blockchain does not support revoking user certificates\. After an admin is created, the admin persists for the life of the member\.

To register and enroll an identity as an admin, you must have the following:
+ The member CA endpoint
+ The user name and password either of the bootstrap identity or of an admin with permissions to register and enroll identities
+ A valid certificate file and the path to the MSP directory of your identity, which will register the new administrator

## Registering an Admin<a name="managed-blockchain-hyperledger-admin-register"></a>

The following example uses a [Fabric\-CA Client CLI](https://hyperledger-fabric-ca.readthedocs.io/en/release-1.4/clientcli.html) `register` command to register an admin with these options:
+ `--url` specifies the endpoint of the CA along with an existing user name of an admin with permissions to register, such as the bootstrap identity\. The example uses a user name of `AdminUser` with password `Password123`\.
+ `--id.name` and `--id.secret` parameters establish the user name and password for the new admin\.
+ `--id.type` is set to `user` and `--id.affiliation` is set to the member name to which the admins belong\. The example member name is `org1`\. 
+ `--id.attrs` is set to `'hf.admin=true'`\. This is a property specific to Managed Blockchain that registers the identity as an admin\.
+ The `--tls.certfiles` option specifies the location and file name of the Managed Blockchain TLS certificate that you copied from Amazon S3 \(see [Step 5\.1: Create the Certificate File](get-started-enroll-admin.md#get-started-enroll-member-create-cert)\)\.
+ `--mspdir` specifies the MSP directory on the local machine where certificates are saved\. The example uses `/home/ec2-user/admin-msp`\.

```
fabric-ca-client register \
–-url https://AdminUser:Password123@ca.m-K46ICRRXJRCGRNNS4ES4XUUS5A.n-MWY63ZJZU5HGNCMBQER7IN6OIU.managedblockchain.us-east-1.amazonaws.com:30002 \
--id.name AdminUser2 --id.secret Password456 \
–-id.type user --id.affiliation org1 \
--id.attrs ‘hf.Admin=true’ --tls.certfiles /home/ec2-user/managedblockchain-tls-chain.pem \
--mspdir /home/ec2-user/admin-msp
```

## Enrolling an Admin<a name="admin-hyperledger-admin-enroll"></a>

After registering an identity as an admin, or creating a member along with the bootstrap identity, you can use the [Fabric\-CA Client CLI](https://hyperledger-fabric-ca.readthedocs.io/en/release-1.4/clientcli.html) `enroll` command to enroll that identity as an admin\. This is shown in the following example using these options:
+ `-u` \(an alternative for `--url`\) specifies the endpoint of the CA along with the user name and password of the identity that you are enrolling\.
+ `tls.certfiles` specifies the location and file name of the Managed Blockchain TLS certificate that you copied from Amazon S3 \(see [Step 5\.1: Create the Certificate File](get-started-enroll-admin.md#get-started-enroll-member-create-cert)\)\.
+ `-M` \(an alternative for `--mspdir`\) specifies the MSP directory on the local machine where certificates are saved\. The example uses `/home/ec2-user/admin-msp`\.

```
fabric-ca-client enroll \
-u https://AdminUser:Password123@ca.m-K46ICRRXJRCGRNNS4ES4XUUS5A.n-MWY63ZJZU5HGNCMBQER7IN6OIU.managedblockchain.us-east-1.amazonaws.com:30002 \
--tls.certfiles /home/ec2-user/managedblockchain-tls-chain.pem \
-M /home/ec2-user/admin-msp
```

## Copying the Admin Certificate<a name="managed-blockchain-hyperledger-admin-copy-cert"></a>

After you enroll the admin, copy the certificates from the `signcerts` directory to the `admincerts` directory as shown in the following example\. The MSP directory `/home/ec2-user/admin-msp` is used in the example, and the example assumes that you are running the command in the `/home/ec2-user` directory\.

```
cp -r admin-msp/signcerts admin-msp/admincerts
```