# Hyperledger Fabric

[tutorial](https://hyperledger-fabric.readthedocs.io/en/latest/tutorials.html)

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

   - 為了確保交易的有效性，使用智慧合約創建的交易通常需要由多個組織簽署(多重簽名)才能提交到渠道分類賬。為了簽署交易，每個組織需要在其對等體上調用並執行智慧合約，然後簽署交易的輸出。
- 指定通道上需要執行智慧合約的集合組織的策略被稱為背書策略，作為Chaincode definition一部分由Chaincode設定
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
      ```
    
      ```

      ```
    
      ```

    3. Get All Assets
      
        ```sh
    peer Chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
      ```
      ```
    
      ```

    4. Transfer Assets
      
        ```sh
    peer Chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"TransferAsset","Args":["asset6","Christopher"]}'
      ```
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

在Hyperledger Fabric中，智慧合約部署在稱為Chaincode的package中。想要驗證交易或查詢帳本的組織需要在他們的peers身上安裝Chaincode。在連接到channel的peernode上安裝了Chaincode之後，channel成員可以將Chaincode部署到channel，並使用Chaincode中的智慧合約建立或更新通道分類帳上的資產。

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

    

## Commercial paper tutorial(Lab 4)

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
    ## 應用程式呼叫 papercontract.js 中的 CommercialPaper 智慧合約中定義的 issue交易。智慧合約通過 Fabric API 與帳本進行互動，最主要的是 putState() 和 getState()，將新的商業票據表示為世界狀態中的一個向量狀態。
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
    ## 看到程式輸出，MagnetoCorp 商業票據 00001 被 Balaji 代表 DigiBank 成功購買。buy.js 呼叫了CommercialPaper智慧合約中定義的buy交易，該交易使用 putState() 和 getState() Fabric API 更新了世界狀態中的商業票據00001。
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

## Using Private Data in Fabric(Lab 5)

本教學演示使用私有資料集合(PDC)為組織的授權peer提供區塊鏈網路上私有資料的儲存和檢索，使用包含管理該集合的策略的"集合定義文件"指定該集合。

1. Asset transfer private data sample use case

    這個案例演示了使用三個私有資料集合 assetCollection、Org1MSPPrivateCollection 和 Org2MSPPrivateCollection 在 Org1 和 Org2 之間轉移資產，使用的是以下案例

    -   Org1 的一個成員創建了一個新的資產，以下簡稱為擁有者。資產的公開細節，包括擁有者的身份，被存儲在名為 assetCollection 的私有資料集合中。資產的建立也包含了由擁有者提供的估價值。估價值由每個參與者用來同意資產的轉讓，並且只儲存在擁有者組織的集合中。在案例中，擁有者同意的初始估價值會儲存在 Org1MSPPrivateCollection 中。

    -   要購買資產，買方需要同意資產擁有者估價值。在這一步驟中，買方(Org2的成員)使用智慧合約函數 `AgreeToTransfer`建立一個交易協議，並同意一個估價值。這個值會儲存在 Org2MSPPateCollection 集合中。然後資產擁有者可以使用智慧合約函數`TransferAsset`將資產轉讓給買方。`TransferAsset`函數在轉讓資產之前，使用channel帳本上的hash值來確認資產擁有者和買方已經同意相同的估價價值。

    

2. Build a collection definition JSON file

    在一組組織使用私有資料進行交易之前，channel 上的所有組織都需要建立一個集合定義文件，定義與每個 chaincode 相關的私有資料集合。儲存在私有資料集合中的資料只分配給某些組織的peer，而不是channel的所有成員。集合定義文件描述了組織可以從 chaincode 中讀取和寫入的所有私有資料集合。例如：

    (Policy:  Defines the organization peers allowed to persist the collection data.)

    ```json
    // collections_config.json
    
    [
       {
       "name": "assetCollection",
       "policy": "OR('Org1MSP.member', 'Org2MSP.member')",
       "requiredPeerCount": 1,
       "maxPeerCount": 1,
       "blockToLive":1000000,
       "memberOnlyRead": true,
       "memberOnlyWrite": true
       },
       {
       "name": "Org1MSPPrivateCollection",
       "policy": "OR('Org1MSP.member')",
       "requiredPeerCount": 0,
       "maxPeerCount": 1,
       "blockToLive":3,
       "memberOnlyRead": true,
       "memberOnlyWrite": false,
       "endorsementPolicy": {
           "signaturePolicy": "OR('Org1MSP.member')"
       }
       },
       {
       "name": "Org2MSPPrivateCollection",
       "policy": "OR('Org2MSP.member')",
       "requiredPeerCount": 0,
       "maxPeerCount": 1,
       "blockToLive":3,
       "memberOnlyRead": true,
       "memberOnlyWrite": false,
       "endorsementPolicy": {
           "signaturePolicy": "OR('Org2MSP.member')"
       }
       }
    ]
    ```

    所有使用chaincode的組織都需要部署同一個集合定義文件，即使該組織不屬於任何集合。除了在集合文件中明確定義的集合之外，每個組織還可以訪問其peer上的隱式集合，該集合只能由其組織讀取。

    

3. Read and Write private data using chaincode APIs

    資料格式定義在chaincode中

    ```go
    // Peers in Org1 and Org2 will have this private data in a side database
    type Asset struct {
           Type  string `json:"objectType"` //Type is used to distinguish the various types of objects in state database
           ID    string `json:"assetID"`
           Color string `json:"color"`
           Size  int    `json:"size"`
           Owner string `json:"owner"`
    }
    
    // AssetPrivateDetails describes details that are private to owners
    
    // Only peers in Org1 will have this private data in a side database
    type AssetPrivateDetails struct {
           ID             string `json:"assetID"`
           AppraisedValue int    `json:"appraisedValue"`
    }
    
    // Only peers in Org2 will have this private data in a side database
    type AssetPrivateDetails struct {
           ID             string `json:"assetID"`
           AppraisedValue int    `json:"appraisedValue"`
    }
    
    ```

    -   Reading collection data
        -   智慧合約使用 chaincode API `GetPrivateData()`來查詢資料庫中的私有資料
        -   **ReadAsset** for querying the values of the `assetID, color, size and owner` attributes.
        -   **ReadAssetPrivateDetails** for querying the values of the `appraisedValue` attribute.
    -   Writing private data
        -   智慧合約使用 chaincode API `PutPrivateData()`來儲存私有資料到資料庫中

4.  Start the network

    ```sh
    cd fabric-samples/test-network
    ./network.sh down
    ./network.sh up createChannel -ca -s couchdb
    ```

5.  Deploy the private data smart contract to the channel

    ```sh
    ## package -> install -> approveformyorg -> commit -> querrycommit
    ./network.sh deployCC -ccn private -ccp ../asset-transfer-private-data/chaincode-go/ -ccl go -ccep "OR('Org1MSP.peer','Org2MSP.peer')" -cccg ../asset-transfer-private-data/chaincode-go/collections_config.json
    ```

6.  Register identities

    使用Org1和Org2的CA註冊兩個新身份，然後使用CA生成每個身份的憑證和私鑰。

    ```sh
    export PATH=${PWD}/../bin:${PWD}:$PATH
    export FABRIC_CFG_PATH=$PWD/../config/
    
    ## Org1
    export FABRIC_CA_CLIENT_HOME=${PWD}/organizations/peerOrganizations/org1.example.com/
    
    ## register a new owner client identity
    fabric-ca-client register --caname ca-org1 --id.name owner --id.secret ownerpw --id.type client --tls.certfiles ${PWD}/organizations/fabric-ca/org1/tls-cert.pem
    
    ## enroll(generate the identity certificates and MSP folder)
    fabric-ca-client enroll -u https://owner:ownerpw@localhost:7054 --caname ca-org1 -M ${PWD}/organizations/peerOrganizations/org1.example.com/users/owner@org1.example.com/msp --tls.certfiles ${PWD}/organizations/fabric-ca/org1/tls-cert.pem
    
    ##  copy the Node OU configuration file
    cp ${PWD}/organizations/peerOrganizations/org1.example.com/msp/config.yaml ${PWD}/organizations/peerOrganizations/org1.example.com/users/owner@org1.example.com/msp/config.yaml
    
    ## Org2
    export FABRIC_CA_CLIENT_HOME=${PWD}/organizations/peerOrganizations/org2.example.com/
    ## register a new owner client identity
    fabric-ca-client register --caname ca-org2 --id.name buyer --id.secret buyerpw --id.type client --tls.certfiles ${PWD}/organizations/fabric-ca/org2/tls-cert.pem
    
    ## enroll(generate the identity certificates and MSP folder)
    fabric-ca-client enroll -u https://buyer:buyerpw@localhost:8054 --caname ca-org2 -M ${PWD}/organizations/peerOrganizations/org2.example.com/users/buyer@org2.example.com/msp --tls.certfiles ${PWD}/organizations/fabric-ca/org2/tls-cert.pem
    
    ##  copy the Node OU configuration file
    cp ${PWD}/organizations/peerOrganizations/org2.example.com/msp/config.yaml ${PWD}/organizations/peerOrganizations/org2.example.com/users/buyer@org2.example.com/msp/config.yaml
    ```

7.  Create an asset in private data

    -   私有資料是通過`--transient`傳遞的，而且需要使用二進制，因此下面把它編碼程 base64

    ```sh
    export PATH=${PWD}/../bin:$PATH
    export FABRIC_CFG_PATH=$PWD/../config/
    
    ## Org1
    export CORE_PEER_TLS_ENABLED=true
    export CORE_PEER_LOCALMSPID="Org1MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/owner@org1.example.com/msp
    export CORE_PEER_ADDRESS=localhost:7051
    
    ## CreateAssest
    export ASSET_PROPERTIES=$(echo -n "{\"objectType\":\"asset\",\"assetID\":\"asset1\",\"color\":\"green\",\"size\":20,\"appraisedValue\":100}" | base64 | tr -d \\n)
    peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"CreateAsset","Args":[]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"
    ```

8.  Query the private data as an authorized peer

    ReadAsset & ReadAssetPrivateDetails chaincode

    ```go
    // ReadAsset reads the information from collection
    func (s *SmartContract) ReadAsset(ctx contractapi.TransactionContextInterface, assetID string) (*Asset, error) {
    
         log.Printf("ReadAsset: collection %v, ID %v", assetCollection, assetID)
         assetJSON, err := ctx.GetStub().GetPrivateData(assetCollection, assetID) //get the asset from chaincode state
         if err != nil {
             return nil, fmt.Errorf("failed to read asset: %v", err)
         }
    
         //No Asset found, return empty response
         if assetJSON == nil {
             log.Printf("%v does not exist in collection %v", assetID, assetCollection)
             return nil, nil
         }
    
         var asset *Asset
         err = json.Unmarshal(assetJSON, &asset)
         if err != nil {
             return nil, fmt.Errorf("failed to unmarshal JSON: %v", err)
         }
    
         return asset, nil
    
     }
    
    // ReadAssetPrivateDetails reads the asset private details in organization specific collection
    func (s *SmartContract) ReadAssetPrivateDetails(ctx contractapi.TransactionContextInterface, collection string, assetID string) (*AssetPrivateDetails, error) {
         log.Printf("ReadAssetPrivateDetails: collection %v, ID %v", collection, assetID)
         assetDetailsJSON, err := ctx.GetStub().GetPrivateData(collection, assetID) // Get the asset from chaincode state
         if err != nil {
             return nil, fmt.Errorf("failed to read asset details: %v", err)
         }
         if assetDetailsJSON == nil {
             log.Printf("AssetPrivateDetails for %v does not exist in collection %v", assetID, collection)
             return nil, nil
         }
    
         var assetDetails *AssetPrivateDetails
         err = json.Unmarshal(assetDetailsJSON, &assetDetails)
         if err != nil {
             return nil, fmt.Errorf("failed to unmarshal JSON: %v", err)
         }
    
         return assetDetails, nil
     }
    
    ```

    ```sh
    ## query
    peer chaincode query -C mychannel -n private -c '{"function":"ReadAsset","Args":["asset1"]}'
    peer chaincode query -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org1MSPPrivateCollection","asset1"]}'
    ```

9.  Query the private data as an unauthorized peer

    ```sh
    ## Org 2
    export CORE_PEER_LOCALMSPID="Org2MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/buyer@org2.example.com/msp
    export CORE_PEER_ADDRESS=localhost:9051
    
    ## qeury -> ok
    peer chaincode query -C mychannel -n private -c '{"function":"ReadAsset","Args":["asset1"]}'
    
    ## qeury Org2 -> empty
    peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org2MSPPrivateCollection","asset1"]}'
    
    ## qeury Org1 -> fail
    peer chaincode query -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org1MSPPrivateCollection","asset1"]}'
    ```

10.  Transfer the Asset

     ```sh
     ## Org2
     ## AgreeToTransfer
     export ASSET_VALUE=$(echo -n "{\"assetID\":\"asset1\",\"appraisedValue\":100}" | base64 | tr -d \\n)
     peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"AgreeToTransfer","Args":[]}' --transient "{\"asset_value\":\"$ASSET_VALUE\"}"
     
     ## ReadAssetPrivateDetails
     peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org2MSPPrivateCollection","asset1"]}'
     ```

     在smartcontract 利用 hash 確認 owner appraise = buyer appraise ，以此方式做到 blind auction

     ```sh
     ## Org1
     export CORE_PEER_LOCALMSPID="Org1MSP"
     export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/owner@org1.example.com/msp
     export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
     export CORE_PEER_ADDRESS=localhost:7051
     
     ## ReadTransferAgreement
     peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"ReadTransferAgreement","Args":["asset1"]}'
     
     ## Transfer
     export ASSET_OWNER=$(echo -n "{\"assetID\":\"asset1\",\"buyerMSP\":\"Org2MSP\"}" | base64 | tr -d \\n)
     peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"TransferAsset","Args":[]}' --transient "{\"asset_owner\":\"$ASSET_OWNER\"}" --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
     
     ## query
     peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"ReadAsset","Args":["asset1"]}'
     
     ## query -> empty
     peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org1MSPPrivateCollection","asset1"]}'
     
     ```

11.  Purge Private Data

     ```sh
     ## Org2
     export CORE_PEER_LOCALMSPID="Org2MSP"
     export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
     export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/buyer@org2.example.com/msp
     export CORE_PEER_ADDRESS=localhost:9051
     
     ## 還查的到估價值
     peer chaincode query -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org2MSPPrivateCollection","asset1"]}'
     
     ## 查詢logs看目前 block 高度
     docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'
     
     ## 發三次交易之前的記錄就會清除(因為設定blockToLive: 3)
     export ASSET_PROPERTIES=$(echo -n "{\"objectType\":\"asset\",\"assetID\":\"asset2\",\"color\":\"blue\",\"size\":30,\"appraisedValue\":100}" | base64 | tr -d \\n)
     peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"CreateAsset","Args":[]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"
     export ASSET_PROPERTIES=$(echo -n "{\"objectType\":\"asset\",\"assetID\":\"asset3\",\"color\":\"red\",\"size\":25,\"appraisedValue\":100}" | base64 | tr -d \\n)
     peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"CreateAsset","Args":[]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"
     export ASSET_PROPERTIES=$(echo -n "{\"objectType\":\"asset\",\"assetID\":\"asset4\",\"color\":\"orange\",\"size\":15,\"appraisedValue\":100}" | base64 | tr -d \\n)
     peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n private -c '{"function":"CreateAsset","Args":[]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"
     
     ## 查詢logs看目前 block 高度
     docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'
     
     ## 查不到估價值了
     peer chaincode query -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org2MSPPrivateCollection","asset1"]}'
     
     ```

12.  Using indexes with private data

     索引也可以應用於私有資料集合，通過在chaincode旁邊打包`META-INF/statedb/couchdb/collections/<collection_name>/indexes`目錄中的索引。[example](https://github.com/hyperledger/fabric-samples/blob/master//asset-transfer-private-data/chaincode-go/META-INF/statedb/couchdb/collections/assetCollection/indexes/indexOwner.json)

     為了將 chaincode 部署到正式環境中，建議在寫 chaincode 的同時定義任何索引，以便在 chaincode 安裝到 peer 並在channel實例化後，將chaincode和索引作為一個單元自動部署。當指定`--collections-config`指向集合JSON文件的位置時，關聯的索引將在channel上的chaincode實例化時自動部署。

13.  Clean up

     ```sh
     ./network.sh down
     ```

14.  Additional resources

     -   註1：測試後 query 不需要加 -o 那些有關 orderer 的參數也可以正確執行，而需要提交到帳本的動作，因為觸發到交易則需要
     -   註2：peerAddresses 參數在測試後看起來只有在 commit 的時候有用

     [video](https://youtu.be/qyjDi93URJE)

## Secured asset transfer in Fabric(Lab 6)

本教學將演示如何在Hyperledger Fabric區塊鏈channel的組織之間表示和交易資產，同時使用私有資料來保持資產和交易的細節。每個鏈上資產都是一個不可替換的令牌(NFT)，它代表一個特定的資產，具有特定的不可變元資料屬性(比如大小和顏色)，並且有一個唯一的擁有者。當擁有者想要出售資產時，需要雙方同意相同的價格才能轉讓資產。私人資產轉移智慧合約強制規定只有資產的擁有者才能轉移資產。在實驗中將瞭解Fabric特性(如：基於狀態的背書、私有資料和存取控制)如何結合在一起，以提供既私密性又可驗證的安全交易。

說明：

-   私人資產轉讓的情形受以下要求的約束：
    -   資產可由第一個擁有者的組織發放(在現實世界中，發放可能僅限於證明資產屬性的某些機構)。
    -   所有權在組織層面進行管理(Fabric權限方案同樣支援組織內的個人身份層面之所有權)。
    -   資產標識符和擁有者被儲存為公開資料，供所有channel成員查看。
    -   然而資產元資料屬性是只有資產擁有者(以及之前的擁有者)才知道的私有資訊。
    -   有興趣的買家會想驗證資產的私有屬性。
    -   有興趣的買家會想驗證資產的出處，特別是資產的來源和保管鏈。他們還希望核實該資產自發行以來沒有發生變化，而且之前的所有轉讓都是合法的。
    -   要轉讓資產，買方和賣方必須在銷售價格達成一致。
    -   只有現任擁有者才可以將其資產轉讓給另一個組織。
    -   實際的私人資產轉讓必須確認轉讓的是合法的資產，並核實價格是否已經商定。買賣雙方必須認可轉讓。

-   智慧合約使用以下技術來確保資產屬性保持私有:
    -   資產元資料屬性僅儲存在當前擁有者的組織的隱式私有資料集合中。
    -   私有屬性包含salt的hash被自動儲存在鏈上公開供所有channel成員查看，加鹽使其他通道成員不能通過字典攻擊來猜測私有資料的原像。
    -   智慧合約請求利用私有資料的瞬態欄位，這樣私有資料就不會包含在最終的鏈上交易中。
    -   私有資料查詢的客戶端org id必須與資產所有者的org id相同。

-   如何實施資產轉移?

    1.  建立資產
        -   私有資產轉移的智慧合約部署了一個背書政策，該政策要求任何一個channel成員進行背書即可(自己也可)。
        -   使用chaincode級別的背書策略
        -   建立資產時將提交請求的組織ID(MSP ID)儲存在以key/value形式公開儲存在chaincode
        -   當有更新或轉移請求時先確認 MSP ID 是否來自同一個組織
        -   智慧合約為資產的key設置基於狀態的背書策略，防止其他組織使用被惡意修改的智慧合約來更新或轉移資產。

    2.  同意轉移
        -   在建立資產後，channel成員可以使用智慧合約來同意轉讓資產
            -   資產的擁有者可以更改公共擁有權記錄中的描述部分
            -   智慧合約存取控制要求這種變更需要由資產擁有者組織的成員提交
            -   基於狀態的背書策略強制要求該更改必須由擁有者組織的peer背書
        -   資產擁有者和買方同意以一定價格轉讓資產
            -   私有資料集合的背書策略保證了各自組織的peer認可價格協議
            -   智能合約存取控制邏輯保證了價格協議是由相關組織的客戶提交。
            -   買賣雙方約定的價格被儲存在每個組織的隱性私有資料集合中。
            -   每個價格協議的hash值都儲存在分類帳上，只有當兩個組織同意相同價格時才會匹配。
            -   每個價格協議的hash值都會加入隨機交易ID當作Salt防止猜測價格

    3.  轉移資產
        -   智能合約存取控制確保轉移必須由擁有資產的組織的成員發起。
        -   轉移函數會驗證資產Hash是否匹配，以確保資產擁有者出售的是自己擁有的同一資產。
        -   轉移函數使用帳本上價格協議的hash，以確保兩個組織同意相同的價格。
        -   如果滿足轉移條件，轉移函數將資產添加到買方的隱式私有資料集合中，並從賣方的集合中刪除資產，以及更新公有所有權記錄中的擁者。
        -   由於賣方和買方隱性資料集合的背書政策，以及公共記錄的基於狀態的背書政策（需要賣方背書），轉移需要由買方和賣方的peer背書。
        -   基於狀態的背書政策的公共資產記錄被更新，以便只有資產的新擁有者的peer可以更新或出售他們的新資產
        -   價格協議也從買賣雙方隱性私人資料集合中刪除，並且在每個私有資料集合中建力銷售收據

---

實驗：`asset-transfer-secured-agreement`(chaincode)

1.  Deploy the test network & smart contract

    ```sh
    cd fabric-samples/test-network
    ./network.sh down
    ./network.sh up createChannel -c mychannel
    ./network.sh deployCC -ccn secured -ccp ../asset-transfer-secured-agreement/chaincode-go/ -ccl go -ccep "OR('Org1MSP.peer','Org2MSP.peer')"
    ```

2.   Use two terminals to represent Org1 & Org2

    ```sh
    ## Org1
    export PATH=${PWD}/../bin:${PWD}:$PATH
    export FABRIC_CFG_PATH=$PWD/../config/
    export CORE_PEER_TLS_ENABLED=true
    export CORE_PEER_LOCALMSPID="Org1MSP"
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    export CORE_PEER_ADDRESS=localhost:7051
    ```

    ```sh
    ## Org2
    export PATH=${PWD}/../bin:${PWD}:$PATH
    export FABRIC_CFG_PATH=$PWD/../config/
    export CORE_PEER_TLS_ENABLED=true
    export CORE_PEER_LOCALMSPID="Org2MSP"
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    export CORE_PEER_ADDRESS=localhost:9051
    ```

3.  Create an asset

    ```sh
    ## Org1
    
    ## 資產細節
    export ASSET_PROPERTIES=$(echo -n "{\"object_type\":\"asset_properties\",\"asset_id\":\"asset1\",\"color\":\"blue\",\"size\":35,\"salt\":\"a94a8fe5ccb19ba61c4c0873d391e987982fbbd3\"}" | base64 | tr -d \\n)
    
    ## 建立資產
    peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"CreateAsset","Args":["asset1", "A new asset for Org1MSP"]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"
    
    ## 查詢隱性資料集合
    peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"GetAssetPrivateProperties","Args":["asset1"]}'
    
    ## 查詢公開所有權記錄
    peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"ReadAsset","Args":["asset1"]}'
    
    ## 更改描述部份
    peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"ChangePublicDescription","Args":["asset1","This asset is for sale"]}'
    
    ## 重新查詢公開所有權記錄
    peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"ReadAsset","Args":["asset1"]}'
    ```

    目前狀態(Key/Value)：

    -   Channel World State
        -   assest1/owner-Org1
        -   h(assest1)/h(details)
    -   Org1
        -   assest1/details

4.  Agree to sell the asset(通過 email 溝通細節...)

    ```sh
    ## Org1 sell
    
    ## 價格 $110
    export ASSET_PRICE=$(echo -n "{\"asset_id\":\"asset1\",\"trade_id\":\"109f4b3c50d7b0df729d299bc6f8e9ef9066971f\",\"price\":110}" | base64)
    
    ## AgreeToSell
    peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"AgreeToSell","Args":["asset1"]}' --transient "{\"asset_price\":\"$ASSET_PRICE\"}"
    ```

    ```sh
    ## Org2 buy
    
    ## 資產屬性
    export ASSET_PROPERTIES=$(echo -n "{\"object_type\":\"asset_properties\",\"asset_id\":\"asset1\",\"color\":\"blue\",\"size\":35,\"salt\":\"a94a8fe5ccb19ba61c4c0873d391e987982fbbd3\"}" | base64)
    
    ## VerifyAssetProperties
    peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"VerifyAssetProperties","Args":["asset1"]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"
    
    ## 價格 $100
    export ASSET_PRICE=$(echo -n "{\"asset_id\":\"asset1\",\"trade_id\":\"109f4b3c50d7b0df729d299bc6f8e9ef9066971f\",\"price\":100}" | base64)
    
    ## AgreeToBuy
    peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"AgreeToBuy","Args":["asset1"]}' --transient "{\"asset_price\":\"$ASSET_PRICE\"}"
    
    ## GetAssetBidPrice
    peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"GetAssetBidPrice","Args":["asset1"]}'
    ```

    目前狀態(Key/Value)：

    -   Channel World State
        -   assest1/owner-Org1
        -   h(assest1)/h(details)
        -   h(S:assest1)/h(110)
        -   h(B:assest1)/h(100)
    -   Org1
        -   assest1/details
        -   S:assest1/110
    -   Org2
        -   B:assest1/100

5.    Transfer the asset from Org1 to Org2

      下面的命令使用`--peerAddresses`，以Org1和Org2的peer為目標。這兩個組織都需要認可這次轉移。請注意，資產屬性和價格在轉移請求中作為瞬時屬性傳遞。傳遞這些屬性是為了讓當前擁有者可以確保以正確的價格轉移正確的資產。這些屬性將由兩個背書人根據鏈上hash值進行檢查。

      ```sh
      ## Org1
      
      ## TransferAsset -> Error 因為價格不同
      peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"TransferAsset","Args":["asset1","Org2MSP"]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\",\"asset_price\":\"$ASSET_PRICE\"}" --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
      
      ## AgreeToSell(Drop the price to $100)
      export ASSET_PRICE=$(echo -n "{\"asset_id\":\"asset1\",\"trade_id\":\"109f4b3c50d7b0df729d299bc6f8e9ef9066971f\",\"price\":100}" | base64)
      peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"AgreeToSell","Args":["asset1"]}' --transient "{\"asset_price\":\"$ASSET_PRICE\"}"
      
      ## TransferAsset
      peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"TransferAsset","Args":["asset1","Org2MSP"]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\",\"asset_price\":\"$ASSET_PRICE\"}" --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
      
      ## 查詢公開所有權記錄
      peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"ReadAsset","Args":["asset1"]}'
      ```

      ```sh
      ## Org2
      
      ## 查詢隱性資料集合
      peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"GetAssetPrivateProperties","Args":["asset1"]}'
      
      ## 更改描述部份
      peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"ChangePublicDescription","Args":["asset1","This asset is not for sale"]}'
      
      ## 查詢公開所有權記錄
      peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n secured -c '{"function":"ReadAsset","Args":["asset1"]}'
      
      ```

      

      目前狀態(Key/Value)：

      -   Channel World State
          -   assest1/owner-Org2
          -   h(assest1)/h(details)

          -   h(S:assest1)/h(100)
          -   h(B:assest1)/h(100)

      -   Org1
          -   S:assest1/100

      -   Org2
          -   assest1/details
          -   B:assest1/100

6.  Clean up

    ```sh
    ./network.sh down
    ```

    