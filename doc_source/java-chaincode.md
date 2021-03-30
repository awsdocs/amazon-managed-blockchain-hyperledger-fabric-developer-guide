# Develop Hyperledger Fabric Chaincode Using Java on Amazon Managed Blockchain<a name="java-chaincode"></a>

Hyperledger Fabric 1\.4 networks on Amazon Managed Blockchain are based on the following components:
+ [Hyperledger Fabric v1\.4\.7](https://github.com/hyperledger/fabric/tree/v1.4.7)
+ [Hyperledger Fabric CA v1\.4\.7](https://github.com/hyperledger/fabric-ca/tree/v1.4.6)
+ [fabric\-chaincode\-java](https://github.com/hyperledger/fabric-chaincode-java/tree/release-1.4)

## Managed Blockchain Dependency on Javaenv Docker Image<a name="java-docker-dependency"></a>

Managed Blockchain uses the `fabric-chaincode-java` package to build the `hyperledger/fabric-javaenv` Docker image\. This image is the base layer of the Java chaincode container\. It has built\-in support to build the Java chaincode using Gradle or Maven at the following versions:
+ gradle@4\.6
+ maven@3\.5\.0

You can verify this by examining the [Docker file in the fabric\-chaincode\-java package](https://github.com/hyperledger/fabric-chaincode-java/blob/v.1.4.6/fabric-chaincode-docker/Dockerfile)\.

The default settings support Gradle or Maven for building Java chaincode\. Once the `javaenv` Docker image is deployed, it is used to provision all new peer nodes and instantiate Java chaincode in those peer nodes\. You cannot modify these settings or update versions on the `javaenv` Docker image because Managed Blockchain manages these\.

## Installing and Running Java Chaincode<a name="java-chaincode-build-and-run"></a>

The following is a high\-level summary of the steps to install and run Java Chaincode using Managed Blockchain with Hyperledger Fabric 1\.4

1. **Marshall dependencies locally** – This step uses Gradle and the Gradle shadow plugin to download dependencies to the client machine and package them for installation to the peer node\.

1. **Install the chaincode** – This step packages the Java chaincode project and copies it to the Hyperledger Fabric peer node\.

1. **Instantiate the chaincode** – This step uses Gradle to resolve the dependencies, access the dependency packages, and build the Java chaincode\. This step also builds the Docker image of this Java chaincode, starts the chaincode container, and instantiates the first object in the ledger\.

1. **Invoke and query the chaincode** – After the chaincode instantiates successfully, you can send query transactions or invoke transactions to modify the ledger\.

![\[Java chaincode steps on Managed Blockchain: build, install, instantiate, and query.\]](http://docs.aws.amazon.com/managed-blockchain/latest/hyperledger-fabric-dev/images/java_chaincode.png)

## Building Java Chaincode with Managed Blockchain: Step\-by\-step<a name="java-chaincode-running"></a>

Java chaincode can depend on other third\-party packages during the **Instantiate chaincode** step\. In a traditional installation of Hyperledger Fabric chaincode built with Java, Gradle automatically resolves dependencies by loading them locally in addition to downloading them from remote repositories\. Managed Blockchain does not allow downloaded dependencies to help ensure the security and reliability of the network\. For this reason, to run Java chaincode, or any chaincode, in a Hyperledger Fabric network on Managed Blockchain, you manually download dependencies to the client machine and add them into the chaincode project\.

The steps in this section demonstrate how to use Gradle to build and deploy a [sample Java chaincode](https://github.com/hyperledger/fabric-samples/tree/v1.4.6/chaincode/chaincode_example02/java) project with fabric\-shim@1\.4\.6 to a Hyperledger Fabric network on Managed Blockchain\. The [Gradle Shadow plugin](https://github.com/johnrengelman/shadow) is needed to combine all jar files into a single jar file\.

### Prerequisites and Assumptions<a name="java-chaincode-prerequisites"></a>

The steps below require that you have the following:
+ A Hyperledger Fabric client machine with access to the internet and access to a member's peer node, ordering service, and CA on an Managed Blockchain network running Hyperledger Fabric v1\.4 or later\. For more information, see the following:
  + [Create a Hyperledger Fabric Blockchain Network on Amazon Managed Blockchain](create-network.md)
  + [Create an Interface VPC Endpoint for Hyperledger Fabric on Amazon Managed Blockchain](managed-blockchain-endpoints.md)
  + [Work with Hyperledger Fabric Peer Nodes on Managed Blockchain](managed-blockchain-hyperledger-peer-nodes.md)
  + [Step 3: Create an Amazon EC2 Instance and Set Up the Hyperledger Fabric Client](get-started-create-client.md)\. Optionally, modify [step 3\.4](get-started-create-client.md#get-started-client-configure-peer-cli) when launching the Docker container to establish CLI variables demonstrated in the install and instantiate steps below\.
+ You must be a Hyperledger Fabric admin to install and instantiate chaincode\. For more information, see [Register and Enroll a Hyperledger Fabric Admin](managed-blockchain-hyperledger-create-admin.md)\.

The working directory of a typical Java chaincode project using Gradle on a Hyperledger Fabric client machine looks similar to the directory structure shown in the following diagram\. The `src` directory includes the chaincode source\. The `build.gradle` file declares all the dependencies and defines the scripted tasks\. The steps below assume this directory structure\.

![\[Java chaincode steps\]](http://docs.aws.amazon.com/managed-blockchain/latest/hyperledger-fabric-dev/images/java_dependencies.png)

### Step 1: Download the Gradle Shadow plugin and add it to the chaincode project<a name="java-ref-gradle-shadow"></a>

Because Managed Blockchain peer nodes don't have internet access, the [Gradle Shadow plugin](https://github.com/johnrengelman/shadow) needs to be downloaded manually to the client and added to the project\. The plugin is downloaded from the [Gradle plugin repository](https://plugins.gradle.org/m2/com/github/jengelman/gradle/plugins/shadow/2.0.4/)\.

1. Create a sub\-directory in the Java chaincode working directory on the Hyperledger Fabric client machine where you want to download the plugin, and then switch to that directory\. The example below uses a directory named `plugin`\.

   ```
   mkdir plugin
   cd plugin
   ```

1. Download the plugin\.

   ```
   wget https://plugins.gradle.org/m2/com/github/jengelman/gradle/plugins/shadow/2.0.4/shadow-2.0.4.jar
   ```

1. Verify that the plugin was downloaded as expected\.

   ```
   ls
   
   shadow-2.0.4.jar
   ```

1. After downloading the shadow plugin jar file, use an editor to modify the `build.gradle` file so that it references the local directory and plugin\. 

   Make sure to remove the `id 'com.github.johnrengelman.shadow' version '2.0.3'` entry from the `plugins` section\.

   The following example shows an excerpt from an edited `build.gradle` file, with *relevant sections emphasized*:

   ```
   buildscript {
       dependencies {
           classpath fileTree(dir: 'plugin', include:['*.jar'])
       }
   }
   
   plugins {
       id 'java'
   }
   
   group 'org.hyperledger.fabric-chaincode-java'
   version '1.0-SNAPSHOT'
   
   sourceCompatibility = 1.8
   
   
   repositories {
       mavenLocal()
       mavenCentral()
   }
   
   dependencies {
       compile group: 'org.hyperledger.fabric-chaincode-java', name: 'fabric-chaincode-shim', version: '1.4.6'
       testCompile group: 'junit', name: 'junit', version: '4.12'
   }
   
   apply plugin: 'com.github.johnrengelman.shadow'
   
   shadowJar {
       baseName = 'chaincode'
       version = null
       classifier = null
   
       manifest {
           attributes 'Main-Class': 'org.hyperledger.fabric.example.SimpleChaincode'
       }
   }
   ```

### Step 2: Add fabric\-chaincode\-shim and the getDeps task to the chaincode project<a name="java-declare-dependencies"></a>

Update `build.gradle` to add [fabric\-chaincode\-shim](https://github.com/hyperledger/fabric-chaincode-java/tree/master/fabric-chaincode-shim) as a dependency in the `dependencies` section\. In addition, reference the task `getDeps` to download dependencies to the `libs` sub\-directory on the client machine\. The following example shows an excerpt from a `build.gradle` file with *relevant sections emphasized*\.

```
buildscript {
    dependencies {
        classpath fileTree(dir: 'plugin', include:['*.jar'])
    }
}

plugins {
    id 'java'
}

group 'org.hyperledger.fabric-chaincode-java'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8

repositories {
    mavenLocal()
    mavenCentral()
}

dependencies {
    compile group: 'org.hyperledger.fabric-chaincode-java', name: 'fabric-chaincode-shim', version: '1.4.6'
    testCompile group: 'junit', name: 'junit', version: '4.12'
}

apply plugin: 'com.github.johnrengelman.shadow'

shadowJar {
    baseName = 'chaincode'
    version = null
    classifier = null

    manifest {
        attributes 'Main-Class': 'org.hyperledger.fabric.example.SimpleChaincode'
    }
}

task getDeps(type: Copy) {
  from sourceSets.main.compileClasspath
  into 'libs/'
}
```

### Step 3: Build the project locally and download dependencies<a name="java-build-chaincode"></a>

Run `gradle build` and `gradle getDeps` as shown in the following examples to build the chaincode locally and download dependencies\. When the `gradle getDeps` task runs, Gradle creates the `lib` sub\-directory on the client machine and downloads dependencies to it\.

1. Build the project using Gradle\.

   ```
   gradle build
   ```

   The command creates output similar to the following example\.

   ```
   > Task :compileJava
   Note: /home/ec2-user/fabric-samples/chaincode/chaincode_example02/java/src/main/java/org/hyperledger/fabric/example/SimpleChaincode.java uses or overrides a deprecated API.
   Note: Recompile with -Xlint:deprecation for details.
   
   Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
   Use '--warning-mode all' to show the individual deprecation warnings.
   See https://docs.gradle.org/6.5/userguide/command_line_interface.html#sec:command_line_warnings
   
   BUILD SUCCESSFUL in 1s
   2 actionable tasks: 1 executed, 1 up-to-date
   ```

1. Download chaincode project dependencies locally\.

   ```
   gradle getDeps
   ```

   The command creates output similar to the following example\.

   ```
   Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
   Use '--warning-mode all' to show the individual deprecation warnings.
   See https://docs.gradle.org/6.5/userguide/command_line_interface.html#sec:command_line_warnings
   
   BUILD SUCCESSFUL in 1s
   1 actionable task: 1 executed
   ```

1. Verify that dependencies were downloaded to the libs sub\-directory\.

   ```
   ls libs
                     
   animal-sniffer-annotations-1.17.jar  grpc-api-1.23.0.jar             jsr305-3.0.2.jar                                  netty-transport-4.1.38.Final.jar
   annotations-4.1.1.4.jar              grpc-context-1.23.0.jar         netty-buffer-4.1.38.Final.jar                     opencensus-api-0.21.0.jar
   bcpkix-jdk15on-1.62.jar              grpc-core-1.23.0.jar            netty-codec-4.1.38.Final.jar                      opencensus-contrib-grpc-metrics-0.21.0.jar
   bcprov-jdk15on-1.62.jar              grpc-netty-1.23.0.jar           netty-codec-http2-4.1.38.Final.jar                perfmark-api-0.17.0.jar
   checker-compat-qual-2.5.2.jar        grpc-protobuf-1.23.0.jar        netty-codec-http-4.1.38.Final.jar                 protobuf-java-3.9.0.jar
   classgraph-4.8.47.jar                grpc-protobuf-lite-1.23.0.jar   netty-codec-socks-4.1.38.Final.jar                protobuf-java-util-3.5.1.jar
   commons-cli-1.4.jar                  grpc-stub-1.23.0.jar            netty-common-4.1.38.Final.jar                     proto-google-common-protos-1.12.0.jar
   commons-logging-1.2.jar              gson-2.7.jar                    netty-handler-4.1.38.Final.jar                    swagger-annotations-2.0.0.jar
   error_prone_annotations-2.3.2.jar    guava-26.0-android.jar          netty-handler-proxy-4.1.38.Final.jar
   fabric-chaincode-protos-1.4.6.jar    j2objc-annotations-1.1.jar      netty-resolver-4.1.38.Final.jar
   fabric-chaincode-shim-1.4.6.jar      javax
   ```

### Step 4: Add dependencies downloaded in the previous step to the project<a name="java-declare-libs-dependencies"></a>

Use a text editor to update the `build.gradle` file again so that it reference the dependencies in the `libs` sub\-directory\. This adds the dependencies that you downloaded in the previous step to the project\.

The following example shows an excerpt from a `build.gradle` file with *relevant sections emphasized*\.

```
buildscript {
    dependencies {
        classpath fileTree(dir: 'plugin', include:['*.jar'])
    }
}

plugins {
    id 'java'
}

group 'org.hyperledger.fabric-chaincode-java'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8

repositories {
    mavenLocal()
    mavenCentral()
}

dependencies {
    compile fileTree(dir:'libs',includes:['*.jar'])
    testCompile group: 'junit', name: 'junit', version: '4.12'
}

apply plugin: 'com.github.johnrengelman.shadow'

shadowJar {
    baseName = 'chaincode'
    version = null
    classifier = null

    manifest {
        attributes 'Main-Class': 'org.hyperledger.fabric.example.SimpleChaincode'
    }
}

task getDeps(type: Copy) {
  from sourceSets.main.compileClasspath
  into 'libs/'
}
```

### Step 5: Use the Hyperledger Fabric CLI to install the chaincode on the peer node<a name="java-install-chaincode"></a>

Use the Hyperledger Fabric CLI to package the Java chaincode project and copy it to the peer node\. The `-l java` option is required to specify that the chaincode is Java\-based\. The path to the Java chaincode working directory is an absolute path, which is different from chaincode written in golang\.

1. Before running the `instantiate`command, establish variables for the Hyperledger Fabric CLI Docker container as shown in the following example\. Alternatively, you can configure a Docker compose file to establish these variables\. For more information, see [Step 3\.4: Configure and Run Docker Compose to Start the Hyperledger Fabric CLI](get-started-create-client.md#get-started-client-configure-peer-cli) in the getting started tutorial\. Replace the values with those appropriate for your network\. Set `CORE_PEER_ADDRESS` to the endpoint of the peer node for which you want to instantiate chaincode, set `CORE_PEER_LOCALMSPID` to the ID of the member that owns the peer node, and set `CORE_PEER_MSPCONFIGPATH` as shown, which is the location of the membership service provider directory on the peer node\.

   ```
   docker exec -e "CORE_PEER_TLS_ENABLED=true" \
   -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/home/managedblockchain-tls-chain.pem
   " \
   -e "CORE_PEER_ADDRESS=nd-6EAJ5VA43JGGNPXOUZP7Y47E4Y.m-K46ICRRXJRCGRNNS4ES4XUUS5A.n-MWY63ZJZU5HGNCMBQER7IN6OIU.managedblockchain.us-east-1.amazonaws.com:30003" \
   -e "CORE_PEER_LOCALMSPID=m-K46ICRRXJRCGRNNS4ES4XUUS5A" \
   -e "CORE_PEER_MSPCONFIGPATH=/opt/home/admin-msp"
   ```

1. Running the Hyperledger Fabric `install` command on the client installs the chaincode on the peer node\. The following example demonstrates an `install` command using the following flags\.
   + The `-o` flag specifies the ordering service endpoint for the member\.
   + The `-l` flag specifies that the chaincode language is `java`\.
   + The `--cafile` flag specifies the location of the certificate for the ordering service that you copied when you set up the Hyperledger Fabric admin\. For more information, see [step 4\.1](get-started-enroll-admin.md#get-started-enroll-member-create-cert) in the [Getting Started](managed-blockchain-get-started-tutorial.md) tutorial\.
   + The `--tls` flag specifies that communication with the ordering service uses TLS\.
   + The `-p` flag specifies the location of the chaincode on the client machine\. When installing Java chaincode, this must be an absolute path\.
   + The `-n` and `-v` options establish the name and version of the chaincode\. You'll reference this name and version when you instantiate the chaincode in the next step\. 

   For more information about options, see [peer chaincode install](https://hyperledger-fabric.readthedocs.io/en/release-1.4/commands/peerchaincode.html#peer-chaincode-install) in Hyperledger Fabric documentation\.

   ```
   cli peer chaincode install \
   -o orderer.n-MWY63ZJZU5HGNCMBQER7IN6OIU.managedblockchain.amazonaws.com:30001 \
   -l java \
   --cafile /opt/home/managedblockchain-tls-chain.pem \
   --tls -p  /opt/home/fabric-samples/chaincode/chaincode_example02/java -v MyCCVerNumber -n MyJavaChaincodeName
   ```

### Step 6:Use the Hyperledger Fabric CLI to instantiate the chaincode<a name="java-instantiate-chaincode"></a>

Running the Hyperledger Fabric CLI `instantiate` command directs the peer node to build the Java chaincode, create the chaincode image, start the chaincode container, and instantiate object values\. Similar to a previous example, the command should have the `-l java` option specified\. 

1. Before running the `install`command, establish variables for the Hyperledger Fabric CLI Docker container as shown in the following example\. Alternatively, you can configure a Docker compose file to establish these variables\. For more information, see [Step 3\.4: Configure and Run Docker Compose to Start the Hyperledger Fabric CLI](get-started-create-client.md#get-started-client-configure-peer-cli) in the getting started tutorial\. Set `CORE_PEER_ADDRESS` to the endpoint of the peer node for which you want to instantiate chaincode, set `CORE_PEER_LOCALMSPID` to the ID of the member that owns the peer node, and set `CORE_PEER_MSPCONFIGPATH` as shown, which is the location of the membership service provider directory on the peer node\.

   ```
   docker exec -e "CORE_PEER_TLS_ENABLED=true" \
   -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/home/managedblockchain-tls-chain.pem" \
   -e "CORE_PEER_ADDRESS=nd-6EAJ5VA43JGGNPXOUZP7Y47E4Y.m-K46ICRRXJRCGRNNS4ES4XUUS5A.n-MWY63ZJZU5HGNCMBQER7IN6OIU.managedblockchain.us-east-1.amazonaws.com:30003" \
   -e "CORE_PEER_LOCALMSPID=m-K46ICRRXJRCGRNNS4ES4XUUS5A" \
   -e "CORE_PEER_MSPCONFIGPATH=/opt/home/admin-msp"
   ```

1. Running the Hyperledger Fabric CLI `instantiate` command on the client instantiates the chaincode on the peer\. The following example demonstrates an `instantiate` command\.
   + The `-o` flag specifies the ordering service endpoint for the member\.
   + The `-l` flag specifies that the chaincode language is `java`\.
   + The `--cafile` flag specifies the location of the certificate for the ordering service that you copied when you set up the Hyperledger Fabric admin\. For more information, see [step 4\.1](get-started-enroll-admin.md#get-started-enroll-member-create-cert) in the [Getting Started](managed-blockchain-get-started-tutorial.md) tutorial\.
   + The `--tls` flag specifies that communication with the ordering service uses TLS\.
   + The `-C` flag specifies the channel on which to instantiate the chaincode\.
   + The `-c` flag specifies the constructor message in JSON format that initializes object values for [the sample chaincode](https://github.com/hyperledger/fabric-samples/blob/v1.4.6/chaincode/chaincode_example02/java/src/main/java/org/hyperledger/fabric/example/SimpleChaincode.java) that you installed on the peer node in the previous step\. `a` is set to a value of `100` and `b` is set to a value of `200`\.
   + The `-n` and `-v` options specify the name and version of the chaincode to be instantiated, which you established when installing the chaincode in the previous step\.

   For more information about options, see [peer chaincode instantiate](https://hyperledger-fabric.readthedocs.io/en/release-2.0/commands/peerchaincode.html#peer-chaincode-instantiate) in Hyperledger Fabric documentation\.

   ```
   cli peer chaincode instantiate \
   -o orderer.n-MWY63ZJZU5HGNCMBQER7IN6OIU.managedblockchain.amazonaws.com:30001 \
   -l java \
   --cafile /opt/home/managedblockchain-tls-chain.pem --tls \
   -C my-hlf-channel-id \
   -c '{"Args":["init", "a","100","b","200"]}' -v v0 -n javacc
   ```

You can verify that Gradle is using local dependencies by examining the build logs\. The following example shows that, as expected, Gradle did not download jar files during build\.

```
2020-06-26T07:42:37.371-07:00 Gradle build
2020-06-26T07:42:37.371-07:00 Starting a Gradle Daemon, 1 incompatible and 1 stopped Daemons could not be reused, use --status for details
2020-06-26T07:42:37.371-07:00 :compileJavaNote: /tmp/tmp.xANvpIqhV0/src/main/java/org/hyperledger/fabric/example/SimpleChaincode.java uses or overrides a deprecated API.
2020-06-26T07:42:37.371-07:00 Note: Recompile with -Xlint:deprecation for details.
2020-06-26T07:42:37.371-07:00 :processResources NO-SOURCE
2020-06-26T07:42:37.371-07:00 :classes
2020-06-26T07:42:37.371-07:00 :jar
2020-06-26T07:42:37.371-07:00 :assemble
2020-06-26T07:42:37.371-07:00 :check
2020-06-26T07:42:37.371-07:00 :build
2020-06-26T07:42:37.371-07:00 :shadowJar
2020-06-26T07:42:37.371-07:00 Deprecated Gradle features were used in this build, making it incompatible with Gradle 5.0.
2020-06-26T07:42:37.371-07:00 See https://docs.gradle.org/4.6/userguide/command_line_interface.html#sec:command_line_warnings
2020-06-26T07:42:37.371-07:00 BUILD SUCCESSFUL in 26s
2020-06-26T07:42:37.371-07:00 3 actionable tasks: 3 executed
```

If Gradle had downloaded the dependencies, the file would have references to plugin locations as shown in the following example\.

```
2020-06-22T12:42:20.417-07:00 Gradle build
2020-06-22T12:42:20.417-07:00 Starting a Gradle Daemon, 1 incompatible and 1 stopped Daemons could not be reused, use --status for details
2020-06-22T12:42:20.417-07:00 Download https://plugins.gradle.org/m2/com/github/johnrengelman/shadow/com.github.johnrengelman.shadow.gradle.plugin/2.0.4/com.github.johnrengelman.shadow.gradle.plugin-2.0.4.pom
2020-06-22T12:42:20.417-07:00 Download https://plugins.gradle.org/m2/com/github/jengelman/gradle/plugins/shadow/2.0.4/shadow-2.0.4.pom
2020-06-22T12:42:20.417-07:00 Download https://plugins.gradle.org/m2/org/codehaus/groovy/groovy-backports-compat23/2.4.4/groovy-backports-compat23-2.4.4.pom
2020-06-22T12:42:20.417-07:00 Download https://plugins.gradle.org/m2/com/github/jengelman/gradle/plugins/shadow/2.0.4/shadow-2.0.4.jar
2020-06-22T12:42:20.417-07:00 Download https://plugins.gradle.org/m2/org/codehaus/groovy/groovy-backports-compat23/2.4.4/groovy-backports-compat23-2.4.4.jar
2020-06-22T12:42:20.417-07:00 Download https://hyperledger.jfrog.io/hyperledger/fabric-maven/org/hyperledger/fabric-chaincode-java/fabric-chaincode-shim/maven-metadata.xml
2020-06-22T12:42:20.417-07:00 Download https://hyperledger.jfrog.io/hyperledger/fabric-maven/org/hyperledger/fabric-chaincode-java/fabric-chaincode-shim/1.4.5/fabric-chaincode-shim-1.4.5.pom
2020-06-22T12:42:20.417-07:00 Download https://jcenter.bintray.com/org/hyperledger/fabric-chaincode-java/fabric-chaincode-shim/maven-metadata.xml
2020-06-22T12:42:20.417-07:00 Download https://jcenter.bintray.com/org/hyperledger/fabric-chaincode-java/fabric-chaincode-shim/1.4.6/fabric-chaincode-shim-1.4.6.pom
2020-06-22T12:42:20.417-07:00 Download https://jcenter.bintray.com/com/owlike/genson/1.5/genson-1.5.pom
2020-06-22T12:42:20.417-07:00 Download https://jcenter.bintray.com/com/owlike/genson-parent/1.5/genson-parent-1.5.pom
2020-06-22T12:42:20.417-07:00 Download https://jcenter.bintray.com/org/hyperledger/fabric-chaincode-java/fabric-chaincode-protos/1.4.6/fabric-chaincode-protos-1.4.6.pom
2020-06-22T12:42:20.417-07:00 Download https://jcenter.bintray.com/io/grpc/grpc-core/maven-metadata.xml
2020-06-22T12:42:20.417-07:00 Download https://jcenter.bintray.com/io/netty/netty-codec-http2/maven-metadata.xml
2020-06-22T12:42:20.417-07:00 Download https://jcenter.bintray.com/io/grpc/grpc-api/maven-metadata.xml
2020-06-22T12:42:20.417-07:00 Download https://jcenter.bintray.com/com/google/errorprone/error_prone_annotations/maven-metadata.xml
2020-06-22T12:42:20.417-07:00 Download https://jcenter.bintray.com/com/owlike/genson/1.5/genson-1.5.jar
2020-06-22T12:42:20.417-07:00 Download https://jcenter.bintray.com/org/hyperledger/fabric-chaincode-java/fabric-chaincode-shim/1.4.6/fabric-chaincode-shim-1.4.6.jar
2020-06-22T12:42:20.417-07:00 Download https://jcenter.bintray.com/org/hyperledger/fabric-chaincode-java/fabric-chaincode-protos/1.4.6/fabric-chaincode-protos-1.4.6.jar
...
2020-06-22T12:42:20.417-07:00 :compileJava
2020-06-22T12:42:20.417-07:00 :processResources NO-SOURCE
2020-06-22T12:42:20.417-07:00 :classes
2020-06-22T12:42:20.417-07:00 :jar
2020-06-22T12:42:20.417-07:00 :assemble
2020-06-22T12:42:20.417-07:00 :check
2020-06-22T12:42:20.417-07:00 :build
2020-06-22T12:42:20.417-07:00 :shadowJar
2020-06-22T12:42:20.417-07:00 BUILD SUCCESSFUL in 41s
2020-06-22T12:42:20.417-07:00 3 actionable tasks: 3 executed
```

### Step 7: Invoke the sample chaincode to perform a transaction<a name="java-chaincode-invoke"></a>

Precede the following example command with the variable overrides for the CLI container as shown in the previous example\.

The following example invokes the sample chaincode that you instantiated\. The parameters are the same as the previous example, except for the JSON constructor\. This constructor invokes the [the sample chaincode](https://github.com/hyperledger/fabric-samples/blob/v1.4.6/chaincode/chaincode_example02/java/src/main/java/org/hyperledger/fabric/example/SimpleChaincode.java) to transfer a value of 10 from `a` to `b`\.

```
cli peer chaincode invoke \
-o orderer.n-MWY63ZJZU5HGNCMBQER7IN6OIU.managedblockchain.amazonaws.com:30001 \
--cafile /opt/home/managedblockchain-tls-chain.pem --tls \
-C one-org-channel-java \
-c '{"Args":["invoke","a", "b", "10"]}' -n javacc
```

The command should print output similar to the following\.

```
2020-06-26 23:49:43.895 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 003 Chaincode invoke successful. result: status:200 message:"invoke finished successfully" payload:"a: 90 b: 210
```

### Step 8: Invoke the sample chaincode to query<a name="java-chaincode-query"></a>

Precede the following command with the variable overrides for the CLI container as shown in the previous example\.

The following example queries the ledger for the value attributed to `a`\. The query should print `90` as output\.

```
cli peer chaincode query \
-o orderer.n-MWY63ZJZU5HGNCMBQER7IN6OIU.managedblockchain.amazonaws.com:30001 \
--cafile /opt/home/managedblockchain-tls-chain.pem --tls \
-C one-org-channel-java \
-c '{"Args":["query","a"]}' -n javacc
```