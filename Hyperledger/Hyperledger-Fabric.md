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

3. Create channel: `./network.sh createChannel`，-c 取名:

4. Starting a chaincode on the channel: 

    1. ```sh
        ./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
        ```

    - 為了確保交易的有效性，使用智慧合約創建的交易通常需要由多個組織簽署(多重簽名)才能提交到渠道分類賬。為了簽署交易，每個組織需要在其對等體上調用並執行智慧合約，然後簽署交易的輸出。

- 指定通道上需要執行智慧合約的集合組織的策略被稱為背書策略，作為chaincode definition一部分由chaincode設定
    - 一個chaincode被安裝在組織的對等體上，然後部署到一個通道上，然後它可以用來認可交易並與區塊鏈分類賬進行互動。
    - 在將chaincode部署到通道之前，通道的成員需要對建立chaincode治理的chaincode definition達成一致。當所需數量的組織同意時，chaincode definition可以提交給channel，chaincode 就可以使用了。

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
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'
       ```
    
    3. Get All Assets

        ```sh
        peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
        ```

    4. Transfer Assets
    
        ```sh
    peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"TransferAsset","Args":["asset6","Christopher"]}'
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
        peer chaincode query -C mychannel -n basic -c '{"Args":["ReadAsset","asset6"]}'
        ```
    
6. Bring down the network

    > The command will stop and remove the node and chaincode containers, delete the organization crypto material, and remove the chaincode images from your Docker Registry. The command also removes the channel artifacts and docker volumes from previous runs, allowing you to run `./network.sh up` again if you encountered any problems.

    ```
    ./network.sh dow
    ```

## Deploying a smart contract to a channel(Lab 2)

在Hyperledger Fabric中，智慧合約部署在稱為chaincode的package中。想要驗證交易或查詢帳本的組織需要在他們的peers身上安裝chaincode。在連接到channel的peer上安裝了chaincode之後，channel成員可以將chaincode部署到channel，並使用chaincode中的智慧合約建立或更新通道分類帳上的資產。

chaincode生命週期的流程首先會將chaincode部署到通道上，然後允許多個組織在使用chaincode建立交易之前就操作chaincode的方式達成一致。

1. Startup network

    ```sh
    ./network.sh down
    ./network.sh up createChannel
    ```

2. Setup Logspout

    - 這個工具將來自不同Docker容器的輸出收集到一個位置，從而可以輕鬆地從單個視窗查看發生的情況。

    ```sh
    cd fabric-samples/test-network
    cp ../commercial-paper/organization/digibank/configuration/cli/monitordocker.sh .
    ./monitordocker.sh net_test
    ```

3. Package the smart contract

    ```sh
    cd fabric-samples/asset-transfer-basic/chaincode-go
    cat go.mod
    GO111MODULE=on go mod vendor
    cd ../../test-network
    export PATH=${PWD}/../bin:$PATH
    export FABRIC_CFG_PATH=$PWD/../config/
    peer lifecycle chaincode package basic.tar.gz --path ../asset-transfer-basic/chaincode-go/ --lang golang --label basic_1.0
    ```

4. Install the chaincode package

    ```sh
    ## peer0.org1.example.com
    export CORE_PEER_TLS_ENABLED=true
    export CORE_PEER_LOCALMSPID="Org1MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    export CORE_PEER_ADDRESS=localhost:7051
    
    peer lifecycle chaincode install basic.tar.gz
    ```

    ```sh
    ## peer0.org2.example.com
    export CORE_PEER_TLS_ENABLED=true
    export CORE_PEER_LOCALMSPID="Org2MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    export CORE_PEER_ADDRESS=localhost:9051
    
    peer lifecycle chaincode install basic.tar.gz
    ```


5. Approve a chaincode definition

    Packeage ID 用於將 peer 安裝的 chaincode 與同意的 chaincode definition 相關聯，並允許組織使用 chaincode 對交易進行背書。

    ```sh
    ## Package ID
    peer lifecycle chaincode queryinstalled
    ```

    chaincode 是在組織級別上得到同意的，因此該命令僅需要針對一個peer，然後會使用 gossip 分發到組織內的其他peer上。 使用`peer lifecycle chaincode approveformyorg`同意 chaincode definition

    ```sh
    ## 注意！每個人拿到的ID都不一樣
    export CC_PACKAGE_ID=basic_1.0:4ec191e793b27e953ff2ede5a8bcc63152cecb1e4c3f301a26e22692c61967ad
    
    ## Org1 
    export CORE_PEER_TLS_ENABLED=true
    export CORE_PEER_LOCALMSPID="Org1MSP"
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    export CORE_PEER_ADDRESS=localhost:7051
    
    peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    
    ## Org2
    export CORE_PEER_LOCALMSPID="Org2MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    export CORE_PEER_ADDRESS=localhost:9051
    
    peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    ```

    我們可以向 `approveformyorg`命令提供`--signature-policy`或`--channel-config-policy`參數，以指定 chaincode 背書策略。默認為多數決，[Endorsement Policies](https://hyperledger-fabric.readthedocs.io/en/release-2.2/endorsement-policies.html)。
     	
    需要具管理員身份的角色同意 chaincode difinination。因此，`CORE_PEER_MSPCONFIGPATH`變數需要指向包含管理員身份的MSP資料夾，不能使用客戶端使用者同意 chaincode difinination。同意需要提交給 ordering service，該服務將驗證管理員簽名，然後將同意分發給peer。


6. Committing the chaincode definition to the channel

    在足夠多的組織同意了 chaincode difinination 之後，其中一個組織可以將 chaincode difinination 提交給 channel，交易將會成功提交，並且 chaincode difinination中同意的參數將在該channel上實現。

    ```sh
    ## 查看同意情形(check commit readiness)
    peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --output json
    
    ## Org1
    export CORE_PEER_TLS_ENABLED=true
    export CORE_PEER_LOCALMSPID="Org1MSP"
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    export CORE_PEER_ADDRESS=localhost:7051
        
    ## Commit
    peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 1.0 --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    ```

    上面的交易使用`peerAddresses`針對Org1的peer0.org1.example.com、Org2的peer0.org2.example.com。提交的交易被提交到連接到channel的peer。該命令需要以足夠多的組織為目標，以滿足部署 chaincode 的策略。由於同意分佈在每個組織中，所以可以針對屬於channel成員的任何peer。

    Channel member 對 chaincode difinination 的背書將提交給ordering service，以添加到區塊中並分發給channel。 然後，channel上的peers將驗證是否有足夠的組織同意了chaincode difinination。`peer lifecycle chaincode commit`命令將等待peer的驗證才返回回應。

    ```sh
    ## 查訊 commit 情形
    peer lifecycle chaincode querycommitted --channelID mychannel --name basic --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    ```

7. Invoking the chaincode

    在chaincode difinination被提交到channel後，chaincode將在加入channel有安裝chaincode的peers上啓動。

    使用 `peer chaincode invoke`呼叫chaincode以及`peer chaincode query`查詢結果

    ```sh
    peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'
    
    ## 查詢
    peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
    ```

8. Upgrading a smart contract

    channel成員可以通過安裝新的chaincode package，然後用新的package ID、新的chaincode版本以及遞增序列號來升級chaincode。新的chaincode可以在chaincode difinination提交給channel後使用。這個過程允許channel成員協調chaincode升級的時間，並確保在新的chaincode被部署到channel之前，有足夠數量的chaincode成員準備好使用新的chaincode。通過同意帶有新的背書策略的chaincode difinination，並將chaincode difinination提交給channel，chnnel成員可以更改管理chaincode的背書策略，而無需安裝新的chaincode package。

    以下將以JS語言代替GO語言寫成的package當作**升級範例**：

    ```sh
    ## npm install
    cd ../asset-transfer-basic/chaincode-javascript
    npm install
    cd ../../test-network
    
    ## package
    export PATH=${PWD}/../bin:$PATH
    export FABRIC_CFG_PATH=$PWD/../config/
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    peer lifecycle chaincode package basic_2.tar.gz --path ../asset-transfer-basic/chaincode-javascript/ --lang node --label basic_2.0
    
    ## Org1
    export CORE_PEER_TLS_ENABLED=true
    export CORE_PEER_LOCALMSPID="Org1MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    export CORE_PEER_ADDRESS=localhost:7051
    
    ## install package
    peer lifecycle chaincode install basic_2.tar.gz
    
    ## query installed(注意！package id 會不同)
    peer lifecycle chaincode queryinstalled
    
    ## approve a new chaincode definition
    export NEW_CC_PACKAGE_ID=basic_2.0:4603980dac5d585e20e181b3ddc7c1d063dc62c3b39bb4e2b271217cde2d9e4d
    peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 2.0 --package-id $NEW_CC_PACKAGE_ID --sequence 2 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    
    ## Org2
    export CORE_PEER_LOCALMSPID="Org2MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    export CORE_PEER_ADDRESS=localhost:9051
    
    ## install package
    peer lifecycle chaincode install basic_2.tar.gz
    
    ## approve a new chaincode definition
    peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 2.0 --package-id $NEW_CC_PACKAGE_ID --sequence 2 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    
    ## check commited readiness
    peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 2.0 --sequence 2 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --output json
    
    ## commit
    peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name basic --version 2.0 --sequence 2 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    
    ## verify result
    docker ps
    
    ## invoke
    peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"CreateAsset","Args":["asset8","blue","16","Kelley","750"]}'
    
    ## query
    peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
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

1.  全部流程

    ```sh
    ## 啟動網路
    cd fabric-samples/test-network
    ./network.sh down
    ./network.sh up createChannel -c mychannel -ca
    ./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-javascript/ -ccl javascript
    ```

    ```sh
    ## 範例應用
    cd ../asset-transfer-basic/application-javascript
    rm -rf wallet/
    npm install
    node app.js
    cd ../../test-network/
    ./network.sh down
    ```

2.  Start test-network -> deploy basic

3.  The application enrolls the admin user

    建立了一個管理員用戶(admin)作為憑證頒發機構(CA)的註冊商。

    我們的第一步是通過應用程式呼叫enrollAdmin來生成用於管理的公私鑰和X.509證書。 此過程使用憑證簽名請求(CSR): 首先在本地生成公私鑰，然後將公鑰發送到CA，CA傳回編碼的憑證以供應用程式使用，然後這些憑證會儲存在錢包中，使我們能夠充當CA的管理員。

    需要注意的是，註冊管理員和註冊應用程式使用者是發生在應用程式和憑證頒發機構之間的互動，而不是應用程式和chaincode之間的互動。

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

4.  The application registers and enrolls an application user

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

5.  The sample application prepares a connection to the channel and smart contract

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

6.  The application initializes the ledger with some sample data

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

    

7.  The application invokes each of the chaincode functions

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
    >Evaluate Transaction: GetAllAssets, function returns all the current assets on the ledger
    >Result: [...]
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
    >"ID": "asset13",
    >"Color": "yellow",
    >"Size": "5",
    >"Owner": "Tom",
    >"AppraisedValue": "1300"
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
    >"ID": "asset1",
    >"Color": "blue",
    >"Size": "5",
    >"Owner": "Tomoko",
    >"AppraisedValue": "350"
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
    >"ID": "asset1",
    >"Color": "blue",
    >"Size": "5",
    >"Owner": "Tom",
    >"AppraisedValue": "350"
    >}
    >```

8.  clean up

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
    ## organization/magnetocorp/application
    cd application
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
    rm ../identity/user/isabella/wallet/*
    node enrollUser.js
    cat ../identity/user/isabella/wallet/*
    node issue.js
    ## 應用程式呼叫 papercontract.js 中的 CommercialPaper 智慧合約中定義的 issue交易。智慧合約通過 Fabric API 與帳本進行互動，最主要的是 putState() 和 getState()，將新的商業票據表示為世界狀態中的一個向量狀態。
    ```

7.  Digibank application - buy

    ```sh
    ## Digibank(Balaji)
    ## organization/digibank/application/
    cd application
    npm install
    ```

    `buy.js`:

    -   整體來說和 `issue.js`很像
    -   主要差別在`const buyResponse = await contract.submitTransaction('buy', 'MagnetoCorp', '00001', ...);`

    在 Balaji 的 wallet 中產生 X.509 certificate，然後執行`buy.js`，

    ```sh
    rm ../identity/user/balaji/wallet/*
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
    
    ## CreateAsset
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
     -   註2：peerAddresses 參數為指定 endorsing peers

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

## Using CouchDB(Lab 7)

本教學將介紹使用CouchDB作為Hyperledger Fabric的狀態資料庫。將使用資產轉移帳本查詢案例來演示如何使用CouchDB與Fabric，包括對狀態資料庫執行JSON查詢。

Fabric支持兩種類型的peer狀態資料庫

-   LevelDB是嵌入在peer中的默認狀態資料庫。LevelDB將chaincode資料儲存為key-value，只支持key、range key和composite key查詢。

-   CouchDB是另一個可選的選擇，它允許將帳本上的資料建模為JSON，並針對資料value(而不是key)執行查詢。CouchDB還支援使用chaincode部署index，提高查詢效率。

    -   使用：編輯core.yaml的stateDatabase部分。指定CouchDB為stateDatabase，並填寫相關的couchDBConfig屬性。[CouchDB configuration](https://hyperledger-fabric.readthedocs.io/en/latest/couchdb_as_state_database.html#couchdb-configuration)

    -   index: 索引允許查詢資料庫時不必查詢每一行，從而使它們運行得更快、更有效。要定義索引，需要三項資訊，並放在`META-INF/statedb/couchdb/indexes`

        -   欄位：這些是要查詢的欄位
        -   名稱：索引名稱
        -   類型：在此情境永遠為"json"

    -   在每個chaincode中只安裝幾個支持大部分查詢的索引。添加過多的索引，或者在索引中使用過多的字欄位，會降低網路的性能。這是因為索引是在每個區塊提交後更新的(index warming)。

    -   ```go
        type Asset struct {
                DocType        string `json:"docType"` //docType is used to distinguish the various types of objects in state database
                ID             string `json:"ID"`      //the field tags are needed to keep case from bouncing around
                Color          string `json:"color"`
                Size           int    `json:"size"`
                Owner          string `json:"owner"`
                AppraisedValue int    `json:"appraisedValue"`
        }
        ```

        ```go
        {
          "index":{
              "fields":["owner"] // Names of the fields to be queried
          },
          "ddoc":"index1Doc", // (optional) Name of the design document in which the index will be created.
          "name":"index1",
          "type":"json"
        }
        
        {
          "index":{
              "fields":["owner", "color"] // Names of the fields to be queried
          },
          "ddoc":"index2Doc", // (optional) Name of the design document in which the index will be created.
          "name":"index2",
          "type":"json"
        }
        
        {
          "index":{
              "fields":["owner", "color", "size"] // Names of the fields to be queried
          },
          "ddoc":"index3Doc", // (optional) Name of the design document in which the index will be created.
          "name":"index3",
          "type":"json"
        }
        ```

1.  Start the network & Deploy the smart contract

    ```sh
    cd ../asset-transfer-ledger-queries/chaincode-go
    GO111MODULE=on go mod vendor
    cd ../../test-network
    ./network.sh up createChannel -s couchdb
    ./network.sh deployCC -ccn ledger -ccp ../asset-transfer-ledger-queries/chaincode-go/ -ccl go -ccep "OR('Org1MSP.peer','Org2MSP.peer')"
    ```

    verify index was deployed

    ```sh
    docker logs peer0.org1.example.com  2>&1 | grep "CouchDB index"
    ```

2.  Query the CouchDB State Database

    -    QueryAssest: 即席 JSON 查詢範例，將JSON 查詢字串傳遞到函數，這對運行時客戶端動態建立查詢有幫助
        -   `{"selector":{"docType":"asset","owner":"tom"}`: 
        -   `"use_index":["_design/indexOwnerDoc", "indexOwner"]`: ddoc / index name
    -   QueryAssetsByOwner: 參數化查詢範例，在chaincode中定義查詢傳遞一個查詢參數。在這種情況下，即資產擁有者，然後，它使用JSON查詢語法在狀態資料庫中查詢符合擁有者id以及docType為 "asset "的 JSON 文件。

    ```sh
    ## export bin & config
    export PATH=${PWD}/../bin:${PWD}:$PATH
    export FABRIC_CFG_PATH=$PWD/../config/
    
    ## Org1
    export CORE_PEER_TLS_ENABLED=true
    export CORE_PEER_LOCALMSPID="Org1MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    export CORE_PEER_ADDRESS=localhost:7051
    
    ## CreateAsset
    peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n ledger -c '{"Args":["CreateAsset","asset1","blue","5","tom","35"]}'
    
    ## Rich Query with index name explicitly specified:
    peer chaincode query -C mychannel -n ledger -c '{"Args":["QueryAssets", "{\"selector\":{\"docType\":\"asset\",\"owner\":\"tom\"}, \"use_index\":[\"_design/indexOwnerDoc\", \"indexOwner\"]}"]}'
    ```

    以下是 index 和幾個查詢範例

    -   索引中的欄位也必須在你的查詢的選擇器或排序片段，才能使用索引。
    -   更複雜的查詢將具有較低的性能，並且不太可能使用索引。
    -   避免使用會導致全表掃描或全索引掃描的操作行為，如`$or`、`$in`和`$regex`。
    -   Example
        1.  這個查詢完全支持索引
        2.  這個查詢仍然使用索引，但會額外掃描其他欄位
        3.  這個查詢無法使用索引，將不得不掃描整個數據庫
        4.  這個查詢仍然使用索引，但查詢中的`$or`條件需要掃描索引中的所有項目
        5.  這個查詢不會使用索引，因為它需要搜索整個表來滿足`$or`條件

    ```json
    {
      "index": {
        "fields": [
          "docType",
          "owner"
        ]
      },
      "ddoc": "indexOwnerDoc",
      "name": "indexOwner",
      "type": "json"
    }
    ```

    ```sh
    export CHANNEL_NAME=mychannel
    
    ## Example one: query fully supported by the index
    peer chaincode query -C $CHANNEL_NAME -n ledger -c '{"Args":["QueryAssets", "{\"selector\":{\"docType\":\"asset\",\"owner\":\"tom\"}, \"use_index\":[\"indexOwnerDoc\", \"indexOwner\"]}"]}'
    
    ## Example two: query fully supported by the index with additional data
    peer chaincode query -C $CHANNEL_NAME -n ledger -c '{"Args":["QueryAssets", "{\"selector\":{\"docType\":\"asset\",\"owner\":\"tom\",\"color\":\"blue\"}, \"use_index\":[\"/indexOwnerDoc\", \"indexOwner\"]}"]}'
    
    ## Example three: query not supported by the index
    peer chaincode query -C $CHANNEL_NAME -n ledger -c '{"Args":["QueryAssets", "{\"selector\":{\"owner\":\"tom\"}, \"use_index\":[\"indexOwnerDoc\", \"indexOwner\"]}"]}'
    
    ## Example four: query with $or supported by the index
    peer chaincode query -C $CHANNEL_NAME -n ledger -c '{"Args":["QueryAssets", "{\"selector\":{\"$or\":[{\"docType\":\"asset\"},{\"owner\":\"tom\"}]}, \"use_index\":[\"indexOwnerDoc\", \"indexOwner\"]}"]}'
    
    ## Example five: Query with $or not supported by the index
    peer chaincode query -C $CHANNEL_NAME -n ledger -c '{"Args":["QueryAssets", "{\"selector\":{\"$or\":[{\"docType\":\"asset\",\"owner\":\"tom\"},{\"color\":\"yellow\"}]}, \"use_index\":[\"indexOwnerDoc\", \"indexOwner\"]}"]}'
    
    ```

    注意使用索引並不是收集大量數據的解決方案。區塊鏈資料結構經過優化，用於驗證和確認交易，不適合資料分析或報告。最好的做法是查詢一個複製peer的鏈外資料庫。可以使用來自應用程式的區塊或鏈碼事件將交易資料寫入鏈外資料庫或分析引擎。[off_chain_data](https://github.com/hyperledger/fabric-samples/tree/master/off_chain_data)

3.  Query the CouchDB State Database With Pagination

    -   [topic on pagination with CouchDB](https://hyperledger-fabric.readthedocs.io/en/latest/couchdb_as_state_database.html#couchdb-pagination)
    -   QueryAssetsWithPagination: 分頁即席 JSON 查詢範例
        -   pagesize: 一頁幾筆
        -   bookmark: 書籤，告訴couchDB從哪筆開始
    -   getQueryResultForQueryStringWithPagination: [Iterate code](https://github.com/hyperledger/fabric-samples/blob/master/asset-transfer-ledger-queries/chaincode-go/asset_transfer_ledger_chaincode.go)

    ```sh
    ## 增加幾筆資料
    peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile  ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n ledger -c '{"Args":["CreateAsset","asset2","yellow","5","tom","35"]}'
    peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile  ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n ledger -c '{"Args":["CreateAsset","asset3","green","6","tom","20"]}'
    peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile  ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n ledger -c '{"Args":["CreateAsset","asset4","purple","7","tom","20"]}'
    peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile  ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n ledger -c '{"Args":["CreateAsset","asset5","blue","8","tom","40"]}'
    ```

    ```sh
    ## pagesize = 3, bookmark = ""
    peer chaincode query -C mychannel -n ledger -c '{"Args":["QueryAssetsWithPagination", "{\"selector\":{\"docType\":\"asset\",\"owner\":\"tom\"}, \"use_index\":[\"_design/indexOwnerDoc\", \"indexOwner\"]}","3",""]}'
    
    ## pagesize = 3, bookmark = 第三筆
    peer chaincode query -C mychannel -n ledger -c '{"Args":["QueryAssetsWithPagination", "{\"selector\":{\"docType\":\"asset\",\"owner\":\"tom\"}, \"use_index\":[\"_design/indexOwnerDoc\", \"indexOwner\"]}","3","g1AAAABJeJzLYWBgYMpgSmHgKy5JLCrJTq2MT8lPzkzJBYqzJRYXp5YYg2Q5YLI5IPUgSVawJIjFXJKfm5UFANozE8s"]}'
    ```

4.  Update/Delete an Index(Fauxton interface)

    要更新索引，請使用相同的索引名稱，但更改索引欄位(另外Fabric只支持索引類型為JSON)，當chaincode definition提交到channel時，更新後的索引被重新部署到peer的狀資料庫中。索引名稱或ddoc屬性的更改將導致建立新的索引，並且在CouchDB中原始索引保持不變，直到它被刪除。

    -   更新chaincode：參考 Lab2 & [Changing Endorsement Policy and Chaincode Upgrade After Chaincode Committed (Hyperledger Fabric 2.0)](https://kctheservant.medium.com/changing-endorsement-policy-and-chaincode-upgrade-after-chaincode-committed-hyperledger-fabric-cfa87fec4594)

    -   Fauxton 介面是一個用於建立、更新和部署索引到CouchDB的Web UI。 http://localhost:5984/_utils/
        -   COUCHDB_USER=admin
        -   COUCHDB_PASSWORD=adminpw

    -   CLI CURL

    ```sh
    ## Index for docType, owner.
    ##Example curl command line to define index in the CouchDB channel_chaincode database
    curl -i -X POST -H "Content-Type: application/json" -u admin:adminpw -d "{\"index\":{\"fields\":[\"docType\",\"owner\"]},\"name\":\"indexOwner\",\"ddoc\":\"indexOwnerDoc\",\"type\":\"json\"}" http://localhost:5984/mychannel_ledger/_index
     
    curl -X DELETE -H "accept: */*" -u admin:adminpw http://localhost:5984/mychannel_ledger/_index/indexOwnerDoc/json/indexOwner
    ```

5.  Cleanup

    ```sh
    ./network.sh down
    ```

## Creating a channel(Lab 8)

為了在 Hyperledger Fabric 網路上建立和轉移資產，一個組織需要加入一個 channel。channel是特定組織之間的私人通信層，網路的其他成員是看不到的。每個channel由一個獨立的 ledger 組成，只能由 channel 成員進行讀寫，channel 成員允許將其 peer 加入到 channel 中，並從 ordering service 中接收新的交易區塊。peer、node和 CA構成了網路的基礎設施，而 channel 則是各組織相互連接和互動的過程。

-   Create a new channel

    channel 是通過建立一個 channel creation TX 並將該交易提交給 ordering service 來建立的。 channel creation TX 指定了channel 的初始配置，並被 ordering service 用來寫成 channel genesis block。雖然可以手動建立channel creation TX，但使用 configtxgen 工具更方便。該工具的工作原理是讀取一個定義了你的channel配置(configtx.yaml)，然後將相關資訊寫入channel creation TX。

1.  Setting up the configtxgen tool

    ```sh
    cd fabric-samples/test-network
    export PATH=${PWD}/../bin:$PATH
    export FABRIC_CFG_PATH=${PWD}/configtx
    configtxgen --help
    ```

2.  The configtx.yaml file(`configtx/configtx.yaml`)

    -   Organizations: 通道配置中包含的最重要資訊是通道成員的組織。每個組織由一個 MSP ID 和一個 channel MSP 識別。channel MSP 會儲存在通道配置中，包含用於識別組織的節點、應用程式和管理員的憑證。configtx.yaml 文件的 Organizations 部分用於為通道的每個成員建立 channel MSP 和相應的 MSP ID。測試網路使用的 configtx.yaml 文件包含三個組織。兩個組織是對等節點組織，Org1和Org2，可以添加到應用通道中。一個組織 OrdererOrg 是排序服務的管理員。因為使用不同的證書頒發機構來部署對等節點和排序節點是一種最佳的做法，所以組織通常被稱為對等組織或排序組織，即使它們實際上是由同一家公司管理的。
    -   Capabilities: Fabric 通道可以由運行不同版本 Hyperledger Fabric 的排序節點和對等節點加入。通道功能允許運行不同 Fabric 二進制的組織通過僅啓用某些功能來參與同一通道。例如，運行Fabric v1.4的組織和運行Fabric v2.x的組織可以加入同一個通道，只要通道能力級別設置為V1_4_X或以下。通道成員都不能使用Fabric v2.0中引入的功能。
        -   Application: 管理對等節點使用的功能，如Fabric鏈碼生命週期，並設置加入通道的對等節點可以運行的Fabric二進制文件的最低版本。
        -   Orderer: 管理排序節點所使用的功能，如Raft共識，並設置屬於通道consenter set的排序節點可以運行的Fabric二進制文件的最小版本。
        -   Channel: 設置了對等節點和排序節點可以運行的Fabric的最小版本。
    -   Application: 定義了管理對等組織如何與應用通道交互的策略。這些策略規定了需要批准鏈碼定義或簽署更新通道配置請求的對等組織的數量。這些策略還用於限制對通道資源的存取，例如向通道帳本寫入或查詢通道事件的能力。
    -   Orderer: 定義哪些排序節點將形成網路的排序服務(consenter set)，以及他們將使用的共識方法來同意交易的共同順序，以及將成為排序服務管理員的組織。
    -   Channel: 定義了如何與channel交互以及哪些組織需要批准channel更新的策略。為通道配置最高級別的策略。對於應用通道，這些策略管理雜湊演算法、用於產生新區塊的資料雜湊結構以及通道能力級別。在系統通道中，這些策略還管理對等組織聯盟的建立或刪除。
    -   Profiles: 引用文件其他部分的資訊來建立一個通道配置。這些配置文件用於
        -   建立orderer system channel(系統通道)的gensis block (`TwoOrgsOrdererGenesis`)，系統通道定義了排序服務的節點和作為排序服務管理員的一組組織。系統通道還包括一些屬於區塊鏈聯盟的對等組織。聯盟每個成員的通道MSP都包含在系統通道中，允許他們建立新的應用通道，並將聯盟成員添加到新通道中。
        -   建立peer 使用的 application channel(應用通道)(``TwoOrgsChannel`)，系統通道被排序服務作為建立應用通道的模板。在系統通道中定義的排序服務節點成為新通道的默認consenter set，而排序服務的管理員則成為通道的排序員管理員。通道成員的通道MSP從系統通道轉移到新通道中。通道建立後，可以通過更新通道配置從通道中添加或刪除排序節點。

3.  Channel policies(`configtx/configtx.yaml`)

    通道是組織間交流的一種私密方式。因此，對通道配置的大多數更改都需要得到通道的其他成員的同意。Channel policies是由 `configtx.yaml `的不同部分决定的([了解policy](https://hyperledger-fabric.readthedocs.io/en/latest/policies.html)、[Channel policies](https://hyperledger-fabric.readthedocs.io/en/latest/create_channel/channel_policies.html))

    -   Signature policies(Organizations/Policies): 當一個提案提交給對等節點，或者一個交易提交給排序節點時，節點會讀取附加到交易的簽名，並根據通道配置中定義的簽名策略來評估它們。
    -   ImplicitMeta Policies:(Application/Policies & Orderer/Policies & Channel/Policies) 隱式策略，每個組織的簽名策略將由ImplicitMeta策略在較高級別的通道配置中評估。e.g.可以在通道級別設置隱式規則`MAJORITY Admins`，同時允許每通道道成員選擇對其組織進行簽名所需的身份`Admins`。優點：在從通道中添加或刪除組織時不需要更新它們；缺點：它們不能顯式讀取通道成員使用的簽名策略。
    -   Channel modification policies
    -   Channel policies and Access Control Lists

4.  Start the network

    ```sh
    ./network.sh down
    ./network.sh up
    ```

4.  Creating an application channel

    -   `-profile` : 使用在`configtx.yaml`定義的TwoOrgsChannel(ref.SampleConsortium)
    -   `-channelID`: channel 名稱

    ```sh
    configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel1.tx -channelID channel1
    export FABRIC_CFG_PATH=$PWD/../config/
    
    ## Org1
    export CORE_PEER_TLS_ENABLED=true
    export CORE_PEER_LOCALMSPID="Org1MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    export CORE_PEER_ADDRESS=localhost:7051
    
    ## create channel 產生 gensis block
    peer channel create -o localhost:7050  --ordererTLSHostnameOverride orderer.example.com -c channel1 -f ./channel-artifacts/channel1.tx --outputBlock ./channel-artifacts/channel1.block --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    ```

5.  Join peers to the channel

    -   Org1 使用 genesis block 加入 channel

    ```sh
    ## 加入
    peer channel join -b ./channel-artifacts/channel1.block
    
    ## 確認
    peer channel getinfo -c channel1
    ```

    -   Org2 使用 fetch 從 oredering service 抓 gensis block

    ```sh
    ## Org2
    export CORE_PEER_TLS_ENABLED=true
    export CORE_PEER_LOCALMSPID="Org2MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    export CORE_PEER_ADDRESS=localhost:9051
    
    ## fetch gensis block
    peer channel fetch 0 ./channel-artifacts/channel_org2.block -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com -c channel1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    
    ## 加入
    peer channel join -b ./channel-artifacts/channel_org2.block
    
    ## 確認
    peer channel getinfo -c channel1
    ```

6.  Set anchor peers

    當一個組織將其 peer 加入 channel 後，他們應該至少選擇一個peer成為 anchor peer，為了private data 和 service discovery...等功能。每個組織應該在一個channel上設置多個anchor peer 實現冗餘。ref.[Gossip data dissemination protocol](https://hyperledger-fabric.readthedocs.io/en/release-2.2/gossip.html)

    通道配置中包含了每個組織的 anchor peer 的終端資訊。每個channel成員可以通過更新channel來指定自己的anchor peer。接下來將使用 configtxlator 工具更新通道配置，為Org1和Org2選擇一個anchor peer。

    !! 記得下載 jq 

    ```sh
    ## Org1
    export FABRIC_CFG_PATH=$PWD/../config/
    export CORE_PEER_TLS_ENABLED=true
    export CORE_PEER_LOCALMSPID="Org1MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    export CORE_PEER_ADDRESS=localhost:7051
    
    ## fetch most recent configuration block
    peer channel fetch config channel-artifacts/config_block.pb -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com -c channel1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    
    cd channel-artifacts
    
    ## protobuf 轉為 json
    configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
    jq '.data.data[0].payload.data.config' config_block.json > config.json
    
    ## 新增 anchor peer 到 Org1
    cp config.json config_copy.json
    jq '.channel_group.groups.Application.groups.Org1MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org1.example.com","port": 7051}]},"version": "0"}}' config_copy.json > modified_config.json
    
    ## original config json -> protobuf
    configtxlator proto_encode --input config.json --type common.Config --output config.pb
    ## modified config json -> protobuf
    configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb
    ## compute update
    configtxlator compute_update --channel_id channel1 --original config.pb --updated modified_config.pb --output config_update.pb
    configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
    
    ## 包成 transaction envelope
    echo '{"payload":{"header":{"channel_header":{"channel_id":"channel1", "type":2}},"data":{"config_update":'$(cat config_update.json)'}}}' | jq . > config_update_in_envelope.json
    configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output config_update_in_envelope.pb
    cd ..
    
    ## update channel
    peer channel update -f channel-artifacts/config_update_in_envelope.pb -c channel1 -o localhost:7050  --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    ```

    ```sh
    ## Org2
    export CORE_PEER_TLS_ENABLED=true
    export CORE_PEER_LOCALMSPID="Org2MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    export CORE_PEER_ADDRESS=localhost:9051
    
    ## fetch most recent configuration block
    peer channel fetch config channel-artifacts/config_block.pb -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com -c channel1 --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    
    cd channel-artifacts
    
    ## protobuf 轉為 json
    configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
    jq '.data.data[0].payload.data.config' config_block.json > config.json
    cp config.json config_copy.json
    
    ## 新增 anchor peer 到 Org2
    jq '.channel_group.groups.Application.groups.Org2MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org2.example.com","port": 9051}]},"version": "0"}}' config_copy.json > modified_config.json
    
    ## original config json -> protobuf
    configtxlator proto_encode --input config.json --type common.Config --output config.pb
    ## modified config json -> protobuf
    configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb
    ## compute update
    configtxlator compute_update --channel_id channel1 --original config.pb --updated modified_config.pb --output config_update.pb
    configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
    
    ## 包成 transaction envelope
    echo '{"payload":{"header":{"channel_header":{"channel_id":"channel1", "type":2}},"data":{"config_update":'$(cat config_update.json)'}}}' | jq . > config_update_in_envelope.json
    configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output config_update_in_envelope.pb
    cd ..
    
    ## update channel
    peer channel update -f channel-artifacts/config_update_in_envelope.pb -c channel1 -o localhost:7050  --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    
     
    ## 確認結果(block height 應該要=3)
    peer channel getinfo -c channel1
    ```

7.  Deploy a chaincode to the new channel

    ```sh
    ## deploy basic chaincode on channel 1
    ./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go/ -ccl go -c channel1 -cci InitLedger
    
    # query
    peer chaincode query -C channel1 -n basic -c '{"Args":["getAllAssets"]}'
    ```

8.  Clean up

    ```sh
    ./network.sh down
    ```

## Adding an Org to a Channel(Lab 9)

1.  Setup the Environment

    ```sh
    cd fabric-samples/test-network
    ./network.sh down
    ./network.sh up createChannel
    
    ## Script(一步完成)
    ## cd addOrg3
    ## ./addOrg3.sh up
    ## ./addOrg3.sh down
    ```

2.  Generate the Org3 Crypto Material

    ```SH
    cd addOrg3
    ../../bin/cryptogen generate --config=org3-crypto.yaml --output="../organizations"
    
    ## org3.json
    export FABRIC_CFG_PATH=$PWD
    ../../bin/configtxgen -printOrg Org3MSP > ../organizations/peerOrganizations/org3.example.com/org3.json
    ```

    組織定義中包含了：Org3的策略定義、Org3的NodeOU定義，以及一個CA根憑證，用於建立組織的信任根、一個TLS根憑證，由gossip協定識別Org3，用於區塊傳播和服務發現。

3.  Bring up Org3 components

    ```sh
    ## 這個 docker-compose 已設定好 bridge 到 initial network，並啟動peer0.org3和一個CLI(fabric-tools)
    docker-compose -f docker/docker-compose-org3.yaml up -d
    ```

4.  Prepare the CLI Environment

    ```sh
    ## 這個 Org3cli container 有 mount organizations資料夾，所以可以利用環境變數改變角色(admin, Org1, Org2, Org3)
    docker exec -it Org3cli bash
    ```

    ```sh
    export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
    export CHANNEL_NAME=mychannel
    ```

5.  Fetch the Configuration

    ```sh
    ## Org1
    export CORE_PEER_LOCALMSPID="Org1MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    
    ## fetch -> config_block.pb
    peer channel fetch config config_block.pb -o orderer.example.com:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA
    ```

6.  Convert the Configuration to JSON and Trim It Down

    ```sh
    configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
    jq '.data.data[0].payload.data.config' config_block.json > config.json
    ```

7.  Add the Org3 Crypto Material

    ```sh
    ## merge 2 file(ref.https://stackoverflow.com/questions/19529688/how-to-merge-2-json-objects-from-2-files-using-jq)
    jq -s '.[0] * {"channel_group":{"groups":{"Application":{"groups": {"Org3MSP":.[1]}}}}}' config.json ./organizations/peerOrganizations/org3.example.com/org3.json > modified_config.json
    
    ## config json -> protobuf
    configtxlator proto_encode --input config.json --type common.Config --output config.pb
    configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb
    
    ## compute update
    configtxlator compute_update --channel_id $CHANNEL_NAME --original  config.pb --updated modified_config.pb --output org3_update.pb
    configtxlator proto_decode --input org3_update.pb --type common.ConfigUpdate | jq . > org3_update.json
    
    ## 包成 transaction envelope
    echo '{"payload":{"header":{"channel_header":{"channel_id":"'$CHANNEL_NAME'", "type":2}},"data":{"config_update":'$(cat org3_update.json)'}}}' | jq . > org3_update_in_envelope.json
    configtxlator proto_encode --input org3_update_in_envelope.json --type common.Envelope --output org3_update_in_envelope.pb
    ```

8.  Sign and Submit the Config Update

    ```sh
    ## Org1
    peer channel signconfigtx -f org3_update_in_envelope.pb
    ```

    ```sh
    ## Org2
    export CORE_PEER_LOCALMSPID="Org2MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    export CORE_PEER_ADDRESS=peer0.org2.example.com:9051
    
    ## update(會附帶sign)
    ## 現實中 org3_update_in_envelope.pb 需要安全傳輸
    peer channel update -f org3_update_in_envelope.pb -c $CHANNEL_NAME -o orderer.example.com:7050 --tls --cafile $ORDERER_CA
    ```

9.  Join Org3 to the Channel

    ```sh
    ## Org3
    export CORE_PEER_LOCALMSPID="Org3MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/peerOrganizations/org3.example.com/users/Admin@org3.example.com/msp
    export CORE_PEER_ADDRESS=peer0.org3.example.com:11051
    
    ## 法一：fetch genesis block
    peer channel fetch 0 mychannel.block -o orderer.example.com:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA
    
    ## 法二：joinbysnapshot
    ## peer channel joinbysnapshot --snapshotpath <path to snapshot>
    
    ## join channel
    peer channel join -b mychannel.block
    ```

10.  Configuring Leader Election

     新加入的peer是用genesis block，它不包含通道配置更新資訊。因此，新的peer無法利用gossip，因為他們無法驗證其他peer從自己的組織轉發的區塊，直到他們得到那筆交易(將組織添加到通道配置)。因此，新添加的peer必須具有以下配置(2.2以後預設)，以便它們從排序服務中接收區塊。

     ```sh
     CORE_PEER_GOSSIP_USELEADERELECTION=false
     CORE_PEER_GOSSIP_ORGLEADER=true
     ```

     若想要改為[Dynamic leader election](https://hyperledger-fabric.readthedocs.io/en/latest/gossip.html#dynamic-leader-election)

     ```sh
     CORE_PEER_GOSSIP_USELEADERELECTION=true
     CORE_PEER_GOSSIP_ORGLEADER=false
     ```

11.  Install, define, and invoke chaincode

     通過一系列操作來確認 org3 是 mychannel 的成員之一

     ```sh
     ## new terminal
     cd fabric-samples/test-network
     ./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go/ -ccl go
     
     export PATH=${PWD}/../bin:$PATH
     export FABRIC_CFG_PATH=$PWD/../config/
     export CORE_PEER_TLS_ENABLED=true
     export CORE_PEER_LOCALMSPID="Org3MSP"
     export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt
     export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org3.example.com/users/Admin@org3.example.com/msp
     export CORE_PEER_ADDRESS=localhost:11051
     
     ## Approvals: [Org1MSP: true, Org2MSP: true, Org3MSP: false]
     peer lifecycle chaincode querycommitted --channelID mychannel --name basic
     
     
     ## package -> install -> approveformyorg
     peer lifecycle chaincode package basic.tar.gz --path ../asset-transfer-basic/chaincode-go/ --lang golang --label basic_1
     peer lifecycle chaincode install basic.tar.gz
     peer lifecycle chaincode queryinstalled
     export CC_PACKAGE_ID=basic_1:9820659c595e662a849033ca23b4424e87a126e8f40b5f81ace59820b81fe8e7
     peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --channelID mychannel --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1
     
     ## 看approve結果
     peer lifecycle chaincode querycommitted --channelID mychannel --name basic
     
     ## invoke
     peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt --peerAddresses localhost:11051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'
     
     ## query
     peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
     ```

12.  Updating the Channel Config to include an Org3 Anchor Peer

     類似 lab8-7 步驟，新增加的組織Org3也應該在通道配置中定義他們的anchor peers，這樣任何來自其他組織的peer都可以直接發現Org3的peer([ref.gossip](https://hyperledger-fabric.readthedocs.io/en/latest/gossip.html))

     ```sh
     docker exec -it Org3cli bash
     
     export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
     export CHANNEL_NAME=mychannel
     
     ## fetch recent config_block -> 轉換 -> add anchorpeer to config_block
     peer channel fetch config config_block.pb -o orderer.example.com:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA
     
     configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
     
     jq .data.data[0].payload.data.config config_block.json > config.json
     jq '.channel_group.groups.Application.groups.Org3MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org3.example.com","port": 11051}]},"version": "0"}}' config.json > modified_anchor_config.json
     
     ## 新舊config_block轉換成pb -> compute update -> 包成 transaction envelope
     configtxlator proto_encode --input config.json --type common.Config --output config.pb
     configtxlator proto_encode --input modified_anchor_config.json --type common.Config --output modified_anchor_config.pb
     
     configtxlator compute_update --channel_id $CHANNEL_NAME --original config.pb --updated modified_anchor_config.pb --output anchor_update.pb
     configtxlator proto_decode --input anchor_update.pb --type common.ConfigUpdate | jq . > anchor_update.json
     
     echo '{"payload":{"header":{"channel_header":{"channel_id":"'$CHANNEL_NAME'", "type":2}},"data":{"config_update":'$(cat anchor_update.json)'}}}' | jq . > anchor_update_in_envelope.json
     configtxlator proto_encode --input anchor_update_in_envelope.json --type common.Envelope --output anchor_update_in_envelope.pb
     
     
     ## Org3
     export CORE_PEER_LOCALMSPID="Org3MSP"
     export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt
     export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/peerOrganizations/org3.example.com/users/Admin@org3.example.com/msp
     export CORE_PEER_ADDRESS=peer0.org3.example.com:11051
     
     ## channel update
     peer channel update -f anchor_update_in_envelope.pb -c $CHANNEL_NAME -o orderer.example.com:7050 --tls --cafile $ORDERER_CA
     ```

     ```sh
     docker logs -f peer0.org1.example.com
     ```

     ```log
     [gossip.gossip] learnAnchorPeers -> INFO 088 Learning about the configured anchor peers of Org2MSP for channel mychannel: [{peer0.org2.example.com 9051}]
     [gossip.gossip] learnAnchorPeers -> INFO 089 Learning about the configured anchor peers of Org3MSP for channel mychannel: [{peer0.org3.example.com 11051}]
     [gossip.gossip] learnAnchorPeers -> INFO 08a Learning about the configured anchor peers of Org1MSP for channel mychannel: [{peer0.org1.example.com 7051}]
     ```

13.  Clean up

     ```sh
     cd addOrg3
     ./addOrg3.sh down
     ```

## Updating a channel configuration

像許多複雜的系統一樣，Hyperledger Fabric 網路由結構和一些相關的流程組成。識別區塊鏈網路結構的資訊和管理結構如何互動的過程都包含在通道配置中。由於配置是包含在區塊中的，更新通道配置的過程被稱為配置更新交易。在正式環境中，這些配置更新交易通常在帶外討論後會由單個通道管理員提出，就像通道的初始配置會由通道的初始成員在帶外決定一樣。

-   結構：包含用戶（如管理員）、組織、對等節點、排序節點、CA、智能合約和應用。

-   流程：這些結構之間的互動方式。其中最重要的是Policy，即規定哪些用戶可以做什麼，在什麼條件下做什麼的規則。

More about these parameters

```json
{
  "channel_group": {
    "groups": {
      "Application": {
        "groups": {
          "Org1MSP": {
          },
          "Org2MSP": {
          }
        },
        "mod_policy": "Admins",
        "policies": {
          "Admins": {
          },
          "Endorsement": {
          },
          "LifecycleEndorsement": {
          },
          "Readers": {
          },
          "Writers": {
          }
        },
        "values": {
          "Capabilities": {
          }
        },
        "version": "1"
      },
      "Orderer": {
        "groups": {
          "OrdererOrg": {
          }
        },
        "mod_policy": "Admins",
        "policies": {
          "Admins": {
          },
          "BlockValidation": {
          },
          "Readers": {
          },
          "Writers": {
          }
        },
        "values": {
          "BatchSize": {
          },
          "BatchTimeout": {
          },
          "Capabilities": {
          },
          "ChannelRestrictions": {
          },
          "ConsensusType": {
          }
        },
        "version": "0"
      }
    },
    "mod_policy": "Admins",
    "policies": {
      "Admins": {
      },
      "Readers": {
      },
      "Writers": {
      }
    },
    "values": {
      "BlockDataHashingStructure": {
      },
      "Capabilities": {
      },
      "Consortium": {
      },
      "HashingAlgorithm": {
      },
      "OrdererAddresses": {
      }
    },
    "version": "0"
  },
  "sequence": "3"
}
```

-   Policies:  策略不僅僅是一個配置值，它們定義了在什麼情況下可以更改所有參數。 [Policies](https://hyperledger-fabric.readthedocs.io/en/latest/policies/policies.html)
-   Capabilities: 確保網路和通道以同樣的方式處理事情，為通道配置更新和鏈碼呼叫等事情建立確定性的結果。 [Capabilities](https://hyperledger-fabric.readthedocs.io/en/latest/capabilities_concept.html)
-   channel/application: 管理應用程式通道特有的配置參數
    -   Add orgs to a channel: 要新增 orgs 要把他加入這裡(Channel/Application/groups)
    -   Organization-related parameters: 任何特定於組織的參數都可以被更改
-   channel/orderer: 管理排序服務或排序者系統通道特有的配置參數
    -   Batch size
    -   Batch timeout
    -   Block validation
    -   Consensus type
    -   Raft ordering service parameters
-   channel: 管控對等組織和排序服務組織都需要同意的配置參數
    -   Orderer addresses
    -   Hashing structure
    -   Hashing algorithm

---

Editing a config(參考lab8, lab9)

1.  Pull and translate the config
2.  Modify the config
3.  Re-encode and submit the config
4.  Get the Necessary Signatures

## Writing Your First Chaincode(Lab 10)

Chaincode是一個程式，用Go、Node.js或Java編寫，實現了一個規定的介面。Chaincode運行在一個獨立於對等節點的進程中，通過應用程式提交的交易來初始化和管理帳本狀態。

請注意，當使用合約api時，每個被調用的鏈碼函數都會被傳遞一個交易上下文 "ctx"，你可以從這個上下文中獲得鏈碼存根(GetStub())，其中有存取分類帳的函數(例如GetState())，並提出更新分類帳的請求(例如PutState())。

1.  Choosing a Location for the Code

    ```sh
    cd fabric-samples
    mkdir atcc && cd atcc
    go mod init atcc
    touch atcc.go
    ```

2.  Housekeeping

    ```go
    package main
    
    import (
      "fmt"
      "log"
      "github.com/hyperledger/fabric-contract-api-go/contractapi"
    )
    
    // SmartContract provides functions for managing an Asset
    type SmartContract struct {
      contractapi.Contract
    }
    ```

    ```go
    // Asset describes basic details of what makes up a simple asset
    type Asset struct {
      ID             string `json:"ID"`
      Color          string `json:"color"`
      Size           int    `json:"size"`
      Owner          string `json:"owner"`
      AppraisedValue int    `json:"appraisedValue"`
    }
    ```

3.  Initializing the Chaincode

    ```go
    // InitLedger adds a base set of assets to the ledger
    func (s *SmartContract) InitLedger(ctx contractapi.TransactionContextInterface) error {
      assets := []Asset{
        {ID: "asset1", Color: "blue", Size: 5, Owner: "Tomoko", AppraisedValue: 300},
        {ID: "asset2", Color: "red", Size: 5, Owner: "Brad", AppraisedValue: 400},
        {ID: "asset3", Color: "green", Size: 10, Owner: "Jin Soo", AppraisedValue: 500},
        {ID: "asset4", Color: "yellow", Size: 10, Owner: "Max", AppraisedValue: 600},
        {ID: "asset5", Color: "black", Size: 15, Owner: "Adriana", AppraisedValue: 700},
        {ID: "asset6", Color: "white", Size: 15, Owner: "Michel", AppraisedValue: 800},
      }
    
      for _, asset := range assets {
        assetJSON, err := json.Marshal(asset)
        if err != nil {
            return err
        }
    
        err = ctx.GetStub().PutState(asset.ID, assetJSON)
        if err != nil {
            return fmt.Errorf("failed to put to world state. %v", err)
        }
      }
    
      return nil
    }
    ```

4.  CRUD

    ```go
    // CreateAsset issues a new asset to the world state with given details.
    func (s *SmartContract) CreateAsset(ctx contractapi.TransactionContextInterface, id string, color string, size int, owner string, appraisedValue int) error {
      exists, err := s.AssetExists(ctx, id)
      if err != nil {
        return err
      }
      if exists {
        return fmt.Errorf("the asset %s already exists", id)
      }
    
      asset := Asset{
        ID:             id,
        Color:          color,
        Size:           size,
        Owner:          owner,
        AppraisedValue: appraisedValue,
      }
      assetJSON, err := json.Marshal(asset)
      if err != nil {
        return err
      }
    
      return ctx.GetStub().PutState(id, assetJSON)
    }
    ```

    ```go
    // ReadAsset returns the asset stored in the world state with given id.
    func (s *SmartContract) ReadAsset(ctx contractapi.TransactionContextInterface, id string) (*Asset, error) {
      assetJSON, err := ctx.GetStub().GetState(id)
      if err != nil {
        return nil, fmt.Errorf("failed to read from world state: %v", err)
      }
      if assetJSON == nil {
        return nil, fmt.Errorf("the asset %s does not exist", id)
      }
    
      var asset Asset
      err = json.Unmarshal(assetJSON, &asset)
      if err != nil {
        return nil, err
      }
    
      return &asset, nil
    }
    ```

    ```go
    // UpdateAsset updates an existing asset in the world state with provided parameters.
    func (s *SmartContract) UpdateAsset(ctx contractapi.TransactionContextInterface, id string, color string, size int, owner string, appraisedValue int) error {
      exists, err := s.AssetExists(ctx, id)
      if err != nil {
        return err
      }
      if !exists {
        return fmt.Errorf("the asset %s does not exist", id)
      }
    
      // overwriting original asset with new asset
      asset := Asset{
        ID:             id,
        Color:          color,
        Size:           size,
        Owner:          owner,
        AppraisedValue: appraisedValue,
      }
      assetJSON, err := json.Marshal(asset)
      if err != nil {
        return err
      }
    
      return ctx.GetStub().PutState(id, assetJSON)
    }
    ```

    ```go
    // DeleteAsset deletes an given asset from the world state.
    func (s *SmartContract) DeleteAsset(ctx contractapi.TransactionContextInterface, id string) error {
      exists, err := s.AssetExists(ctx, id)
      if err != nil {
        return err
      }
      if !exists {
        return fmt.Errorf("the asset %s does not exist", id)
      }
    
      return ctx.GetStub().DelState(id)
    }
    
    ```

5.  AssetExists & TransferAsset

    ```go
    // AssetExists returns true when asset with given ID exists in world state
    func (s *SmartContract) AssetExists(ctx contractapi.TransactionContextInterface, id string) (bool, error) {
      assetJSON, err := ctx.GetStub().GetState(id)
      if err != nil {
        return false, fmt.Errorf("failed to read from world state: %v", err)
      }
    
      return assetJSON != nil, nil
    }
    ```

    ```go
    // TransferAsset updates the owner field of asset with given id in world state.
    func (s *SmartContract) TransferAsset(ctx contractapi.TransactionContextInterface, id string, newOwner string) error {
      asset, err := s.ReadAsset(ctx, id)
      if err != nil {
        return err
      }
    
      asset.Owner = newOwner
      assetJSON, err := json.Marshal(asset)
      if err != nil {
        return err
      }
    
      return ctx.GetStub().PutState(id, assetJSON)
    }
    ```

    ```go
    // GetAllAssets returns all assets found in world state
    func (s *SmartContract) GetAllAssets(ctx contractapi.TransactionContextInterface) ([]*Asset, error) {
      // range query with empty string for startKey and endKey does an
      // open-ended query of all assets in the chaincode namespace.
      resultsIterator, err := ctx.GetStub().GetStateByRange("", "")
      if err != nil {
        return nil, err
      }
      defer resultsIterator.Close()
    
      var assets []*Asset
      for resultsIterator.HasNext() {
        queryResponse, err := resultsIterator.Next()
        if err != nil {
          return nil, err
        }
    
        var asset Asset
        err = json.Unmarshal(queryResponse.Value, &asset)
        if err != nil {
          return nil, err
        }
        assets = append(assets, &asset)
      }
    
      return assets, nil
    }
    ```

6.  Put it all together

    ```sh
    func main() {
      assetChaincode, err := contractapi.NewChaincode(&SmartContract{})
      if err != nil {
        log.Panicf("Error creating asset-transfer-basic chaincode: %v", err)
      }
    
      if err := assetChaincode.Start(); err != nil {
        log.Panicf("Error starting asset-transfer-basic chaincode: %v", err)
      }
    }
    ```

7.  Chaincode access control

    Chaincode可以通過`ctx.GetStub().GetCreator()`利用憑證進行存取控制決策。此外，Fabric Contract API還提供了擴展API，以檢索該提交者資訊，無論是基於客戶身份本身，還是基於org身份，還是基於客戶身份屬性，進而做出存取控制決策。

8.  Managing external dependencies for chaincode written in Go

    因為chaincode依賴於不屬於標準函式庫的go package，這些套件的原始碼必須在你的chaincode package安裝到對等節點時被包含在內。最簡單的方式就是 go vendor

    ```
    go mod tidy
    go mod vendor
    ```

    一旦依賴關係被放置在你的鏈碼目錄中，``peer chaincode package` and `peer chaincode install`將會包含依賴關係相關的程式碼。

## Running chaincode in development mode(Lab 11)

在智能合約開發過程中，開發者需要一種方法來快速迭代測試鏈碼包，而不必為每次修改運行鏈碼生命週期命令。這允許快速部署、除錯和更新，而無需每次更新時重新下對等節點生命週期鏈碼命令。

```sh
## build the binaries for orderer, peer, and configtxgen:
cd fabric
make orderer peer configtxgen
```

```sh
export PATH=$(pwd)/build/bin:$PATH
export FABRIC_CFG_PATH=$(pwd)/sampleconfig

## genesisblock
configtxgen -profile SampleDevModeSolo -channelID syschannel -outputBlock genesisblock -configPath $FABRIC_CFG_PATH -outputBlock $(pwd)/sampleconfig/genesisblock
```

```sh
## Start the orderer（SampleDevModeSolo)
#### if panic: Error creating dir if missing: error while creating dir: /var/hyperledger/production/orderer/index: mkdir /var/hyperledger: permission denied
# sudo mkdir -p /var/hyperledger/production/orderer/index
# sudo chown -R $(whoami) /var/hyperledger
####
ORDERER_GENERAL_GENESISPROFILE=SampleDevModeSolo orderer
```

```sh
## Start the peer in DevMode
## new window
export PATH=$(pwd)/build/bin:$PATH
export FABRIC_CFG_PATH=$(pwd)/sampleconfig
FABRIC_LOGGING_SPEC=chaincode=debug CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052 peer node start --peer-chaincodedev=true
```

```sh
## Create channel and join peer
## new window

#### if Error: failed to initialize operations subsystem: listen tcp 127.0.0.1:9443: bind: address already in use
# change ./sampleconfig/core.yaml
# operations:
#	host and port for the operations server
#	listenAddress: 127.0.0.1:9444
#### ref.https://stackoverflow.com/questions/65387254/port-error-when-setting-up-dev-mode-of-hyperledger-fabric
export PATH=$(pwd)/build/bin:$PATH
export FABRIC_CFG_PATH=$(pwd)/sampleconfig
configtxgen -channelID ch1 -outputCreateChannelTx ch1.tx -profile SampleSingleMSPChannel -configPath $FABRIC_CFG_PATH
peer channel create -o 127.0.0.1:7050 -c ch1 -f ch1.tx
```

```sh
## Build the chaincode
go build -o simpleChaincode ./integration/chaincode/simple/cmd
```

>   When `DevMode` is enabled on the peer, the `CORE_CHAINCODE_ID_NAME` environment variable must be set to `<CHAINCODE_NAME>`:`<CHAINCODE_VERSION>` otherwise, the peer is unable to find the chaincode. 

```sh
## Start the chaincode(從 peer log 確認)
CORE_CHAINCODE_LOGLEVEL=debug CORE_PEER_TLS_ENABLED=false CORE_CHAINCODE_ID_NAME=mycc:1.0 ./simpleChaincode -peer.address 127.0.0.1:7052
```

```sh
## Approve and commit the chaincode definition
## new window

export PATH=$(pwd)/build/bin:$PATH
export FABRIC_CFG_PATH=$(pwd)/sampleconfig

peer lifecycle chaincode approveformyorg  -o 127.0.0.1:7050 --channelID ch1 --name atcc --version 1.0 --sequence 1 --init-required --signature-policy "OR ('SampleOrg.member')"  --package-id atcc:1.0
peer lifecycle chaincode checkcommitreadiness -o 127.0.0.1:7050 --channelID ch1 --name mycc --version 1.0 --sequence 1 --init-required --signature-policy "OR ('SampleOrg.member')"
peer lifecycle chaincode commit -o 127.0.0.1:7050 --channelID ch1 --name mycc --version 1.0 --sequence 1 --init-required --signature-policy "OR ('SampleOrg.member')" --peerAddresses 127.0.0.1:7051
```

```sh
## invoke
CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n mycc -c '{"Args":["init","a","100","b","200"]}' --isInit
CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n mycc -c '{"Args":["invoke","a","b","10"]}'
CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n mycc -c '{"Args":["query","a"]}'
```

試著部署 lab10 的 atcc

```sh
## go build
cd fabric-samples/atcc/
go build -o atcc
mv atcc ../../
cd ../../
```

```sh
## Start the chaincode(從 peer log 確認)
CORE_CHAINCODE_LOGLEVEL=debug CORE_PEER_TLS_ENABLED=false CORE_CHAINCODE_ID_NAME=atcc:1.0 ./simpleChaincode -peer.address 127.0.0.1:7052
```

```sh
## Approve and commit the chaincode definition
export PATH=$(pwd)/build/bin:$PATH
export FABRIC_CFG_PATH=$(pwd)/sampleconfig

peer lifecycle chaincode approveformyorg  -o 127.0.0.1:7050 --channelID ch1 --name atcc --version 1.0 --sequence 1 --signature-policy "OR ('SampleOrg.member')"  --package-id atcc:1.0

peer lifecycle chaincode commit -o 127.0.0.1:7050 --channelID ch1 --name atcc --version 1.0 --sequence 1 --signature-policy "OR ('SampleOrg.member')" --peerAddresses 127.0.0.1:7051
```

```sh
## invoke & query

## init
CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n atcc -c '{"function":"InitLedger","Args":[]}'

## c
CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n atcc -c '{"function":"CreateAsset","Args":["asset8","blue","16","Kelley","750"]}'

## r
CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode query -o 127.0.0.1:7050 -C ch1 -n atcc -c '{"Args":["ReadAsset","asset8"]}'

## u
CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n atcc -c '{"Args":["UpdateAsset", "asset8", "blue", "5", "Mark", "350"]}'

## d
CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n atcc -c '{"Args":["DeleteAsset","asset8"]}'

## transfer
CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n atcc -c '{"function":"TransferAsset","Args":["asset6","Mark"]}'

## get all
CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode query -o 127.0.0.1:7050 -C ch1 -n atcc -c '{"Args":["GetAllAssets"]}'
```

