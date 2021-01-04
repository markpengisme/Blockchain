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

