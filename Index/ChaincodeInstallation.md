# Deploying a smart contract to a channel, the fabric chaincode lifecycle

In this tutorial we are going to install the medtransfer chaincode from the chaincodeDev environemnt.

Steps:
1. Start the network, create and join a channel
2. Package the smart contract
3. Install the chaincode package
4. Approve a chaincode definition
5. Committing the chaincode definition to the channel
6. Upgrade the chaincode


# Start network

```bash
# In terminal 1
docker-compose up

# Check running docker containers
docker ps
```

# Package the chaincode
```bash
# switch onto the folder
cd fabricDev/chaincode/medtransfer

# go needs a package and you have to install the dependencies
go mod init medproduct
go mod vendor

# switch back to your base folder
cd ../../
# Set environment variables
export FABRIC_CFG_PATH=../fabric-samples/config/
export GOPATH=$PWD

# package your chaincode
peer lifecycle chaincode package medtransfer.tar.gz --path chaincode/medtransfer/ --lang golang --label medtransfer_1

# inspect the tar package
tar tfz medtransfer.tar.gz

# extract the package and inspect
tar -xzf medtransfer.tar.gz

cat metadata.json | jq .
```

# Install the chaincode
After we packaged or chaincode, we can install the chaincode on our peers. The chaincode needs to be installed on every peer that will endorse a transaction. Because we are going to set the endorsement policy to require endorsements from logistic 1, logistic2 and pharmacy, we need to install the chaincode on the peers operated by all organizations:

peer0.logistic1.medtransfer.com
peer0.logistic2.medtransfer.com
peer0.pharmacy.medtransfer.com


```bash

1. Copy the medtransfer directory in your chaincodeDev environment and paste it inside your own network.

# Switch to andreas_network
cd ../
cd fabric-samples/andreas_network

```

 # Install the chaincode on logistic1
 ```bash
# Since we already have env files for each peer org with the correct environment variables, we don't need to repeat this process. However if you don't, follow this:
vi logistic1.env

export FABRIC_CFG_PATH=$PWD/../config/
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="logistic1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/logistic1.medtransfer.com/peers/peer0.logistic1.medtransfer.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/logistic1.medtransfer.com/users/Admin@logistic1.medtransfer.com/msp
export CORE_PEER_ADDRESS=localhost:7051

# Execute the env file
source logistic1.env
```

Call the peer lifecycle chaincode install command to install the chaincode on the peer.

```bash
peer lifecycle chaincode install medtransfer.tar.gz

```

As a result we will receive the chaincode package identifier.

```bash
Chaincode code package identifier: medtransfer_1:bf3adeff71ae1ba0b9bc708ad30924b8867cabf67c74c19a96d5791188038389
```

 # Install chaincode on logistic2
 ```bash
 # Since we already have env files for each peer org with the correct environment variables, we don't need to repeat this process. However if you don't, follow this:
 vi logistic2.env

 export FABRIC_CFG_PATH=$PWD/../config/
 export CORE_PEER_TLS_ENABLED=true
 export CORE_PEER_LOCALMSPID="logistic2MSP"
 export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/logistic2.medtransfer.com/peers/peer0.logistic2.medtransfer.com/tls/ca.crt
 export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/logistic2.medtransfer.com/users/Admin@logistic2.medtransfer.com/msp
 export CORE_PEER_ADDRESS=localhost:9051

 # Execute the env file
 source logistic2.env
 ```

 Call the peer lifecycle chaincode install command to install the chaincode on the peer.

 ```bash
 peer lifecycle chaincode install medtransfer.tar.gz

 ```

 As a result we will receive the chaincode package identifier.

 ```bash
 Chaincode code package identifier: medtransfer_1:bf3adeff71ae1ba0b9bc708ad30924b8867cabf67c74c19a96d5791188038389
 ```

# Install chaincode on pharmacy
 ```bash
 # Since we already have env files for each peer org with the correct environment variables, we don't need to repeat this process. However if you don't, follow this:
 vi pharmacy.env

 export FABRIC_CFG_PATH=$PWD/../config/
 export CORE_PEER_TLS_ENABLED=true
 export CORE_PEER_LOCALMSPID="pharmacyMSP"
 export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/pharmacy.medtransfer.com/peers/peer0.pharmacy.medtransfer.com/tls/ca.crt
 export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/pharmacy.medtransfer.com/users/Admin@pharmacy.medtransfer.com/msp
 export CORE_PEER_ADDRESS=localhost:10051

 # Execute the env file
 source pharmacy.env
 ```

 Call the peer lifecycle chaincode install command to install the chaincode on the peer.

 ```bash
 peer lifecycle chaincode install medtransfer.tar.gz

 ```

 As a result we will receive the chaincode package identifier.

 ```bash
 Chaincode code package identifier: medtransfer_1:bf3adeff71ae1ba0b9bc708ad30924b8867cabf67c74c19a96d5791188038389
 ```

# Approve the chaincode definition for each org
After you install the chaincode package, you need to approve a chaincode definition for your organization. If an organization has installed the chaincode on their peer, they need to include the packageID in the chaincode definition approved by their organization. The package ID is used to associate the chaincode installed on a peer with an approved chaincode definition, and allows an organization to use the chaincode to endorse transactions. You can find the package ID of a chaincode by using the peer lifecycle chaincode queryinstalled command to query your peer.

```bash
peer lifecycle chaincode queryinstalled
```

To approve to the chaincode we need one more environment variable, CC_PACKAGE_ID.
```bash 
export CC_PACKAGE_ID=medtransfer_1:bf3adeff71ae1ba0b9bc708ad30924b8867cabf67c74c19a96d5791188038389
```
 # Approve chaincode for logistic1
  ```bash
  source logistic1.env
  
 # Command to copy
 peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.medtransfer.com --channelID channel1 --name medtransfer --version 1 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/medtransfer.com/orderers/orderer.medtransfer.com/msp/tlscacerts/tlsca.medtransfer.com-cert.pem
 
 # Check approval progress
 peer lifecycle chaincode checkcommitreadiness --channelID channel1 --name medtransfer --version 1 --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/medtransfer.com/orderers/orderer.medtransfer.com/msp/tlscacerts/tlsca.medtransfer.com-cert.pem --output json

 ```

  # Approve chaincode for logistic2
  ```bash
  source logistic2.env
  
 # Command to copy
 peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.medtransfer.com --channelID channel1 --name medtransfer --version 1 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/medtransfer.com/orderers/orderer.medtransfer.com/msp/tlscacerts/tlsca.medtransfer.com-cert.pem
 
 # Check approval progress
 peer lifecycle chaincode checkcommitreadiness --channelID channel1 --name medtransfer --version 1 --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/medtransfer.com/orderers/orderer.medtransfer.com/msp/tlscacerts/tlsca.medtransfer.com-cert.pem --output json

 ```
  
  # Approve chaincode for pharmacy
  ```bash
  source pharmacy.env
  
 # Command to copy
 peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.medtransfer.com --channelID channel1 --name medtransfer --version 1 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/medtransfer.com/orderers/orderer.medtransfer.com/msp/tlscacerts/tlsca.medtransfer.com-cert.pem
 
 # Check approval progress
 peer lifecycle chaincode checkcommitreadiness --channelID channel1 --name medtransfer --version 1 --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/medtransfer.com/orderers/orderer.medtransfer.com/msp/tlscacerts/tlsca.medtransfer.com-cert.pem --output json

 ```

# Committing the chaincode definition to the channel

```bash
# Command to copy
peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.medtransfer.com --channelID channel1 --name medtransfer --version 1 --sequence 1 --tls --cafile ${PWD}/organizations/ordererOrganizations/medtransfer.com/orderers/orderer.medtransfer.com/msp/tlscacerts/tlsca.medtransfer.com-cert.pem --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/logistic1.medtransfer.com/peers/peer0.logistic1.medtransfer.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/logistic2.medtransfer.com/peers/peer0.logistic2.medtransfer.com/tls/ca.crt --peerAddresses localhost:10051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/pharmacy.medtransfer.com/peers/peer0.pharmacy.medtransfer.com/tls/ca.crt

```
You can use the peer lifecycle chaincode querycommitted command to confirm that the chaincode definition has been committed to the channel.

```bash
peer lifecycle chaincode querycommitted --channelID channel1 --name medtransfer --cafile ${PWD}/organizations/ordererOrganizations/medtransfer.com/orderers/orderer.medtransfer.com/msp/tlscacerts/tlsca.medtransfer.com-cert.pem
```

# Testing the chaincoed with CLI Commands
````bash
# Init the ledger
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.medtransfer.com  --tls --cafile ${PWD}/organizations/ordererOrganizations/medtransfer.com/orderers/orderer.medtransfer.com/msp/tlscacerts/tlsca.medtransfer.com-cert.pem -C channel1 -n medtransfer --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/logistic1.medtransfer.com/peers/peer0.logistic1.medtransfer.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/logistic2.medtransfer.com/peers/peer0.logistic2.medtransfer.com/tls/ca.crt --peerAddresses localhost:10051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/pharmacy.medtransfer.com/peers/peer0.pharmacy.medtransfer.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}' 

# Query the ledger with ReadAsset
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.medtransfer.com  --tls --cafile ${PWD}/organizations/ordererOrganizations/medtransfer.com/orderers/orderer.medtransfer.com/msp/tlscacerts/tlsca.medtransfer.com-cert.pem -C channel1 -n medtransfer --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/logistic1.medtransfer.com/peers/peer0.logistic1.medtransfer.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/logistic2.medtransfer.com/peers/peer0.logistic2.medtransfer.com/tls/ca.crt --peerAddresses localhost:10051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/pharmacy.medtransfer.com/peers/peer0.pharmacy.medtransfer.com/tls/ca.crt -c '{"function":"ReadAsset","Args":["asset1"]}' 

# CreateAsset
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.medtransfer.com  --tls --cafile ${PWD}/organizations/ordererOrganizations/medtransfer.com/orderers/orderer.medtransfer.com/msp/tlscacerts/tlsca.medtransfer.com-cert.pem -C channel1 -n medtransfer --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/logistic1.medtransfer.com/peers/peer0.logistic1.medtransfer.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/logistic2.medtransfer.com/peers/peer0.logistic2.medtransfer.com/tls/ca.crt --peerAddresses localhost:10051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/pharmacy.medtransfer.com/peers/peer0.pharmacy.medtransfer.com/tls/ca.crt -c '{"function":"CreateAsset","Args":["Id", "CarLicenceNumber", "CurrentTemperature"]}' 

# Update Asset
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.medtransfer.com  --tls --cafile ${PWD}/organizations/ordererOrganizations/medtransfer.com/orderers/orderer.medtransfer.com/msp/tlscacerts/tlsca.medtransfer.com-cert.pem -C channel1 -n medtransfer --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/logistic1.medtransfer.com/peers/peer0.logistic1.medtransfer.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/logistic2.medtransfer.com/peers/peer0.logistic2.medtransfer.com/tls/ca.crt --peerAddresses localhost:10051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/pharmacy.medtransfer.com/peers/peer0.pharmacy.medtransfer.com/tls/ca.crt -c '{"function":"UpdateAsset","Args":["Id", "CarLicenceNumber", "CurrentTemperature"]}' 

# Delete Asset
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.medtransfer.com  --tls --cafile ${PWD}/organizations/ordererOrganizations/medtransfer.com/orderers/orderer.medtransfer.com/msp/tlscacerts/tlsca.medtransfer.com-cert.pem -C channel1 -n medtransfer --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/logistic1.medtransfer.com/peers/peer0.logistic1.medtransfer.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/logistic2.medtransfer.com/peers/peer0.logistic2.medtransfer.com/tls/ca.crt --peerAddresses localhost:10051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/pharmacy.medtransfer.com/peers/peer0.pharmacy.medtransfer.com/tls/ca.crt -c '{"function":"DeleteAsset","Args":["Id"]}' 

# Get All Assets
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.medtransfer.com  --tls --cafile ${PWD}/organizations/ordererOrganizations/medtransfer.com/orderers/orderer.medtransfer.com/msp/tlscacerts/tlsca.medtransfer.com-cert.pem -C channel1 -n medtransfer --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/logistic1.medtransfer.com/peers/peer0.logistic1.medtransfer.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/logistic2.medtransfer.com/peers/peer0.logistic2.medtransfer.com/tls/ca.crt --peerAddresses localhost:10051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/pharmacy.medtransfer.com/peers/peer0.pharmacy.medtransfer.com/tls/ca.crt -c '{"function":"GetAllAsset","Args":[]}' 

´´´



