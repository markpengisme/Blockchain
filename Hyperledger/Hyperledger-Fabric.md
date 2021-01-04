# Hyperledger Fabric

## Prerequisites

1. install git
2. install cURL
3. install docker
4. install docker-compose
5. install samples `curl -sSL https://bit.ly/2ysbOFE | bash -s`
6. `cd fabric-samples/test-network`->`export PATH=${PWD}/bin:$PATH`

## Using the fabric test network(Lab 1)

1. `cd fabric-samples/test-network`

2.  Bring up the network: `./network.sh down`->`./network.sh up`

   - 2 peer nodes: `peer0.org1.example.com`,`peer0.org2.example.com`
     - Peer node 要是某個 org 的成員
     - Peer node 儲存區塊鏈分類帳，並在交易提交到分類帳之前驗證它們(運行智慧合約)。
   - 1 ordering node:`orderer.example.com`
     - Oredeing node從客戶端接收到認可的交易後，它們會就交易順序達成共識，然後將它們添加到區塊中。然後，這些區塊分發給peer node然後添加到區塊鏈分類帳中。
   - 0 channel

3. Create channel: `./network.sh createChannel`，取名:

4. Starting a Chaincode on the channel: 

   1. ```sh
      ./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/Chaincode-go -ccl go
      ```

   - 為了確保交易的有效性，使用智能合約創建的交易通常需要由多個組織簽署(多重簽名)才能提交到渠道分類賬。為了簽署交易，每個組織需要在其對等體上調用並執行智能合約，然後簽署交易的輸出。
- 指定通道上需要執行智能合約的集合組織的策略被稱為背書策略，作為Chaincode definition一部分由Chaincode設定
   - 一個Chaincode被安裝在組織的對等體上，然後部署到一個通道上，然後它可以用來認可交易並與區塊鏈分類賬進行互動。
   - 在將Chaincode部署到通道之前，通道的成員需要對建立Chaincode治理的Chaincode definition達成一致。當所需數量的組織同意時，Chaincode definition可以提交給channel，Chaincode 就可以使用了。
   
5. Interacting with the network

    1. Environment variables for Org
    
        ```sh
        export PATH=${PWD}/../bin:$PATH
        export FABRIC_CFG_PATH=$PWD/../config/
        
        # Environment variables for Org1
        export CORE_PEER_TLS_ENABLED=true
        export CORE_PEER_LOCALMSPID="Org1MSP"
        export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
        export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
        export CORE_PEER_ADDRESS=localhost:7051
        ```
      
    2. Init Ledger
      
        ```sh
       peer Chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'
       ```

    3. Get All Assets
      
        ```sh
    peer Chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
        ```

    4. Transfer Assets
      
        ```sh
    peer Chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"TransferAsset","Args":["asset6","Christopher"]}'
        ```
      
    5. Environment variables for Org2
      
        ```sh
        export CORE_PEER_TLS_ENABLED=true
        export CORE_PEER_LOCALMSPID="Org2MSP"
        export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
        export CORE_PEER_ADDRESS=localhost:9051
        ```
        
    6. Read Assest
      
       ```sh
       peer Chaincode query -C mychannel -n basic -c '{"Args":["ReadAsset","asset6"]}'
       ```
    
6. Bring down the network

	> The command will stop and remove the node and Chaincode containers, delete the organization crypto material, and remove the Chaincode images from your Docker Registry. The command also removes the channel artifacts and docker volumes from previous runs, allowing you to run `./network.sh up` again if you encountered any problems.
	
	```
	./network.sh dow
	```

## Deploying a smart contract to a channel(Lab 2)

在Hyperledger Fabric中，智能合約部署在稱為Chaincode的package中。想要驗證交易或查詢帳本的組織需要在他們的peers身上安裝Chaincode。在連接到channel的peernode上安裝了Chaincode之後，channel成員可以將Chaincode部署到channel，並使用Chaincode中的智能合約建立或更新通道分類帳上的資產。

Chaincode生命週期的流程首先會將Chaincode部署到通道上，然後允許多個組織在使用Chaincode建立交易之前就操作Chaincode的方式達成一致。

1. Startup network

   ```sh
   ./network.sh down
   ./network.sh up createChannel
   ```

2. Setup Logspout

   - 這個工具將來自不同Docker容器的輸出流收集到一個位置，從而可以輕鬆地從單個視窗查看發生的情況。

   ```sh
   cd fabric-samples/test-network
   cp ../commercial-paper/organization/digibank/configuration/cli/monitordocker.sh .
   ./monitordocker.sh net_test
   ```

3. Package the smart contract

   ```sh
   cd fabric-samples/asset-transfer-basic/Chaincode-go
   cat go.mod
   GO111MODULE=on go mod vendor
   cd ../../test-network
   export PATH=${PWD}/../bin:$PATH
   export FABRIC_CFG_PATH=$PWD/../config/
   peer lifecycle Chaincode package basic.tar.gz --path ../asset-transfer-basic/Chaincode-go/ --lang golang --label basic_1.0
   ```

4. Install the Chaincode package

   ```sh
   ## peer0.org1.example.com
   export CORE_PEER_TLS_ENABLED=true
   export CORE_PEER_LOCALMSPID="Org1MSP"
   export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
   export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
   export CORE_PEER_ADDRESS=localhost:7051
   
   peer lifecycle Chaincode install basic.tar.gz
   ```
   
   ```sh
   ## peer0.org2.example.com
   export CORE_PEER_TLS_ENABLED=true
   export CORE_PEER_LOCALMSPID="Org2MSP"
   export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
   export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
   export CORE_PEER_ADDRESS=localhost:9051
   
   peer lifecycle Chaincode install basic.tar.gz
   ```


5. Approve a Chaincode definition
   
   Packeage ID 用於將 peer 安裝的 Chaincode 與同意的 Chaincode definition 相關聯，並允許組織使用 Chaincode 對交易進行背書。
   
   ```sh
   ## Package ID
   peer lifecycle Chaincode queryinstalled
   ```
   
   Chaincode 是在組織級別上得到同意的，因此該命令僅需要針對一個peer，然後會使用 gossip 分發到組織內的其他peer上。 使用`peer lifecycle Chaincode approveformyorg`同意 Chaincode definition
   
   ```sh
   ## 注意！每個人拿到的ID都不一樣
   export CC_PACKAGE_ID=basic_1.0:4ec191e793b27e953ff2ede5a8bcc63152cecb1e4c3f301a26e22692c61967ad
   
   ## Org1 
   export CORE_PEER_TLS_ENABLED=true
   export CORE_PEER_LOCALMSPID="Org1MSP"
   export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
   export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
   export CORE_PEER_ADDRESS=localhost:7051
   
   peer lifecycle Chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
   
   ## Org2
   export CORE_PEER_LOCALMSPID="Org2MSP"
   export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
   export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
   export CORE_PEER_ADDRESS=localhost:9051
   
   peer lifecycle Chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
   ```
   
   我們可以向 `approveformyorg`命令提供`--signature-policy`或`--channel-config-policy`參數，以指定 Chaincode 背書策略。默認為多數決，[Endorsement Policies](https://hyperledger-fabric.readthedocs.io/en/release-2.2/endorsement-policies.html)。
    	
   需要具管理員身份的角色同意 Chaincode difinination。因此，`CORE_PEER_MSPCONFIGPATH`變數需要指向包含管理員身份的MSP資料夾，不能使用客戶端使用者同意 Chaincode difinination。同意需要提交給 ordering service，該服務將驗證管理員簽名，然後將同意分發給peer。


6. Committing the Chaincode definition to the channel

    在足夠多的組織同意了 Chaincode difinination 之後，其中一個組織可以將 Chaincode difinination 提交給 channel，交易將會成功提交，並且 Chaincode difinination中同意的參數將在該channel上實現。

    ```sh
    ## 查看同意情形(check commit readiness)
    peer lifecycle Chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --output json
    
    ## Org1
    export CORE_PEER_TLS_ENABLED=true
    export CORE_PEER_LOCALMSPID="Org1MSP"
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    export CORE_PEER_ADDRESS=localhost:7051
        
    ## Commit
    peer lifecycle Chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 1.0 --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    ```
    
    上面的交易使用`peerAddresses`針對Org1的peer0.org1.example.com、Org2的peer0.org2.example.com。提交的交易被提交到連接到channel的peer。該命令需要以足夠多的組織為目標，以滿足部署 Chaincode 的策略。由於同意分佈在每個組織中，所以可以針對屬於channel成員的任何peer。
    
    Channel member 對 Chaincode difinination 的背書將提交給ordering service，以添加到區塊中並分發給channel。 然後，channel上的peers將驗證是否有足夠的組織同意了Chaincode difinination。`peer lifecycle Chaincode commit`命令將等待peer的驗證才返回回應。
    
    ```sh
    ## 查訊 commit 情形
	peer lifecycle Chaincode querycommitted --channelID mychannel --name basic --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    ```
    
7. Invoking the Chaincode

    在Chaincode difinination被提交到channel後，Chaincode將在加入channel有安裝Chaincode的peers上啓動。

    使用 `peer Chaincode invoke`呼叫Chaincode以及`peer Chaincode query`查詢結果

    ```sh
    peer Chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'

    ## 查詢
    peer Chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
    ```

8. Upgrading a smart contract

    channel成員可以通過安裝新的Chaincode package，然後用新的package ID、新的Chaincode版本以及遞增序列號來升級Chaincode。新的Chaincode可以在Chaincode difinination提交給channel後使用。這個過程允許channel成員協調Chaincode升級的時間，並確保在新的Chaincode被部署到channel之前，有足夠數量的Chaincode成員準備好使用新的Chaincode。通過同意帶有新的背書策略的Chaincode difinination，並將Chaincode difinination提交給channel，chnnel成員可以更改管理Chaincode的背書策略，而無需安裝新的Chaincode package。

    以下將以JS語言代替GO語言寫成的package當作**升級範例**：

    ```sh
    ## npm install
    cd ../asset-transfer-basic/Chaincode-javascript
    npm install
    cd ../../test-network
    
    ## package
    export PATH=${PWD}/../bin:$PATH
    export FABRIC_CFG_PATH=$PWD/../config/
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    peer lifecycle Chaincode package basic_2.tar.gz --path ../asset-transfer-basic/Chaincode-javascript/ --lang node --label basic_2.0
    
    ## Org1
    export CORE_PEER_TLS_ENABLED=true
    export CORE_PEER_LOCALMSPID="Org1MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    export CORE_PEER_ADDRESS=localhost:7051
    
    ## install package
    peer lifecycle Chaincode install basic_2.tar.gz
    
    ## query installed(注意！package id 會不同)
    peer lifecycle Chaincode queryinstalled
    
    ## approve a new Chaincode definition
    export NEW_CC_PACKAGE_ID=basic_2.0:4603980dac5d585e20e181b3ddc7c1d063dc62c3b39bb4e2b271217cde2d9e4d
    peer lifecycle Chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 2.0 --package-id $NEW_CC_PACKAGE_ID --sequence 2 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    
    ## Org2
    export CORE_PEER_LOCALMSPID="Org2MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    export CORE_PEER_ADDRESS=localhost:9051
    
    ## install package
    peer lifecycle Chaincode install basic_2.tar.gz
    
    ## approve a new Chaincode definition
    peer lifecycle Chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 2.0 --package-id $NEW_CC_PACKAGE_ID --sequence 2 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    
    ## check commited readiness
    peer lifecycle Chaincode checkcommitreadiness --channelID mychannel --name basic --version 2.0 --sequence 2 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --output json
    
    ## commit
    peer lifecycle Chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 2.0 --sequence 2 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    
    ## verify result
    docker ps
    
    ## invoke
    peer Chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"CreateAsset","Args":["asset8","blue","16","Kelley","750"]}'
    
    ## query
    peer Chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
    ```

9. Clean up

    ```sh
    docker stop logspout
    docker rm logspout
    ./network.sh down
    ```



## Writing Your First Application(Lab3)

**About Asset Transfer**

-   Sample application: `asset-transfer-basic/application-javascript`
-   Smart contract : `asset-transfer-basic/chaincode-(javascript, java, go, typescript)`

1.  Set up the blockchain network(先安裝好 npm)

    ```sh
    ## 啟動網路
    cd fabric-samples/test-network
    ./network.sh down
    ./network.sh up createChannel -c mychannel -ca
    ./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-javascript/ -ccl javascript
    ```

    ```sh
    ## 範例應用
    cd asset-transfer-basic/application-javascript
    npm install
    node app.js
    ```

    在第一步中啟動Fabric測試網絡時，建立了一個管理員用戶(admin)作為憑證頒發機構(CA)的註冊商。 我們的第一步是通過讓應用程序呼叫enrollAdmin來生成用於管理的公私鑰和X.509證書。 此過程使用憑證簽名請求(CSR): 首先在本地生成公私鑰，然後將公鑰發送到CA，CA傳回編碼的憑證以供應用程式使用，然後這些憑證會儲存在錢包中，使我們能夠充當CA的管理員。

2.  The application enrolls the admin user

    需要注意的是，註冊管理員和註冊應用程式使用者是發生在應用程式和憑證頒發機構之間的互動，而不是應用程式和Chaincode之間的互動。

    ```js
    async function main() {
      try {
        // build an in memory object with the network configuration (also known as a connection profile)
        const ccp = buildCCP();
    
        // build an instance of the fabric ca services client based on
        // the information in the network configuration
        const caClient = buildCAClient(FabricCAServices, ccp);
    
        // setup the wallet to hold the credentials of the application user
        const wallet = await buildWallet(Wallets, walletPath);
    
        // in a real application this would be done on an administrative flow, and only once
        await enrollAdmin(caClient, wallet);
    	// ...
    ```

    >```
    >Wallet path: /Users/<your_username>/fabric-samples/asset-transfer-basic/application-javascript/wallet
    >Successfully enrolled admin user and imported it into the wallet
    >```

    該程式將CA管理員的憑證儲存在錢包目錄中。你可以在`wallet/admin.id` 文件中找到管理員的憑證和私鑰。

    (注！重啟需要刪掉wallet資料夾)

3.  The application registers and enrolls an application user

    ```js
        // in a real application this would be done only when a new user was required to be added
        // and would be part of an administrative flow
        await registerUser(caClient, wallet, userId, 'org1.department1');
        // ...
    ```

    >   ```
    >   Successfully registered and enrolled user appUser and imported it into the wallet
    >   ```

    與管理員註冊類似，這個功能使用CSR來註冊appUser，並將其憑證與管理員的憑證一起儲存在錢包中。現在我們有了兩個獨立用戶的身份admin和appUser

4.  The sample application prepares a connection to the channel and smart contract

    ```js
    // Create a new gateway instance for interacting with the fabric network.
    // In a real application this would be done as the backend server session is setup for
    // a user that has been verified.
    const gateway = new Gateway();
    
    try {
      // setup the gateway instance
      // The user will now be able to create connections to the fabric network and be able to
      // submit transactions and query. All transactions submitted by this gateway will be
      // signed by this user using the credentials stored in the wallet.
      await gateway.connect(ccp, {
        wallet,
        identity: userId,
        discovery: {enabled: true, asLocalhost: true} // using asLocalhost as this gateway is using a fabric network deployed locally
      });
    
      // Build a network instance based on the channel where the smart contract is deployed
      const network = await gateway.getNetwork(channelName);
    
    
      // Get the contract from the network.
      const contract = network.getContract(chaincodeName);
      // When a chaincode package includes multiple smart contracts,
      // on the getContract() API you can specify both the name of the chaincode package
      // and a specific smart contract to target. For example:
      // const contract = await network.getContract('chaincodeName', 'smartContractName');
    ```

    應用程式正在通過閘道使用合約名稱和channel名稱來引用合約

5.  The application initializes the ledger with some sample data

    ```js
    // Initialize a set of asset data on the channel using the chaincode 'InitLedger' function.
    // This type of transaction would only be run once by an application the first time it was started after it
    // deployed the first time. Any updates to the chaincode deployed later would likely not need to run
    // an "init" type function.
    console.log('\n--> Submit Transaction: InitLedger, function creates the initial set of assets on the ledger');
    await contract.submitTransaction('InitLedger');
    console.log('*** Result: committed');
    ```

    >```
    >Submit Transaction: InitLedger, function creates the initial set of assets on the ledger
    >```

    `submitTransaction()`函數用於呼叫chaincode的`InitLedger`函數，用一些樣本資料填充帳本。在幕後，`submitTransaction()`函數將使用服務發現為chaincode找到一組所需的背書節點，在所需數量的peer上呼叫chaincode，從這些peer中收集chaincode背書結果，最後將交易提交給ordering service。

    ```js
    async InitLedger(ctx) {
         const assets = [
             {
                 ID: 'asset1',
                 Color: 'blue',
                 Size: 5,
                 Owner: 'Tomoko',
                 AppraisedValue: 300,
             },
             {
                 ID: 'asset2',
                 Color: 'red',
                 Size: 5,
                 Owner: 'Brad',
                 AppraisedValue: 400,
             },
             {
                 ID: 'asset3',
                 Color: 'green',
                 Size: 10,
                 Owner: 'Jin Soo',
                 AppraisedValue: 500,
             },
             {
                 ID: 'asset4',
                 Color: 'yellow',
                 Size: 10,
                 Owner: 'Max',
                 AppraisedValue: 600,
             },
             {
                 ID: 'asset5',
                 Color: 'black',
                 Size: 15,
                 Owner: 'Adriana',
                 AppraisedValue: 700,
             },
             {
                 ID: 'asset6',
                 Color: 'white',
                 Size: 15,
                 Owner: 'Michel',
                 AppraisedValue: 800,
             },
         ];
    
         for (const asset of assets) {
             asset.docType = 'asset';
             await ctx.stub.putState(asset.ID, Buffer.from(JSON.stringify(asset)));
             console.info(`Asset ${asset.ID} initialized`);
         }
     }
    ```

    

6.  The application invokes each of the chaincode functions

    `evaluateTransaction()`函數用於當你想查詢單個peer，而不向ordering service 提交交易時。

    ```js
    // Let's try a query type operation (function).
    // This will be sent to just one peer and the results will be shown.
    console.log('\n--> Evaluate Transaction: GetAllAssets, function returns all the current assets on the ledger');
    let result = await contract.evaluateTransaction('GetAllAssets');
    console.log(`*** Result: ${prettyJSONString(result.toString())}`);
    ```

    ```js
    // GetAllAssets returns all assets found in the world state.
     async GetAllAssets(ctx) {
         const allResults = [];
         // range query with empty string for startKey and endKey does an open-ended query of all assets in the chaincode namespace.
         const iterator = await ctx.stub.getStateByRange('', '');
         let result = await iterator.next();
         while (!result.done) {
             const strValue = Buffer.from(result.value.value.toString()).toString('utf8');
             let record;
             try {
                 record = JSON.parse(strValue);
             } catch (err) {
                 console.log(err);
                 record = strValue;
             }
             allResults.push({ Key: result.value.key, Record: record });
             result = await iterator.next();
         }
         return JSON.stringify(allResults);
     }
    ```

    >```
    >  Evaluate Transaction: GetAllAssets, function returns all the current assets on the ledger
    >  Result: [...]
    >```

    CreateAsset，asset13

    ```js
    // Now let's try to submit a transaction.
    // This will be sent to both peers and if both peers endorse the transaction, the endorsed proposal will be sent
    // to the orderer to be committed by each of the peer's to the channel ledger.
    console.log('\n--> Submit Transaction: CreateAsset, creates new asset with ID, color, owner, size, and appraisedValue arguments');
    await contract.submitTransaction('CreateAsset', 'asset13', 'yellow', '5', 'Tom', '1300');
    console.log('*** Result: committed');
    ```

    ```js
    // CreateAsset issues a new asset to the world state with given details.
    async CreateAsset(ctx, id, color, size, owner, appraisedValue) {
      const asset = {
          ID: id,
          Color: color,
          Size: size,
          Owner: owner,
          AppraisedValue: appraisedValue,
      };
      return ctx.stub.putState(id, Buffer.from(JSON.stringify(asset)));
    }
    ```

    >```
    >Submit Transaction: CreateAsset, creates new asset with ID, color, owner, size, and appraisedValue arguments
    >```

    ReadAssest，assest13

    ```js
    console.log('\n--> Evaluate Transaction: ReadAsset, function returns an asset with a given assetID');
    result = await contract.evaluateTransaction('ReadAsset', 'asset13');
    console.log(`*** Result: ${prettyJSONString(result.toString())}`);
    ```

    ```js
    // ReadAsset returns the asset stored in the world state with given id.
    async ReadAsset(ctx, id) {
      const assetJSON = await ctx.stub.getState(id); // get the asset from chaincode state
      if (!assetJSON || assetJSON.length === 0) {
          throw new Error(`The asset ${id} does not exist`);
      }
      return assetJSON.toString();
    }
    ```

    >```
    >Evaluate Transaction: ReadAsset, function returns an asset with a given assetID
    >Result: {
    >  "ID": "asset13",
    >  "Color": "yellow",
    >  "Size": "5",
    >  "Owner": "Tom",
    >  "AppraisedValue": "1300"
    >}
    >```

    `AssetExists` ->`UpdateAsse` ->`ReadAsset` ，assest1

    ```js
    console.log('\n--> Evaluate Transaction: AssetExists, function returns "true" if an asset with given assetID exist');
    result = await contract.evaluateTransaction('AssetExists', 'asset1');
    console.log(`*** Result: ${prettyJSONString(result.toString())}`);
    
    console.log('\n--> Submit Transaction: UpdateAsset asset1, change the appraisedValue to 350');
    await contract.submitTransaction('UpdateAsset', 'asset1', 'blue', '5', 'Tomoko', '350');
    console.log('*** Result: committed');
    
    console.log('\n--> Evaluate Transaction: ReadAsset, function returns "asset1" attributes');
    result = await contract.evaluateTransaction('ReadAsset', 'asset1');
    console.log(`*** Result: ${prettyJSONString(result.toString())}`);
    ```

    ```js
    // AssetExists returns true when asset with given ID exists in world state.
       async AssetExists(ctx, id) {
           const assetJSON = await ctx.stub.getState(id);
           return assetJSON && assetJSON.length > 0;
       }
    // UpdateAsset updates an existing asset in the world state with provided parameters.
       async UpdateAsset(ctx, id, color, size, owner, appraisedValue) {
           const exists = await this.AssetExists(ctx, id);
           if (!exists) {
               throw new Error(`The asset ${id} does not exist`);
           }
    
           // overwriting original asset with new asset
           const updatedAsset = {
               ID: id,
               Color: color,
               Size: size,
               Owner: owner,
               AppraisedValue: appraisedValue,
           };
           return ctx.stub.putState(id, Buffer.from(JSON.stringify(updatedAsset)));
       }
     // ReadAsset returns the asset stored in the world state with given id.
     async ReadAsset(ctx, id) {
         const assetJSON = await ctx.stub.getState(id); // get the asset from chaincode state
         if (!assetJSON || assetJSON.length === 0) {
             throw new Error(`The asset ${id} does not exist`);
         }
         return assetJSON.toString();
     }
    ```

    >```
    >Evaluate Transaction: AssetExists, function returns "true" if an asset with given assetID exist
    >Result: true
    >
    >Submit Transaction: UpdateAsset asset1, change the appraisedValue to 350
    >
    >Evaluate Transaction: ReadAsset, function returns "asset1" attributes
    >Result: {
    >  "ID": "asset1",
    >  "Color": "blue",
    >  "Size": "5",
    >  "Owner": "Tomoko",
    >  "AppraisedValue": "350"
    >}
    >```

    `TransferAsset`->`ReadAsset`

    ```js
    console.log('\n--> Submit Transaction: TransferAsset asset1, transfer to new owner of Tom');
    await contract.submitTransaction('TransferAsset', 'asset1', 'Tom');
    console.log('*** Result: committed');
    
    console.log('\n--> Evaluate Transaction: ReadAsset, function returns "asset1" attributes');
    result = await contract.evaluateTransaction('ReadAsset', 'asset1');
    console.log(`*** Result: ${prettyJSONString(result.toString())}`);
    ```

    >```
    >Submit Transaction: TransferAsset asset1, transfer to new owner of Tom
    >Evaluate Transaction: ReadAsset, function returns "asset1" attributes
    >Result: {
    >  "ID": "asset1",
    >  "Color": "blue",
    >  "Size": "5",
    >  "Owner": "Tom",
    >  "AppraisedValue": "350"
    >}
    >```

7.  clean up

    ```sh
    ./network.sh down
    ```

    

## Commercial paper tutorial(Lab4)

情境：MagnetoCorp 和 DigiBank 這兩個組織使用PaperNet(Hyperledger Fabric區塊鏈網路)進行商業票據交易。建立測試網路後，MagnetoCorp 的員工 Isabella，將代表該公司發行商業票據， 然後DigiBank的員工 Balaji，將購買此商業票據，持有一段時間，然後向 MagnetoCorp 贖回以獲取少許利潤。

1.  Prerequisites

    -   node.js
    -   vscode(用 code 來看程式碼時需要) 

2.  Create the network

    ```sh
    cd fabric-samples/commercial-paper
    ./network-starter.sh
    docker ps
    docker network inspect net_test
    ```

    8個 container，

    -   The Org1 peer, `peer0.org1.example.com`, **DigiBank**
    -   The Org2 peer, `peer0.org2.example.com`, **MagnetoCorp**
    -   The CouchDB database for the Org1 peer, `couchdb0`
    -   The CouchDB database for the Org2 peer, `couchdb1`
    -   The Ordering node, `orderer.example.com`
    -   The Org1 CA, `ca_org1`
    -   The Org2 CA, `ca_org2`
    -   The Ordering Org CA

3.  Monitor the network as MagnetoCorp

    ```sh
    ## open new window
    cd organization/magnetocorp
    ./configuration/cli/monitordocker.sh net_test
    ## if default port in use -> ./configuration/cli/monitordocker.sh net_test <port_number>
    ```

4.  Examine the commercial paper smart contract

    ```sh
    ## open new window
    cd organization/magnetocorp
    code contract
    ```

    `lib/papercontract.js`: 合約內容

    -   `const { Contract, Context } = require('fabric-contract-api');`:  Contract & Context
    -   `class CommercialPaperContract extends Contract {`: 裡面定義交易關鍵的函數，e.g. issue, buy, transfer

5.  Deploy the smart contract to the channel

    我們需要以MagnetoCorp和DigiBank的管理員身份安裝和同意chaincode。

    ```sh
    ## MagnetoCorp(open new window)
    cd organization/magnetocorp
    ## set the environment variables
    source magnetocorp.sh 
    ## package
    peer lifecycle chaincode package cp.tar.gz --lang node --path ./contract --label cp_0
    ## install
    peer lifecycle chaincode install cp.tar.gz
    ## query
    peer lifecycle chaincode queryinstalled
    ## package id 每個人都不一樣
    export PACKAGE_ID=cp_0:ddca913c004eb34f36dfb0b4c0bcc6d4afc1fa823520bb5966a3bfcf1808f40a
    ## approve
    peer lifecycle chaincode approveformyorg --orderer localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name papercontract -v 0 --package-id $PACKAGE_ID --sequence 1 --tls --cafile $ORDERER_CA
    ```

    ```sh
    ## DigiBank(open new window)
    cd organization/digibank
    ## set the environment variables
    source digibank.sh 
    ## package
    peer lifecycle chaincode package cp.tar.gz --lang node --path ./contract --label cp_0
    ## install
    peer lifecycle chaincode install cp.tar.gz
    ## query
    peer lifecycle chaincode queryinstalled
    ## package id 每個人都不一樣
    export PACKAGE_ID=cp_0:ddca913c004eb34f36dfb0b4c0bcc6d4afc1fa823520bb5966a3bfcf1808f40a
    ## approve
    peer lifecycle chaincode approveformyorg --orderer localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name papercontract -v 0 --package-id $PACKAGE_ID --sequence 1 --tls --cafile $ORDERER_CA
    ```

    ```sh
    ## Commit(當兩方都同意了，任意一方都可以commit，這裡繼續使用DigiBank)
    peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --peerAddresses localhost:7051 --tlsRootCertFiles ${PEER0_ORG1_CA} --peerAddresses localhost:9051 --tlsRootCertFiles ${PEER0_ORG2_CA} --channelID mychannel --name papercontract -v 0 --sequence 1 --tls --cafile $ORDERER_CA --waitForEvent
    ## commit後啟動兩個chaincode container
    docker ps
    ```

6.  Magnetocorp application - issue

    ![issue 流程](https://hyperledger-fabric.readthedocs.io/en/latest/_images/commercial_paper.diagram.8.png)

    1.  wallet -> Isabella(ap): retrieve
    2.  isabella(ap) -> gateway: submit
    3.  gateway <-> peer: propose/endorse
    4.  gateway -> orderer: order
    5.  order -> peer: distribute
    6.  peer -> gateway: notify
    7.  gateway -> Isabella(ap): response

    ```sh
    ## Magnetocorp(issabela)
    cd organization/magnetocorp
    code application
    npm install
    ```

    `issue.js`: 

    -   `const { Wallets, Gateway } = require('fabric-network');`: Wallets & Gateway, Key SDK classes
    -   `const wallet = await Wallets.newFileSystemWallet('../identity/user/isabella/wallet');`: 使用isabella錢包
    -   `await gateway.connect(connectionProfile, connectionOptions);`: 連接到閘道
    -   `const network = await gateway.getNetwork('mychannel');`: 連接到myChannel網路
    -   `const contract = await network.getContract('papercontract');`: 存取 papercontract 合約
    -   `const issueResponse = await contract.submitTransaction('issue', 'MagnetoCorp', '00001', ...);`: 發issue交易
    -   `let paper = CommercialPaper.fromBuffer(issueResponse);`: response

    在 isabella 的 wallet 中產生 X.509 certificate，然後執行`issue.js`，結果：`MagnetoCorp commercial paper : 00001 successfully issued for value 5000000`

    ```sh
    ## 運行在PaperNet上的MagnetoCorp憑證頒發機構ca_org2有一個應用程式使用者，該使用者是在部署網路時註冊的。Isabella可以使用身份名稱和秘密(enrollmentSecret)為issue.js應用產生X.509加密材料。使用CA產生客戶端加密材料的過程被稱為註冊。在一個實際場景中，網路營運商向應用程式開發者提供CA註冊要的客戶端身份名稱和秘密，然後開發人員將使用該信物來註冊他們的應用程式並與網路互動。
    
    ## enrollUser.js 使用fabric-ca-client來生成公私鑰對，然後向CA發出憑證簽署請求。如果Isabella提交的使用者和秘密與CA註冊的信物相匹配，CA就會簽發一份憑證，確定Isabella屬於MagnetoCorp。當簽名請求完成後，enrollUser.js會將私鑰和簽名憑證存儲存在Isabella的錢包裡。
    node enrollUser.js
    cat ../identity/user/isabella/wallet/*
    node issue.js
    ## 應用程式呼叫 papercontract.js 中的 CommercialPaper 智能合約中定義的 issue交易。智能合約通過 Fabric API 與帳本進行互動，最主要的是 putState() 和 getState()，將新的商業票據表示為世界狀態中的一個向量狀態。
    ```

7.  Digibank application - buy

    ```sh
    ## Digibank(Balaji)
    cd organization/digibank/application/
    code buy.js
    npm install
    ```

    `buy.js`:

    -   整體來說和 `issue.js`很像
    -   主要差別在`const buyResponse = await contract.submitTransaction('buy', 'MagnetoCorp', '00001', ...);`

    在 Balaji 的 wallet 中產生 X.509 certificate，然後執行`buy.js`，

    ```sh
    node enrollUser.js
    cat ../identity/user/balaji/wallet/*
    node buy.js
    ## 看到程式輸出，MagnetoCorp 商業票據 00001 被 Balaji 代表 DigiBank 成功購買。buy.js 呼叫了CommercialPaper智能合約中定義的buy交易，該交易使用 putState() 和 getState() Fabric API 更新了世界狀態中的商業票據00001。
    ```

8.  Digibank application - redeem

    ```sh
    node redeem.js
    ## 查詢歷史
    node queryapp.js
    ```

9.  Clean up

    ```sh
    cd fabric-samples/commercial-paper
    ./network-clean.sh
    ```

    