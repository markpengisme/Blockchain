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

