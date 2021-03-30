# Develop Hyperledger Fabric Chaincode<a name="managed-blockchain-hyperledger-develop-chaincode"></a>

Smart contracts in Hyperledger Fabric are known as *chaincode*\.
+ For a conceptual overview of chaincode, see [Smart Contracts](https://hyperledger-fabric.readthedocs.io/en/release-1.4/whatis.html#smart-contracts) and [Developing Applications](https://hyperledger-fabric.readthedocs.io/en/release-1.4/developapps/application.html#) in the Hyperledger Fabric documentation\.
+ For links to Hyperledger Fabric SDKs, see [Getting Started](https://hyperledger-fabric.readthedocs.io/en/release-1.4/getting_started.html) in the Hyperledger Fabric documentation\.

## Considerations and Limitations When Developing Chaincode for Managed Blockchain<a name="chaincode-considerations"></a>
+ All Hyperledger Fabric networks on Managed Blockchain support a maximum of 8 channels per network, regardless of network edition\.
+ **Hyperledger Fabric Shims for Node\.js**

  To simplify chaincode development, Managed Blockchain includes versions of the [fabric\-shim library](https://www.npmjs.com/package/fabric-shim)\. This library provides a low\-level chaincode interface between applications, peers, and the Hyperledger Fabric system for chaincode applications developed with Node\.js\. The library version is specified using the `dependencies` object in the `package.json` file bundled with your chaincode\. You can specify a version explicitly or use the semantic versioner \(semver\) for NPM to specify a version range\. The following library versions are available without bundling\.
  + 

**Hyperledger Fabric 1\.4**
    + [1\.4\.0](https://www.npmjs.com/package/fabric-shim/v/1.4.0)
    + [1\.4\.1](https://www.npmjs.com/package/fabric-shim/v/1.4.1)
    + [1\.4\.2](https://www.npmjs.com/package/fabric-shim/v/1.4.2)
    + [1\.4\.4](https://www.npmjs.com/package/fabric-shim/v/1.4.4)
    + [1\.4\.5](https://www.npmjs.com/package/fabric-shim/v/1.4.5)
  + 

**Hyperledger Fabric 1\.2**
    + [1\.2\.0](https://www.npmjs.com/package/fabric-shim/v/1.2.0)
    + [1\.2\.2](https://www.npmjs.com/package/fabric-shim/v/1.2.2)
    + [1\.2\.3](https://www.npmjs.com/package/fabric-shim/v/1.2.3)
    + [1\.2\.4](https://www.npmjs.com/package/fabric-shim/v/1.2.4)

  Dependencies on other versions of fabric\-shim or other library packages require that you bundle them with your chaincode because peer nodes do not have internet access to the NPM repository\.
+ The default limit for the size of a transaction payload is 1MB\. To request a limit increase, create a case using the [AWS Support Center](https://console.aws.amazon.com/support/home#/)\.

**Topics**
+ [Considerations and Limitations When Developing Chaincode for Managed Blockchain](#chaincode-considerations)
+ [Work with Private Data Collections in Hyperledger Fabric on Managed Blockchain](managed-blockchain-hyperledger-create-pdc.md)
+ [Develop Hyperledger Fabric Chaincode Using Java on Amazon Managed Blockchain](java-chaincode.md)