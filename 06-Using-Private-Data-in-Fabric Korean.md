# 06. Using Private Data in Fabric

This tutorial will demonstrate the use of collections to provide storage and retrieval of private data on the blockchain network for authorized peers of organizations.

이 튜토리얼은 승인 된 조직의 피어가 컬렉션을 사용하여 블록체인 네트워크에서 private data를 저장하고 검색하는 방법을 설명합니다.

The information in this tutorial assumes knowledge of private data stores and their use cases. For more information, check out [Private data](https://hyperledger-fabric.readthedocs.io/en/latest/private-data/private-data.html).

이 튜토리얼은 private data 저장소 및 해당 사용사례에 대한 지식이 있음을 전제로 합니다. 자세한 내용은 [Private data](https://hyperledger-fabric.readthedocs.io/en/latest/private-data/private-data.html)를 확인하십시오.

**Note**

These instructions use the new Fabric chaincode lifecycle introduced in the Fabric v2.0 Alpha release. If you would like to use the previous lifecycle model to use private data with chaincode, visit the v1.4 version of the [Using Private Data in Fabric tutorial](https://hyperledger-fabric.readthedocs.io/en/release-1.4/private_data_tutorial.html).

**주기**

이 자료는 Fabric v2.0 Alpha에 도입 된 새로운 fabric 체인코드 라이프 사이클을 사용합니다. 이전 라이프 사이클 모델을 사용하여 체인코드와 함께 private data를 사용하려면 v1.4 버전의 [Using Private Data in Fabric tutorial](https://hyperledger-fabric.readthedocs.io/en/release-1.4/private_data_tutorial.html)을 확인하십시오.

The tutorial will take you through the following steps to practice defining, configuring and using private data with Fabric:

이 튜토리얼에서는 Fabric에서 private data를 정의, 구성 및 사용하는 방법을 연습하기 위해 다음 단계를 안내합니다.

1. [Build a collection definition JSON file](https://hyperledger-fabric.readthedocs.io/en/latest/private_data_tutorial.html#pd-build-json)
2. [Read and Write private data using chaincode APIs](https://hyperledger-fabric.readthedocs.io/en/latest/private_data_tutorial.html#pd-read-write-private-data)
3. [Install and define a chaincode with a collection](https://hyperledger-fabric.readthedocs.io/en/latest/private_data_tutorial.html#pd-install-define-cc)
4. [Store private data](https://hyperledger-fabric.readthedocs.io/en/latest/private_data_tutorial.html#pd-store-private-data)
5. [Query the private data as an authorized peer](https://hyperledger-fabric.readthedocs.io/en/latest/private_data_tutorial.html#pd-query-authorized)
6. [Query the private data as an unauthorized peer](https://hyperledger-fabric.readthedocs.io/en/latest/private_data_tutorial.html#pd-query-unauthorized)
7. [Purge Private Data](https://hyperledger-fabric.readthedocs.io/en/latest/private_data_tutorial.html#pd-purge)
8. [Using indexes with private data](https://hyperledger-fabric.readthedocs.io/en/latest/private_data_tutorial.html#pd-indexes)
9. [Additional resources](https://hyperledger-fabric.readthedocs.io/en/latest/private_data_tutorial.html#pd-ref-material)

This tutorial will use the [marbles private data sample](https://github.com/hyperledger/fabric-samples/tree/master/chaincode/marbles02_private) — running on the Building Your First Network (BYFN) tutorial network — to demonstrate how to create, deploy, and use a collection of private data. The marbles private data sample will be deployed to the [Building Your First Network](https://hyperledger-fabric.readthedocs.io/en/latest/build_network.html)(BYFN) tutorial network. You should have completed the task [Install Samples, Binaries and Docker Images](https://hyperledger-fabric.readthedocs.io/en/latest/install.html); however, running the BYFN tutorial is not a prerequisite for this tutorial. Instead the necessary commands are provided throughout this tutorial to use the network. We will describe what is happening at each step, making it possible to understand the tutorial without actually running the sample.

이 튜토리얼은 private data의 수집, 생성 및 사용방법을 설명하기 위해 BYFN (Build Your First Network) 튜토리얼 네트워크에서 실행하는 [Marbles private data sample](https://github.com/hyperledger/fabric-samples/tree/master/chaincode/marbles02_private)을 사용합니다. marbles private data 샘플은 [Building Your First Network](https://hyperledger-fabric.readthedocs.io/en/latest/build_network.html)(BYFN) 튜토리얼 네트워크에 배포됩니다. [Install Samples, Binaries and Docker Images](https://hyperledger-fabric.readthedocs.io/en/latest/install.html) 작업이 완료되어야 하지만 BYFN를 실행하는 것이 이 튜토리얼의 전제조건은 아닙니다. 대신 이 튜토리얼에서는 네트워크를 사용하는 데 필요한 명령이 제공됩니다. 각 단계에서 어떤 일이 일어나고 있는지 설명하여 실제로 샘플을 실행하지 않고도 튜토리얼을 이해할 수 있습니다.

# **Build a collection definition JSON file**

The first step in privatizing data on a channel is to build a collection definition which defines access to the private data.

채널에서 데이터를 개인화하는 첫 번째 단계는 private data에 대한 액세스를 정의하는 컬렉션 정의를 작성하는 것입니다.

The collection definition describes who can persist data, how many peers the data is distributed to, how many peers are required to disseminate the private data, and how long the private data is persisted in the private database. Later, we will demonstrate how chaincode APIs `PutPrivateData`and `GetPrivateData` are used to map the collection to the private data being secured.

컬렉션 정의는 데이터를 유지할 수있는 사람, 데이터가 분산되는 피어 수, private data를 배포하는데 필요한 피어 수 및 private DB에서 private data가 유지되는 기간을 설명합니다. 나중에 체인코드 API`PutPrivateData` 및`GetPrivateData`를 사용하여 컬렉션을 보안성 있는 private data에 매핑하는 방법을 설명합니다.

A collection definition is composed of the following properties:

컬렉션 정의는 다음 속성으로 구성됩니다

- `name`: Name of the collection.
- `name` : 컬렉션 이름

- `policy`: Defines the organization peers allowed to persist the collection data.
- `policy` : 컬렉션 데이터를 유지할 수있는 조직 피어를 정의합니다.

- `requiredPeerCount`: Number of peers required to disseminate the private data as a condition of the endorsement of the chaincode
- `requiredPeerCount` : 체인코드의 보증 조건으로 private data를 배포하는데 필요한 피어 수

- `maxPeerCount`: For data redundancy purposes, the number of other peers that the current endorsing peer will attempt to distribute the data to. If an endorsing peer goes down, these other peers are available at commit time if there are requests to pull the private data.
- `maxPeerCount` : 데이터 중복을 위해 현재 보증하는 피어가 데이터를 배포하려고 시도하는 다른 피어의 수입니다. 보증피어가 다운되면 private data를 가져 오기 위한 요청이 있는 경우 커밋 시 다른 피어를 사용할 수 있습니다.

- `blockToLive`: For very sensitive information such as pricing or personal information, this value represents how long the data should live on the private database in terms of blocks. The data will live for this specified number of blocks on the private database and after that it will get purged, making this data obsolete from the network. To keep private data indefinitely, that is, to never purge private data, set the `blockToLive` property to `0`.

- `blockToLive` : 가격 또는 개인 정보와 같은 매우 민감한 정보의 경우 이 값은 데이터가 private DB에 블록 단위로 얼마나 오래 있어야하는지 나타냅니다. 데이터는 private DB에서 이 지정된 수의 블록에 대해 존재하며 그 후에는 데이터가 제거되어 네트워크에서 사용되지 않습니다. private data를 무기한으로 유지하려면, 즉 절대로 삭제하지 않으려면 `blockblockToLive` 속성을 `0`으로 설정하십시오.

- `memberOnlyRead`: a value of `true` indicates that peers automatically enforce that only clients belonging to one of the collection member organizations are allowed read access to private data.
- `memberOnlyRead` : `true` 값은 피어가 컬렉션 멤버 조직 중 하나에 속하는 클라이언트만 private data에 대한 읽기 액세스를 허용하도록 자동으로 적용함을 나타냅니다.


To illustrate usage of private data, the marbles private data example contains two private data collection definitions: `collectionMarbles` and `collectionMarblePrivateDetails`. The `policy` property in the`collectionMarbles` definition allows all members of the channel (Org1 and Org2) to have the private data in a private database. The`collectionMarblesPrivateDetails` collection allows only members of Org1 to have the private data in their private database.

private data의 사용을 설명하기 위해 marbles private data 예제에는 두 개의 PDC 정의, 즉 `collectionMarbles` 및 `collectionMarblePrivateDetails`가 있습니다. `collectionMarbles` 정의의 `policy` 속성을 사용하면 채널(Org1 및 Org2)의 모든 구성원이 private DB에 private data를 가질 수 있습니다. `collectionMarblesPrivateDetails` 컬렉션을 사용하면 Org1의 구성원만 private DB에 개인 데이터를 가질 수 있습니다.


For more information on building a policy definition refer to the [Endorsement policies](https://hyperledger-fabric.readthedocs.io/en/latest/endorsement-policies.html) topic.

정책 정의의 자세한 내용은 [Endorsement policies](https://hyperledger-fabric.readthedocs.io/en/latest/endorsement-policies.html)를 참고하십시오.

```
// collections_config.json
[ 
 {  "name": "collectionMarbles",
    "policy": "OR('Org1MSP.member', 'Org2MSP.member')",
    "requiredPeerCount": 0,
    "maxPeerCount": 3,
    "blockToLive":1000000,
    "memberOnlyRead": true
 },
 {  "name": "collectionMarblePrivateDetails",
    "policy": "OR('Org1MSP.member')",
    "requiredPeerCount": 0,
    "maxPeerCount": 3,
    "blockToLive":3,
    "memberOnlyRead": true
 }
]
```


The data to be secured by these policies is mapped in chaincode and will be shown later in the tutorial.

이러한 정책으로 보호 할 데이터는 체인코드로 매핑되며 이 튜토리얼의 뒷부분에서 설명합니다.


This collection definition file is deployed when the chaincode definition is committed to the channel using the [peer lifecycle chaincode commit command](http://hyperledger-fabric.readthedocs.io/en/latest/commands/peerchaincode.html#peer-chaincode-instantiate). More details on this process are provided in Section 3 below.

이 컬렉션 정의 파일은 체인코드 정의가 [peer lifecycle chaincode commit command](http://hyperledger-fabric.readthedocs.io/en/latest/commands/peerchaincode.html#peer-chaincode-instantiate)을 사용하여 채널에 커밋 될 때 배포됩니다.


# **Read and Write private data using chaincode APIs**

The next step in understanding how to privatize data on a channel is to build the data definition in the chaincode. The marbles private data sample divides the private data into two separate data definitions according to how the data will be accessed.

채널에서 데이터를 개인화하는 방법을 이해하는 다음 단계는 체인코드에서 데이터 정의를 작성하는 것입니다. marbles private data 샘플은 private data를 데이터 액세스 방법에 따라 두 개의 개별 데이터 정의로 나눕니다.

```
*// Peers in Org1 and Org2 will have this private data in a side database***type** marble **struct** {
    ObjectType **string** `json:"docType"`
    Name **string** `json:"name"`
    Color **string** `json:"color"`
    Size **int** `json:"size"`
    Owner **string** `json:"owner"`
}
*// Only peers in Org1 will have this private data in a side database***type** marblePrivateDetails **struct** {
    ObjectType **string** `json:"docType"`
    Name **string** `json:"name"`
    Price **int** `json:"price"`
}
```

Specifically access to the private data will be restricted as follows:

private data 액세스는 다음과 같이 제한됩니다:


- `name, color, size, and owner` will be visible to all members of the channel (Org1 and Org2)
- `price` only visible to members of Org1
- `name, color, size, and owner`는 채널의 모든 멤버에게 보여줍니다(Org1과 Org2)
- `price`는 Org1 멤버에게만 보여줍니다.


Thus two different sets of private data are defined in the marbles private data sample. The mapping of this data to the collection policy which restricts its access is controlled by chaincode APIs. Specifically, reading and writing private data using a collection definition is performed by calling `GetPrivateData()` and `PutPrivateData()`, which can be found [here](https://github.com/hyperledger/fabric/blob/master/core/chaincode/shim/interfaces.go#L179).

따라서 두 개의 서로 다른 private data 세트가 marbles private data 샘플에 정의됩니다. 이 데이터를 액세스 제한 콜렉션 정책으로의 맵핑은 체인코드 API에 의해 제어됩니다. 특히, 컬렉션 정의를 사용하여 private data를 읽고 쓰는 것은 `GetPrivateData()` 및 `PutPrivateData()`를 호출하여 수행됩니다. [here](https://github.com/hyperledger/fabric/blob/master/core/chaincode/shim/interfaces.go#L179)에서 확인할 수 있습니다.


The following diagrams illustrate the private data model used by the marbles private data sample.

다음 그림은 marbles private data 샘플에서 사용하는 모델을 보여줍니다.


> 

![](https://hyperledger-fabric.readthedocs.io/en/latest/_images/SideDB-org1.png)

![](https://hyperledger-fabric.readthedocs.io/en/latest/_images/SideDB-org2.png)

# **Reading collection data**

Use the chaincode API `GetPrivateData()` to query private data in the database. `GetPrivateData()` takes two arguments, the **collection name** and the data key. Recall the collection `collectionMarbles` allows members of Org1 and Org2 to have the private data in a side database, and the collection `collectionMarblePrivateDetails` allows only members of Org1 to have the private data in a side database. For implementation details refer to the following two [marbles private data functions](https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02_private/go/marbles_chaincode_private.go):

체인 코드 API `GetPrivateData()`를 사용하여 데이터베이스의 private data를 쿼리하십시오. `GetPrivateData()`는 **컬렉션 이름**과 data key라는 두 개의 인수를 취합니다. `collectionMarbles` 컬렉션은 Org1 및 Org2의 멤버가 side DB에 private data를 보유 할 수 있게하고`collectionMarblePrivateDetails` 컬렉션은 Org1의 멤버만 side DB에 private data를 보유 할 수 있도록 합니다. 구현에 대한 자세한 내용은 다음 두 가지의 [marbles private data functions](https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02_private/go/marbles_chaincode_private.go)를 참조하십시오:

- readMarble for querying the values of the `name, color, size and owner` attributesread
- `name, color, size and owner`를 쿼리하기 위한 readMarble 
- MarblePrivateDetails for querying the values of the `price` attribute
- `price`를 쿼리하기 위한 MarblePrivateDetails

When we issue the database queries using the peer commands later in this tutorial, we will call these two functions.

이 튜토리얼의 뒷부분에서 피어 명령을 사용하여 데이터베이스 쿼리를 할 때이 두 함수를 호출합니다.

# **Writing private data**

Use the chaincode API `PutPrivateData()` to store the private data into the private database. The API also requires the name of the collection. Since the marbles private data sample includes two different collections, it is called twice in the chaincode:

private DB에 private data를 저장하려면 체인코드 API `PutPrivateData ()`를 사용하십시오. API에는 컬렉션 이름도 필요합니다. marbles private data 샘플에는 서로 다른 두 가지 컬렉션이 포함되어 있으므로 체인코드에서 두 번 호출됩니다.

1. Write the private data `name, color, size and owner` using the collection named `collectionMarbles`.
1. 컬렉션 이름 `collectionMarbles`를 사용하여 private data `name, color, size and owner` 쓰기.
2. Write the private data `price` using the collection named`collectionMarblePrivateDetails`.
2. 컬렉션 이름 `collectionMarblePrivateDetails`를 사용하여 private data `price` 쓰기.

For example, in the following snippet of the `initMarble` function,`PutPrivateData()` is called twice, once for each set of private data.

```
*// ==== Create marble object, marshal to JSON, and save to state ====*marble **:=** **&**marble{ ObjectType: "marble", Name: marbleInput.Name, Color: marbleInput.Color, Size: marbleInput.Size, Owner: marbleInput.Owner, } marbleJSONasBytes, err **:=** json.Marshal(marble) **if** err **!=** **nil** { **return** shim.Error(err.Error()) } *// === Save marble to state ===*err = stub.PutPrivateData("collectionMarbles", marbleInput.Name, marbleJSONasBytes) **if** err **!=** **nil** { **return** shim.Error(err.Error()) } *// ==== Create marble private details object with price, marshal to JSON, and save to state ====*marblePrivateDetails **:=** **&**marblePrivateDetails{ ObjectType: "marblePrivateDetails", Name: marbleInput.Name, Price: marbleInput.Price, } marblePrivateDetailsBytes, err **:=** json.Marshal(marblePrivateDetails) **if** err **!=** **nil** { **return** shim.Error(err.Error()) } err = stub.PutPrivateData("collectionMarblePrivateDetails", marbleInput.Name, marblePrivateDetailsBytes) **if** err **!=** **nil** { **return** shim.Error(err.Error()) }
```

To summarize, the policy definition above for our `collection.json` allows all peers in Org1 and Org2 to store and transact with the marbles private data `name, color, size, owner` in their private database. But only peers in Org1 can store and transact with the `price`private data in its private database.

요약하면,`collection.json`에 대한 위의 정책 정의를 통해 Org1 및 Org2의 모든 피어는 private DB에 marbles private data `name, color, size, owner`를 저장하고 트랜잭션 할 수 있습니다. 그러나 Org1의 피어만이 private DB에 private data `price`를 저장하고 트랜잭션 할 수 있습니다.

As an additional data privacy benefit, since a collection is being used, only the private data hashes go through orderer, not the private data itself, keeping private data confidential from orderer.

컬렉션을 통한 추가적인 데이터 개인정보보호 장점으로는, private data 자체가 아닌 private data 해시만 orderer를 통과하므로 orderer에게 private data의 기밀성을 유지합니다.

# **Start the network**

Now we are ready to step through some commands which demonstrate how to use private data.

이제 private data 사용을 증명하는 몇가지 명령을 단계별로 진행 할 준비가 되었습니다.

**Try it yourself**

Before installing, defining, and using the marbles private data chaincode below, we need to start the BYFN network. For the sake of this tutorial, we want to operate from a known initial state. The following command will kill any active or stale docker containers and remove previously generated artifacts. Therefore let’s run the following command to clean up any previous environments:

아래의 marbles private data 체인코드를 설치, 정의 및 사용하기 전에 BYFN 네트워크를 시작해야합니다. 이 튜토리얼을 위해 초기 상태에서 작동하려고합니다. 다음 명령은 활성 또는 오래된 도커 컨테이너를 종료하고 이전에 생성 된 아티팩트를 제거합니다. 따라서 다음 명령을 실행하여 이전 환경을 정리하십시오.

```
cd fabric-samples/first-network
./byfn.sh down
```

If you’ve already run through this tutorial, you’ll also want to delete the underlying docker containers for the marbles private data chaincode. Let’s run the following commands to clean up previous environments:

이 튜토리얼을 이미 완료 한 경우 marbles private data 체인코드의 기본 도커 컨테이너를 삭제하려고합니다. 다음 명령을 실행하여 이전 환경을 정리하십시오.

```
docker rm -f $(docker ps -a | awk '($2 ~ /dev-peer.*.marblesp.*/) {print $1}')
docker rmi -f $(docker images | awk '($1 ~ /dev-peer.*.marblesp.*/) {print $3}')
```

Start up the BYFN network with CouchDB by running the following command:

다음 명령을 실행하여 CouchDB와 함께 BYFN 네트워크를 시작하십시요.

```
./byfn.sh up -c mychannel -s couchdb
```

This will create a simple Fabric network consisting of a single channel named `mychannel` with two organizations (each maintaining two peer nodes) and an ordering service while using CouchDB as the state database. Either LevelDB or CouchDB may be used with collections. CouchDB was chosen to demonstrate how to use indexes with private data.

이것은 CouchDB를 state DB로 사용하면서 2개의 조직(각각 2개의 피어노드 유지)과 ordering 서비스를 가진 `mychannel`이라는 단일 채널로 구성된 간단한 Fabric 네트워크를 생성합니다. LevelDB 또는 CouchDB를 컬렉션과 함께 사용할 수 있습니다. CouchDB는 private data와 함께 인덱스를 사용하는 방법을 보여주기 위해 선택되었습니다.

**Note**

For collections to work, it is important to have cross organizational gossip configured correctly. Refer to our documentation on [Gossip data dissemination protocol](https://hyperledger-fabric.readthedocs.io/en/latest/gossip.html), paying particular attention to the section on “anchor peers”. Our tutorial does not focus on gossip given it is already configured in the BYFN sample, but when configuring a channel, the gossip anchors peers are critical to configure for collections to work properly.

**Note**

컬렉션이 작동하려면 조직 간 gossip을 올바르게 구성해야합니다. "anchor peers"섹션에 특히 주의를 기울이고 [Gossip data dissemination protocol](https://hyperledger-fabric.readthedocs.io/en/latest/gossip.html)에 대한 설명서를 참조하십시오. 이 튜토리얼은 BYFN 샘플에서 이미 구성된 gossip에 중점을 두지 않지만 채널을 구성 할 때 gossip anchors peers는 컬렉션이 제대로 작동하도록 구성하는데 중요합니다.

# **Install and define a chaincode with a collection**

Client applications interact with the blockchain ledger through chaincode. Therefore we need to install a chaincode on every peer that will execute and endorse our transactions. However, before we can interact with our chaincode, the members of the channel need to agree on a chaincode definition that establishes chaincode governance, including the private data collection configuration. We are going to package, install, and then define the chaincode on the channel using [peer lifecycle chaincode](https://hyperledger-fabric.readthedocs.io/en/latest/commands/peerlifecycle.html).

클라이언트 응용프로그램은 체인코드를 통해 블록체인 원장과 상호작용합니다. 따라서 트랜잭션을 실행하고 보증 할 모든 피어에 체인코드를 설치해야합니다. 그러나 체인코드와 상호작용하려면 채널 구성원이 PDC 구성을 포함하여 체인코드 거버넌스를 설정하는 체인코드 정의에 동의해야합니다. [peer lifecycle chaincode](https://hyperledger-fabric.readthedocs.io/en/latest/commands/peerlifecycle.html)를 사용하여 채널에서 체인코드를 패키징, 설치 및 정의하려고합니다.

# **Install chaincode on all peers**

The chaincode needs to be packaged before it can be installed on our peers. We can use the [peer lifecycle chaincode package](http://hyperledger-fabric.readthedocs.io/en/latest/commands/peerlifecycle.html#peer-lifecycle-chaincode-package) command to package the marbles chaincode.

피어에 체인코드를 설치하기 전에 체인코드를 패키지해야합니다. '[peer lifecycle chaincode package](http://hyperledger-fabric.readthedocs.io/en/latest/commands/peerlifecycle.html#peer-lifecycle-chaincode-package) 명령을 사용하여 marbles 체인코드를 패키징 할 수 있습니다.

The BYFN network includes two organizations, Org1 and Org2, with two peers each. Therefore, the chaincode package has to be installed on four peers:

BYFN 네트워크에는 각각 2개의 피어가 있는 Org1 및 Org2의 두 조직이 있습니다. 따라서 체인코드 패키지는 4개의 피어에 설치해야합니다.

- peer0.org1.example.com
- peer1.org1.example.com
- peer0.org2.example.com
- peer1.org2.example.com

After the chaincode is packaged, we can use the [peer lifecycle chaincode install](http://hyperledger-fabric.readthedocs.io/en/latest/commands/peerlifecycle.html#peer-lifecycle-chaincode-install) command to install the Marbles chaincode on each peer.

체인코드 패키징 후 [peer lifecycle chaincode install](http://hyperledger-fabric.readthedocs.io/en/latest/commands/peerlifecycle.html#peer-lifecycle-chaincode-install) 명령으로 각 피어에 Marbles 체인코드를 설치합니다.

**Try it yourself**

Assuming you have started the BYFN network, enter the CLI container:

BYFN 네트워크를 시작했다는 가정하에 CLI 컨테이너를 실행합니다:

```
docker exec -it cli bash
```

Your command prompt will change to something similar to:

명령 프롬프트가 다음과 비슷하게 변경됩니다:

```
bash-4.4#
```

1. Use the following command to package the marbles private data chaincode from the git repository inside your local container.

   다음 명령을 사용하여 로컬 컨테이너 내의 git repository에서 marbles private data 체인코드를 패키지하십시오.

```
peer lifecycle chaincode package marblesp.tar.gz --path github.com/hyperledger/fabric-samples/chaincode/marbles02_private/go/ --lang golang --label marblespv1
```

This command will create a chaincode package named marblesp.tar.gz.

이 명령은 marblesp.tar.gz라는 이름의 체인코드 패키지를 생성합니다.

2. Use the following command to install the chaincode package onto the peer `peer0.org1.example.com` in your BYFN network. By default, after starting the BYFN network, the active peer is set to`CORE_PEER_ADDRESS=peer0.org1.example.com:7051`:

   다음 명령을 사용하여 BYFN 네트워크의 `peerorgpeer0.org1.example.com`에 체인코드 패키지를 설치하십시오. BYFN 네트워크를 시작한 후 기본적으로 활성 피어는 `CORE_PEER_ADDRESS = peer0.org1.example.com : 7051`으로 설정됩니다:

```
peer lifecycle chaincode install marblesp.tar.gz
```

A successful install command will return the chaincode identifier, similar to the response below:

성공적인 설치 명령은 아래 응답과 유사한 체인코드를 반환합니다.

```
2019-04-22 19:09:04.336 UTC [cli.lifecycle.chaincode] submitInstallProposal -> INFO 001 Installed remotely: response:<status:200 payload:"\nKmarblespv1:57f5353b2568b79cb5384b5a8458519a47186efc4fcadb98280f5eae6d59c1cd\022\nmarblespv1" >
2019-04-22 19:09:04.336 UTC [cli.lifecycle.chaincode] submitInstallProposal -> INFO 002 Chaincode code package identifier: marblespv1:57f5353b2568b79cb5384b5a8458519a47186efc4fcadb98280f5eae6d59c1cd
```

3. Use the CLI to switch the active peer to the second peer in Org1 and install the chaincode. Copy and paste the following entire block of commands into the CLI container and run them:

   CLI를 사용하여 Org1에서 활성 피어를 두 번째 피어로 전환하고 체인코드를 설치하십시오. 다음 전체 명령 블록을 복사하여 CLI 컨테이너에 붙여넣고 실행하십시오.

```
export CORE_PEER_ADDRESS=peer1.org1.example.com:8051
peer lifecycle chaincode install marblesp.tar.gz
```

4. Use the CLI to switch to Org2. Copy and paste the following block of commands as a group into the peer container and run them all at once:

   CLI를 사용하여 Org2로 전환하십시오. 다음 명령 블록을 그룹으로 복사하여 피어 컨테이너에 붙여넣고 한 번에 모두 실행하십시오.

```
export CORE_PEER_LOCALMSPID=Org2MSP
export PEER0_ORG2_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_TLS_ROOTCERT_FILE=$PEER0_ORG2_CA
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
```

5. Switch the active peer to the first peer in Org2 and install the chaincode:

   활성 피어를 Org2의 첫 번째 피어로 전환하고 체인 코드를 설치하십시오.

```
export CORE_PEER_ADDRESS=peer0.org2.example.com:9051
peer lifecycle chaincode install marblesp.tar.gz
```

6. Switch the active peer to the second peer in org2 and install the chaincode:

   활성 피어를 org2의 두 번째 피어로 전환하고 체인 코드를 설치하십시오.

```
export CORE_PEER_ADDRESS=peer1.org2.example.com:10051
peer lifecycle chaincode install marblesp.tar.gz
```

# **Approve the chaincode definition**

Each channel member that wants to use the chaincode needs to approve a chaincode definition for their organization. Since both organizations are going to use the chaincode in this tutorial, we need to approve the chaincode definition for both Org1 and Org2.

체인코드를 사용하려는 각 채널 구성원은 해당 조직의 체인코드 정의를 승인해야합니다. 이 튜토리얼에서는 두 조직 모두 체인코드를 사용하므로 Org1 및 Org2에 대한 체인코드 정의를 승인해야합니다.

The chaincode definition includes the package identifier that was returned by the install command. This packege ID is used to associate the chaincode package installed on your peers with the chaincode definition approved by your organization. We can also use the [peer lifecycle chaincode queryinstalled](http://hyperledger-fabric.readthedocs.io/en/latest/commands/peerlifecycle.html#peer-lifecycle-chaincode-queryinstalled) command to find the package ID of `marblesp.tar.gz`.

체인코드 정의에는 install 명령으로 반환 된 패키지 식별자가 포함됩니다. 이 패키지 ID는 피어에 설치된 체인코드 패키지를 조직에서 승인 한 체인코드 정의와 연결하는데 사용됩니다. [peer lifecycle chaincode queryinstalled](http://hyperledger-fabric.readthedocs.io/en/latest/commands/peerlifecycle.html#peer-lifecycle-chaincode-queryinstalled) 명령을 사용하여의 패키지 ID `marblesp.tar.gz`를 찾을 수도 있습니다. 

Once we have the package ID, we can then use the [peer lifecycle chaincode approveformyorg](http://hyperledger-fabric.readthedocs.io/en/latest/commands/peerlifecycle.html#peer-lifecycle-chaincode-approveformyorg) command to approve a definition of the marbles chaincode for Org1 and Org2. To approve the private data collection definition that accompanies the `marbles02_private`, sample, provide the path to the collections JSON file using the`--collections-config` flag.

패키지 ID를 확보 한 후에는 [[peer lifecycle chaincode approveformyorg](http://hyperledger-fabric.readthedocs.io/en/latest/commands/peerlifecycle.html#peer-lifecycle-chaincode-approveformyorg) 명령을 사용하여 Org1 및 Org2에 대한 marbles 체인코드의 정의를 보증합니다. `marbles02_private` 샘플과 함께 제공되는 PDC 정의를 승인하려면`--collections-config` flag를 사용하여 콜렉션 JSON 파일의 경로를 제공하십시오.

**Try it yourself**

Run the following commands inside the CLI container to approve a definition for Org1 and Org2.

CLI 컨테이너 내에서 다음 명령을 실행하여 Org1 및 Org2에 대한 정의를 승인하십시오.

1. Use the following command to query your peer for the package ID of the installed chaincode.

   다음 명령을 사용하여 피어에 설치된 체인 코드의 패키지 ID를 쿼리하십시오.

```
peer lifecycle chaincode queryinstalled
```

The command will return the same package identifier as the install command. You should see output similar to the following:

이 명령은 설치 명령과 동일한 패키지 식별자를 반환합니다. 다음과 유사한 출력이 표시되어야합니다.

```
Installed chaincodes on peer:
Package ID: marblespv1:57f5353b2568b79cb5384b5a8458519a47186efc4fcadb98280f5eae6d59c1cd, Label: marblespv1
Package ID: mycc_1:27ef99cb3cbd1b545063f018f3670eddc0d54f40b2660b8f853ad2854c49a0d8, Label: mycc_1
```

2. Declare the package ID as an environment variable. Paste the package ID of marblespv1 returned by the `peer lifecycle chaincode queryinstalled` into the command below. The package ID may not be the same for all users, so you need to complete this step using the package ID returned from your console.

   패키지 ID를 환경 변수로 선언하십시오. `peer lifecycle chaincode queryinstalled`에서 반환 한 marblespv1의 패키지 ID를 아래 명령에 붙여 넣습니다. 패키지 ID가 모든 사용자에게 동일하지 않을 수 있으므로 콘솔에서 반환 된 패키지 ID를 사용하여이 단계를 완료해야합니다.

```
export CC_PACKAGE_ID=marblespv1:57f5353b2568b79cb5384b5a8458519a47186efc4fcadb98280f5eae6d59c1cd
```

3. Make sure we are running the CLI as Org1. Copy and paste the following block of commands as a group into the peer container and run them all at once:

   CLI를 Org1로 실행하고 있는지 확인하십시오. 다음 명령 블록을 그룹으로 복사하여 피어 컨테이너에 붙여넣고 한 번에 모두 실행하십시오.

```
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID=Org1MSP
export PEER0_ORG1_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_TLS_ROOTCERT_FILE=$PEER0_ORG1_CA
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
```

4. Use the following command to approve a definition of the marbles private data chaincode for Org1. This command includes a path to the collection definition file. The approval is distributed within each organization using gossip, so the command does not need to target every peer within an organization.

   다음 명령을 사용하여 Org1에 대한 marbles private data 체인코드의 정의를 승인하십시오. 이 명령에는 컬렉션 정의 파일의 경로가 포함됩니다. 승인은 gossip을 사용하여 각 조직 내에 배포되므로 이 명령은 조직 내의 모든 피어를 대상으로 지정할 필요가 없습니다.

```
export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
peer lifecycle chaincode approveformyorg --channelID mychannel --name marblesp --version 1.0 --collections-config $GOPATH/src/github.com/hyperledger/fabric-samples/chaincode/marbles02_private/collections_config.json --signature-policy "OR('Org1MSP.member','Org2MSP.member')" --init-required --package-id $CC_PACKAGE_ID --sequence 1 --tls true --cafile $ORDERER_CA
```
When the command completes successfully you should see something similar to:

명령이 성공적으로 완료되면 다음과 비슷한 내용이 표시됩니다:

```
2019-03-18 16:04:09.046 UTC [cli.lifecycle.chaincode] InitCmdFactory -> INFO 001 Retrieved channel (mychannel) orderer endpoint: orderer.example.com:7050
2019-03-18 16:04:11.253 UTC [chaincodeCmd] ClientWait -> INFO 002 txid [efba188ca77889cc1c328fc98e0bb12d3ad0abcda3f84da3714471c7c1e6c13c] committed with status (VALID) at
```

5. Use the CLI to switch to Org2. Copy and paste the following block of commands as a group into the peer container and run them all at once.

   CLI를 사용하여 Org2로 전환하십시오. 다음 명령 블록을 그룹으로 복사하여 피어 컨테이너에 붙여넣고 한 번에 모두 실행하십시오.

```
export CORE_PEER_ADDRESS=peer0.org2.example.com:9051
export CORE_PEER_LOCALMSPID=Org2MSP
export PEER0_ORG2_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_TLS_ROOTCERT_FILE=$PEER0_ORG2_CA
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
```

6. You can now approve the chaincode definition for Org2:

   이제 Org2에 대한 체인 코드 정의를 승인 할 수 있습니다.

```
peer lifecycle chaincode approveformyorg --channelID mychannel --name marblesp --version 1.0 --collections-config $GOPATH/src/github.com/hyperledger/fabric-samples/chaincode/marbles02_private/collections_config.json --signature-policy "OR('Org1MSP.member','Org2MSP.member')" --init-required --package-id $CC_PACKAGE_ID --sequence 1 --tls true --cafile $ORDERER_CA
```

# **Commit the chaincode definition**

Once a sufficient number of organizations (in this case, a majority) have approved a chaincode definition, one organization commit the definition to the channel.

충분한 수의 조직(이 경우 과반수)이 체인코드 정의를 승인하면 한 조직이 정의를 채널에 커밋합니다.

Use the [peer lifecycle chaincode commit](http://hyperledger-fabric.readthedocs.io/en/latest/commands/peerlifecycle.html#peer-lifecycle-chaincode-commit) command to commit the chaincode definition. This command needs to target the peers in Org1 and Org2 to collect endorsements for the commit transaction. The peers will endorse the transaction only if their organizations have approved the chaincode definition. This command will also deploy the collection definition to the channel.

체인코드 정의를 커밋하려면 [peer lifecycle chaincode commit](http://hyperledger-fabric.readthedocs.io/en/latest/commands/peerlifecycle.html#peer-lifecycle-chaincode-commit) 명령을 사용하십시오. 이 명령은 커밋 트랜잭션에 대한 승인을 수집하기 위해 Org1 및 Org2의 피어를 대상으로해야합니다. 동료는 조직이 체인코드 정의를 승인 한 경우에만 거래를 승인합니다. 이 명령은 또한 컬렉션 정의를 채널에 배포합니다.

We are ready to use the chaincode after the chaincode definition has been committed to the channel. Because the marbles private data chaincode contains an initiation function, we need to use the [peer chaincode invoke](http://hyperledger-fabric.readthedocs.io/en/master/commands/peerchaincode.html?%20chaincode%20instantiate#peer-chaincode-instantiate) command to invoke `Init()` before we can use other functions in the chaincode.

체인코드 정의가 채널에 커밋 된 후 체인코드를 사용할 준비가되었습니다. marbles private data 체인코드에는 초기화 기능이 포함되어 있기 때문에 체인코드의 다른 기능을 사용하기 전에 [peer chaincode invoke](http://hyperledger-fabric.readthedocs.io/en/master/commands/peerchaincode.html?%20chaincode%20instantiate#peer-chaincode-instantiate) 명령을 사용하여 `InInit()`를 호출합니다.

**Try it yourself**

1. Run the following commands to commit the definition of the marbles private data chaincode to the BYFN channel `mychannel`.

   다음 명령을 실행하여 marbles private data 체인코드의 정의를 BYFN 채널 `mychannel`에 커밋합니다.

```
export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
export ORG1_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt 
export ORG2_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
peer lifecycle chaincode commit -o orderer.example.com:7050 --channelID mychannel --name marblesp --version 1.0 --sequence 1 --collections-config $GOPATH/src/github.com/hyperledger/fabric-samples/chaincode/marbles02_private/collections_config.json --signature-policy "OR('Org1MSP.member','Org2MSP.member')" --init-required --tls true --cafile $ORDERER_CA --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles $ORG1_CA --peerAddresses peer0.org2.example.com:9051 --tlsRootCertFiles $ORG2_CA

.. note:: When specifying the value of the ``--collections-config`` flag, you will need to specify the fully qualified path to the collections_config.json file. For example: .. code:: bash --collections-config $GOPATH/src/github.com/hyperledger/fabric-samples/chaincode/marbles02_private/collections_config.json
When the commit transaction completes successfully you should see something
similar to:

  .. code:: bash [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
```

2. Use the following command to invoke the `Init` function to initialize the chaincode:

   다음 명령을 사용하여 `Init` 함수를 호출하여 체인코드를 초기화하십시오.

```
peer chaincode invoke -o orderer.example.com:7050 --channelID mychannel --name marblesp --isInit --tls true --cafile $ORDERER_CA --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles $ORG1_CA -c '{"Args":["Init"]}'
```

# **Store private data**

Acting as a member of Org1, who is authorized to transact with all of the private data in the marbles private data sample, switch back to an Org1 peer and submit a request to add a marble:

marbles private data 샘플의 모든 private data를 처리 할 수있는 권한이 있는 Org1의 구성원으로 작동하여 Org1 피어로 다시 전환하고 marble 추가 요청을 제출하십시오.

**Try it yourself**

Copy and paste the following set of commands to the CLI command line.

다음 명령 세트를 복사하여 CLI 명령행에 붙여 넣으십시오.

```
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID=Org1MSP
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export PEER0_ORG1_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

Invoke the marbles `initMarble` function which creates a marble with private data — name `marble1` owned by `tom` with a color `blue`, size `35`and price of `99`. Recall that private data **price** will be stored separately from the private data **name, owner, color, size**. For this reason, the `initMarble` function calls the `PutPrivateData()` API twice to persist the private data, once for each collection. Also note that the private data is passed using the `--transient` flag. Inputs passed as transient data will not be persisted in the transaction in order to keep the data private. Transient data is passed as binary data and therefore when using CLI it must be base64 encoded. We use an environment variable to capture the base64 encoded value, and use `tr` command to strip off the problematic newline characters that linux base64 command adds.

private data가 있는 marbles `initmarble` 기능을 호출합니다. 이름은 `marble1` 소유자 `Tom` 색상은 `blue`, 사이즈는 `35`, 가격은 `99`입니다. private data **price**은 private data **name, owner, color, size**와 별도로 저장됩니다. 이러한 이유로, `initMarble' 함수는 'PutPrivateData()` API를 두 번 호출하여 private data를 각 컬렉션마다 한 번씩 유지합니다. 또한 private data는 `--transient` flag를 사용하여 전달됩니다. 임시 데이터로 전달 된 입력은 데이터를 개인용으로 유지하기 위해 트랜잭션에서 유지되지 않습니다. 임시 데이터는 이진 데이터로 전달되므로 CLI를 사용할 때는 base64로 인코딩해야합니다. 환경변수를 사용하여 base64로 인코딩 된 값을 캡처하고 `tr` 명령을 사용하여 줄바꿈 문자를 제거합니다.

```
export MARBLE=$(echo -n "{\"name\":\"marble1\",\"color\":\"blue\",\"size\":35,\"owner\":\"tom\",\"price\":99}" | base64 | tr -d \\n)
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["initMarble"]}' --transient "{\"marble\":\"$MARBLE\"}"
```

You should see results similar to:

다음과 유사한 결과가 나타납니다:

```
> [chaincodeCmd] chaincodeInvokeOrQuery->INFO 001 Chaincode invoke successful. result: status:200
```

# **Query the private data as an authorized peer**

Our collection definition allows all members of Org1 and Org2 to have the `name, color, size, owner` private data in their side database, but only peers in Org1 can have the `price` private data in their side database. As an authorized peer in Org1, we will query both sets of private data.

우리의 컬렉션 정의는 Org1과 Org2의 모든 구성원이 side DB에 '`name, color, size, owner` private data를 가질 수 있도록 허용하지만 Org1의 피어 만 side DB에 `price` private data를 가질 수 있습니다. Org1의 승인 된 피어로서 두 private data 세트를 모두 쿼리합니다.

The first `query` command calls the `readMarble` function which passes`collectionMarbles` as an argument.

첫 번째 `query` 명령은 `collectionMarbles`를 인수로 전달하는 `readMarble` 함수를 호출합니다.

```
// ===============================================// readMarble - read a marble from chaincode state// ===============================================func (t SimpleChaincode) readMarble(stub shim.ChaincodeStubInterface, args []string) pb.Response { var name, jsonResp stringvar err errorif len(args) != 1 { return shim.Error("Incorrect number of arguments. Expecting name of the marble to query") } name = args[0] valAsbytes, err := stub.GetPrivateData("collectionMarbles", name) //get the marble from chaincode stateif err != nil { jsonResp = "{\"Error\":\"Failed to get state for " + name + "\"}" return shim.Error(jsonResp) } else if valAsbytes == nil { jsonResp = "{\"Error\":\"Marble does not exist: " + name + "\"}" return shim.Error(jsonResp) } return shim.Success(valAsbytes)
}
```

The second `query` command calls the `readMarblePrivateDetails` function which passes `collectionMarblePrivateDetails` as an argument.

두 번째 `query` 명령은 `collectionMarblePrivateDetails` 인수를 전달하는 `readMarblePrivateDetails` 함수를 호출합니다.

```
*// ===============================================// readMarblePrivateDetails - read a marble private details from chaincode state// ===============================================***func** (t *****SimpleChaincode) readMarblePrivateDetails(stub shim.ChaincodeStubInterface, args []**string**) pb.Response { **var** name, jsonResp **stringvar** err **errorif** len(args) **!=** 1 { **return** shim.Error("Incorrect number of arguments. Expecting name of the marble to query") } name = args[0] valAsbytes, err **:=** stub.GetPrivateData("collectionMarblePrivateDetails", name) *//get the marble private details from chaincode state***if** err **!=** **nil** { jsonResp = "{\"Error\":\"Failed to get private details for " **+** name **+** ": " **+** err.Error() **+** "\"}" **return** shim.Error(jsonResp) } **else** **if** valAsbytes **==** **nil** { jsonResp = "{\"Error\":\"Marble private details does not exist: " **+** name **+** "\"}" **return** shim.Error(jsonResp) } **return** shim.Success(valAsbytes)
}
```

Now **Try it yourself**

Query for the `name, color, size and owner` private data of `marble1` as a member of Org1. Note that since queries do not get recorded on the ledger, there is no need to pass the marble name as a transient input.

`marble1`의 `name, color, size and owner`의 private data를 Org1의 구성원으로 쿼리합니다. 쿼리는 원장에 기록되지 않으므로 marble 이름을 임시 입력으로 전달할 필요가 없습니다.

```
peer chaincode query **-**C mychannel **-**n marblesp *p*-**c '{"Args":["readMarble","marble1"]}'
```

You should see the following result:

다음과 같은 결과가 나타납니다:

```
{"color":"blue","docType":"marble","name":"marble1","owner":"tom","size":35}
```

Query for the `price` private data of `marble1` as a member of Org1.

`marble1`의 `price` private data를 Org1의 구성원으로 쿼리합니다.

```
peer chaincode query **-**C mychannel **-**n marblesp **-**c '{"Args":["readMarblePrivateDetails","marble1"]}'
```

You should see the following result:

다음과 같은 결과가 나타납니다:

```
{"docType":"marblePrivateDetails","name":"marble1","price":99}
```

# **Query the private data as an unauthorized peer**

Now we will switch to a member of Org2 which has the marbles private data `name, color, size, owner` in its side database, but does not have the marbles `price` private data in its side database. We will query for both sets of private data.

이제 side DB에 marbles private data `name, color, size, owner`가 있지만 조직 데이터베이스에 marbles `price` private data가 없는 Org2 멤버로 전환합니다. 두 private data 세트를 모두 쿼리합니다.

# **Switch to a peer in Org2**

From inside the docker container, run the following commands to switch to the peer which is unauthorized to access the marbles `price` private data.

도커 컨테이너 내부에서 다음 명령을 실행하여 marbles `price` private data에 액세스 권한이 없는 피어로 전환하십시오.

**Try it yourself**

```
export CORE_PEER_ADDRESS=peer0.org2.example.com:9051
export CORE_PEER_LOCALMSPID=Org2MSP
export PEER0_ORG2_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_TLS_ROOTCERT_FILE=$PEER0_ORG2_CA
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
```

# **Query private data Org2 is authorized to**

Peers in Org2 should have the first set of marbles private data (`name, color, size and owner`) in their side database and can access it using the `readMarble()` function which is called with the `collectionMarbles` argument.

Org2의 피어는 첫 번째 marbles private data (`name, color, size and owner`)를 side DB에 가져야하며 `collectionMarbles` 인수로 호출되는`readMarble()` 함수를 사용하여 액세스 할 수 있습니다.

**Try it yourself**

```
peer chaincode query **-**C mychannel **-**n marblesp **-**c '{"Args":["readMarble","marble1"]}'
```

You should see something similar to the following result:

다음과 같은 결과가 나타납니다:

```
{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"tom"}
```

# **Query private data Org2 is not authorized to**

Peers in Org2 do not have the marbles `price` private data in their side database. When they try to query for this data, they get back a hash of the key matching the public state but will not have the private state.

Org2의 피어는 side DB에 marbles `price` 정보가 없습니다. 이 데이터를 쿼리하려고 하면 public state와 일치하는 키의 해시를 얻지만 private state는 갖지 않습니다.

**Try it yourself**

```
peer chaincode query **-**C mychannel **-**n marblesp **-**c '{"Args":["readMarblePrivateDetails","marble1"]}'
```

You should see a result similar to:

다음과 같은 결과가 나타납니다:

```
{"Error":"Failed to get private details for marble1: GET_STATE failed:
transaction ID: b04adebbf165ddc90b4ab897171e1daa7d360079ac18e65fa15d84ddfebfae90:
Private data matching public hash version **is** **not** available**.** Public hash
version **=** **&**version**.**Height{BlockNum:0x6, TxNum:0x0}, Private data version **=**(*****version**.**Height)(nil)"}
```

Members of Org2 will only be able to see the public hash of the private data.

Org2 구성원은 private data의 공개 해시만 볼 수 있습니다.

# **Purge Private Data**

For use cases where private data only needs to be on the ledger until it can be replicated into an off-chain database, it is possible to “purge” the data after a certain set number of blocks, leaving behind only hash of the data that serves as immutable evidence of the transaction.

private data가 off-chain DB로 복제 될 때까지 원장에만 있어야하는 사용 사례의 경우 특정 블록 수 이후에 데이터를 "퍼지"하여 데이터의 해시만 남길 수 있으며 트랜잭션의 불변 증거로 사용됩니다.

There may be private data including personal or confidential information, such as the pricing data in our example, that the transacting parties don’t want disclosed to other organizations on the channel. Thus, it has a limited lifespan, and can be purged after existing unchanged on the blockchain for a designated number of blocks using the `blockToLive`property in the collection definition.

거래 당사자가 채널의 다른 조직에 공개하고 싶지 않은 위 예의 가격 데이터와 같은 개인 정보 또는 기밀 정보를 포함한 private data가 있을 수 있습니다. 따라서 수명이 제한되어 있으며 컬렉션 정의에서 `blockToLive` 속성을 사용하여 지정된 수의 블록에 대해 블록체인에서 제거 할 수 있습니다.

Our `collectionMarblePrivateDetails` definition has a `blockToLive`property value of three meaning this data will live on the side database for three blocks and then after that it will get purged. Tying all of the pieces together, recall this collection definition`collectionMarblePrivateDetails` is associated with the `price` private data in the `initMarble()` function when it calls the `PutPrivateData()` API and passes the `collectionMarblePrivateDetails` as an argument.

우리의 `collectionMarblePrivateDetails` 정의에서 `blockToLive` 속성 값 3의 의미는 이 데이터가 3개의 블록 동안 side DB에 존재하고 그 후에 제거 될 것임을 의미합니다. 모든 조각들을 하나로 묶어이 컬렉션 정의 `collectionMarblePrivateDetails`는 `PutPrivateData()`API를 호출하고 인수로 `collectionMarblePrivateDetails`를 전달할 때 `initMarble()` 함수의 `price` private data와 관련이 있습니다. .

We will step through adding blocks to the chain, and then watch the price information get purged by issuing four new transactions (Create a new marble, followed by three marble transfers) which adds four new blocks to the chain. After the fourth transaction (third marble transfer), we will verify that the price private data is purged.

우리는 체인에 블록을 추가하는 단계를 거쳐 체인에 4개의 새로운 블록을 추가하는 4개의 새로운 트랜잭션(새로운 marble 생성, 3개의 marble 전송)을 발행하여 가격 정보가 제거되는 것을 지켜 볼 것입니다. 네 번째 트랜잭션(세 번째 marble 이전) 후 private data 가격이 제거되었는지 확인합니다.

**Try it yourself**

Switch back to peer0 in Org1 using the following commands. Copy and paste the following code block and run it inside your peer container:

다음 명령을 사용하여 Org1에서 peer0으로 다시 전환하십시오. 다음 코드 블록을 복사하여 붙여 넣고 피어 컨테이너 내에서 실행하십시오.

```
export CORE_PEER_ADDRESS**=**peer0**.**org1**.**example**.**com:7051
export CORE_PEER_LOCALMSPID**=**Org1MSP
export CORE_PEER_TLS_ROOTCERT_FILE**=/**opt**/**gopath**/**src**/**github**.**com**/**hyperledger**/**fabric**/**peer**/**crypto**/**peerOrganizations**/**org1**.**example**.**com**/**peers**/**peer0**.**org1**.**example**.**com**/**tls**/**ca**.**crt
export CORE_PEER_MSPCONFIGPATH**=/**opt**/**gopath**/**src**/**github**.**com**/**hyperledger**/**fabric**/**peer**/**crypto**/**peerOrganizations**/**org1**.**example**.**com**/**users**/**Admin@org1**.**example**.**com**/**msp
export PEER0_ORG1_CA**=/**opt**/**gopath**/**src**/**github**.**com**/**hyperledger**/**fabric**/**peer**/**crypto**/**peerOrganizations**/**org1**.**example**.**com**/**peers**/**peer0**.**org1**.**example**.**com**/**tls**/**ca**.**crt
```

Open a new terminal window and view the private data logs for this peer by running the following command:

새 터미널 창을 열고 다음 명령을 실행하여이 피어의 개인 데이터 로그를보십시오:

```
docker logs peer0**.**org1**.**example**.**com 2**>&**1 **|** grep **-**i **-**a **-**E 'private|pvt|privdata'
```

You should see results similar to the following. Note the highest block number in the list. In the example below, the highest block height is `4`.

다음과 유사한 결과가 나타납니다. 리스트에서 가장 높은 블록 번호를 기록하십시오. 아래 예에서 가장 높은 블록 높이는 `4`입니다.

```
[pvtdatastorage] func1 **->** INFO 023 Purger started: Purging expired private data till block number [0]
[pvtdatastorage] func1 **->** INFO 024 Purger finished
[kvledger] CommitLegacy **->** INFO 022 Channel [mychannel]: Committed block [0] **with** 1 transaction(s)
[kvledger] CommitLegacy **->** INFO 02e Channel [mychannel]: Committed block [1] **with** 1 transaction(s)
[kvledger] CommitLegacy **->** INFO 030 Channel [mychannel]: Committed block [2] **with** 1 transaction(s)
[kvledger] CommitLegacy **->** INFO 036 Channel [mychannel]: Committed block [3] **with** 1 transaction(s)
[kvledger] CommitLegacy **->** INFO 03e Channel [mychannel]: Committed block [4] **with** 1 transaction(s)
```

Back in the peer container, query for the **marble1** price data by running the following command. (A Query does not create a new transaction on the ledger since no data is transacted).

피어 컨테이너로 돌아가서 다음 명령을 실행하여 **marble1** 가격 데이터를 쿼리하십시오. (데이터가 처리되지 않으므로 조회는 원장에 새 트랜잭션을 작성하지 않습니다).

```
peer chaincode query **-**C mychannel **-**n marblesp **-**c '{"Args":["readMarblePrivateDetails","marble1"]}'
```

You should see results similar to:

다음과 유사한 결과가 나타납니다:

```
{"docType":"marblePrivateDetails","name":"marble1","price":99}
```

The `price` data is still in the private data ledger.

`price` 데이터는 여전히 private data 원장에 있습니다.

Create a new **marble2** by issuing the following command. This transaction creates a new block on the chain.

다음 명령을 실행하여 새로운 **marble2**를 작성하십시오. 이 트랜잭션은 체인에 새로운 블록을 생성합니다.

```
export MARBLE=$(echo -n "{\"name\":\"marble2\",\"color\":\"blue\",\"size\":35,\"owner\":\"tom\",\"price\":99}" | base64 | tr -d \\n)
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["initMarble"]}' --transient "{\"marble\":\"$MARBLE\"}"
```

Switch back to the Terminal window and view the private data logs for this peer again. You should see the block height increase by 1.

터미널 창으로 다시 전환하고 이 피어의 private data 로그를 다시보십시오. 블록 높이가 1 씩 증가하는 것을 볼 수 있습니다.

```
docker logs peer0**.**org1**.**example**.**com 2**>&**1 **|** grep **-**i **-**a **-**E 'private|pvt|privdata'
```

Back in the peer container, query for the **marble1** price data again by running the following command:

피어 컨테이너로 돌아가서 다음 명령을 실행하여 **marble1** 가격 데이터를 다시 쿼리하십시오.

```
peer chaincode query **-**C mychannel **-**n marblesp **-**c '{"Args":["readMarblePrivateDetails","marble1"]}'
```

The private data has not been purged, therefore the results are unchanged from previous query:

private data가 제거되지 않았으므로 결과는 이전 쿼리와 변경되지 않습니다.

```
{"docType":"marblePrivateDetails","name":"marble1","price":99}
```

Transfer marble2 to “joe” by running the following command. This transaction will add a second new block on the chain.

다음 명령을 실행하여 marble2를 “joe”로 전송하십시오. 이 트랜잭션은 체인에 두 번째 새 블록을 추가합니다.

```
export MARBLE_OWNER=$(echo -n "{\"name\":\"marble2\",\"owner\":\"joe\"}" | base64 | tr -d \\n)
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["transferMarble"]}' --transient "{\"marble_owner\":\"$MARBLE_OWNER\"}"
```

Switch back to the Terminal window and view the private data logs for this peer again. You should see the block height increase by 1.

터미널 창으로 다시 전환하고 이 피어의 private data 로그를 다시보십시오. 블록 높이가 1 씩 증가하는 것을 볼 수 있습니다.

```
docker logs peer0**.**org1**.**example**.**com 2**>&**1 **|** grep **-**i **-**a **-**E 'private|pvt|privdata'
```

Back in the peer container, query for the marble1 price data by running the following command:

피어 컨테이너로 돌아가서 다음 명령을 실행하여 marble1 가격 데이터를 쿼리하십시오.

```
peer chaincode query **-**C mychannel **-**n marblesp **-**c '{"Args":["readMarblePrivateDetails","marble1"]}'
```

You should still be able to see the price private data.

여전히 가격 private data를 볼 수 있어야합니다.

```
{"docType":"marblePrivateDetails","name":"marble1","price":99}
```

Transfer marble2 to “tom” by running the following command. This transaction will create a third new block on the chain.

다음 명령을 실행하여 marble2를 “tom”으로 전송하십시오. 이 트랜잭션은 체인에 세 번째 새 블록을 만듭니다.

```
export MARBLE_OWNER=$(echo -n "{\"name\":\"marble2\",\"owner\":\"tom\"}" | base64 | tr -d \\n)
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["transferMarble"]}' --transient "{\"marble_owner\":\"$MARBLE_OWNER\"}"
```

Switch back to the Terminal window and view the private data logs for this peer again. You should see the block height increase by 1.

터미널 창으로 다시 전환하고 이 피어의 private data 로그를 다시보십시오. 블록 높이가 1 씩 증가하는 것을 볼 수 있습니다.

```
docker logs peer0**.**org1**.**example**.**com 2**>&**1 **|** grep **-**i **-**a **-**E 'private|pvt|privdata'
```

Back in the peer container, query for the marble1 price data by running the following command:

피어 컨테이너로 돌아가서 다음 명령을 실행하여 marble1 가격 데이터를 쿼리하십시오.

```
peer chaincode query **-**C mychannel **-**n marblesp **-**c '{"Args":["readMarblePrivateDetails","marble1"]}'
```

You should still be able to see the price data.

여전히 가격 private data를 볼 수 있어야합니다.

```
{"docType":"marblePrivateDetails","name":"marble1","price":99}
```

Finally, transfer marble2 to “jerry” by running the following command. This transaction will create a fourth new block on the chain. The `price`private data should be purged after this transaction.

마지막으로 다음 명령을 실행하여 marble2를“jerry”로 옮깁니다. 이 트랜잭션은 체인에 네 번째 새 블록을 만듭니다. `price`private data는 이 거래 후 제거해야합니다.

```
export MARBLE_OWNER=$(echo -n "{\"name\":\"marble2\",\"owner\":\"jerry\"}" | base64 | tr -d \\n)
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["transferMarble"]}' --transient "{\"marble_owner\":\"$MARBLE_OWNER\"}"
```

Switch back to the Terminal window and view the private data logs for this peer again. You should see the block height increase by 1.

터미널 창으로 다시 전환하고이 피어의 개인 데이터 로그를 다시보십시오. 블록 높이가 1 씩 증가하는 것을 볼 수 있습니다.

```
docker logs peer0**.**org1**.**example**.**com 2**>&**1 **|** grep **-**i **-**a **-**E 'private|pvt|privdata'
```

Back in the peer container, query for the marble1 price data by running the following command:

피어 컨테이너로 돌아가서 다음 명령을 실행하여 marble1 가격 데이터를 쿼리하십시오.

```
peer chaincode query **-**C mychannel **-**n marblesp **-**c '{"Args":["readMarblePrivateDetails","marble1"]}'
```

Because the price data has been purged, you should no longer be able to see it. You should see something similar to:

가격 데이터가 제거되었으므로 더 이상 볼 수 없습니다. 다음과 비슷한 내용이 표시되어야합니다.

```
Error: endorsement failure during query**.** response: status:500
message:"{\"Error\":\"Marble private details does not exist: marble1\"}"
```

# **Using indexes with private data**

Indexes can also be applied to private data collections, by packaging indexes in the `META-INF/statedb/couchdb/collections/<collection_name>/indexes`directory alongside the chaincode. An example index is available [here](https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02_private/go/META-INF/statedb/couchdb/collections/collectionMarbles/indexes/indexOwner.json).

체인코드와 함께 `META-INF/statedb/couchdb/collections/<collection_name>/indexes` 디렉토리에 인덱스를 패키징하여 인덱스를 PDC에도 적용 할 수 있습니다. 예를 들어 인덱스를 사용할 수 있습니다 [here](https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02_private/go/META-INF/statedb/couchdb/collections/collectionMarbles/indexes/indexOwner.json).

For deployment of chaincode to production environments, it is recommended to define any indexes alongside chaincode so that the chaincode and supporting indexes are deployed automatically as a unit, once the chaincode has been installed on a peer and instantiated on a channel. The associated indexes are automatically deployed upon chaincode instantiation on the channel when the `--collections-config`flag is specified pointing to the location of the collection JSON file.

체인코드를 프로덕션 환경에 배포하려면 체인코드가 피어에 설치되고 채널에 인스턴스화되면 체인코드 및 지원 인덱스가 자동으로 한 단위로 배포되도록 체인코드와 함께 인덱스를 정의하는 것이 좋습니다. `--collections-config` 플래그가 컬렉션 JSON 파일의 위치를 가리키도록 지정되면 채널에서 체인코드 인스턴스화 시 관련 인덱스가 자동으로 배포됩니다.

# **Additional resources**

For additional private data education, a video tutorial has been created.

추가적인 private data 교육을 위해 비디오 튜토리얼이 작성되었습니다.

**Note**

**주기**

The video uses the previous lifecycle model to install private data collections with chaincode.

비디오는 이전 라이프 사이클 모델을 사용하여 체인코드로 PDC를 설치합니다.

[![Video Label](http://img.youtube.com/vi/qyjDi93URJE/0.jpg)](https://youtu.be/qyjDi93URJE?t=0s) 

