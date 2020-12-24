# How to create your own fabric network
In this guide you will create a persistent three org fabric network with a one node RAFT orderer.

We need the following files:

- crypto-config.yaml (create identities)
- configtx.yaml (system channel and application channel config)
- docker-compose.yaml (create a docker network with some containers)

# Preparation
```bash
# Locate to correct directory
cd fabric-samples

# Create new base folder inside the fabric-samples folder
mkdir own-network

# Copy a few files from the test-network
mkdir configtx
cp ../test-network/configtx/* configtx/
cp ../test-network/docker/docker-compose-test-net.yaml ./docker-compose.yaml
```

# Create Crypto-config.yaml file
```bash
vi crypto-config.yaml
cat ../test-network/organizations/cryptogen/crypto-config-orderer.yaml >> crypto-config.yaml
cat ../test-network/organizations/cryptogen/crypto-config-org1.yaml >> crypto-config.yaml

# Add section for org2 and org3
```

# Modify the configtx.yaml file

Add new section for org3

Add new profiles: ThreeOrgsOrdererGenesis, ThreeOrgsChannel

If you have changed names for the organizations, and domain (eg.logistic1.medtransfer.com), then you have to replace all organization names and container names in the configtx.yaml file, the crypto-config.yaml file, and the docker-compose.yaml file with your new names. 

# Generate Artifacts
```bash 
# Create a path that tells the configtxgen tool where to look for the confix.yaml file
export FABRIC_CFG_PATH=$PWD/configtx

# Create variables for the channel names
export CHANNEL_NAME=channel1 
export SYS_CHANNEL_NAME=sys-channel

# Generate Artifacts (Identities)
cryptogen generate --config=./crypto-config.yaml --output organizations

# Create folders for the channel-artifacts and the system-genesis-block
mkdir channel-artifacts
mkdir system-genesis-block

# Create Genesis Block
configtxgen -profile ThreeOrgsOrdererGenesis -channelID $SYS_CHANNEL_NAME -outputBlock ./system-genesis-block/genesis.block

# Create a Channel Configuration Transaction
configtxgen -profile ThreeOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel_$CHANNEL_NAME.tx -channelID $CHANNEL_NAME

#Create Anchor Peer Transactions for each Peer Org
configtxgen -profile ThreeOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/logistic1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg logistic1MSP

configtxgen -profile ThreeOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/logistic2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg logistic2MSP

configtxgen -profile ThreeOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/pharmacyMSPanchors.tx -channelID $CHANNEL_NAME -asOrg pharmacyMSP



# Check your work using the tree tool
tree ./organizations -L 2
tree ./channel-artifacts
tree ./system-genesis-block
```

# Start Network
Modify the docker-compose.yaml file. This includes making changes to the CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE docker variable and creating a .env file. Remember to also change all the organization names.

```bash
- CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_own-network

vi .env
COMPOSE_PROJECT_NAME=andreas-network
IMAGE_TAG=latest
SYS_CHANNEL=system-channel

# Remember to change the network names to andreas-network
networks:
  - andreas-network
  
# Start network in terminal 1
docker-compose up
```

# Create Channel

```bash
# Terminal 2
cd fabric/fabric-samples/andreas-network

# Set some env vars
export FABRIC_CFG_PATH=$PWD/../config/
export CHANNEL_NAME=channel1 
export ORDERER_CA=${PWD}/organizations/ordererOrganizations/medtransfer.com/orderers/orderer.medtransfer.com/msp/tlscacerts/tlsca.medtransfer.com-cert.pem


# Create env files for each organization

- create logistic1.env
- create logistic2.env
- create pharmacy.env

vi org1.env
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/logistic1.medtransfer.com/peers/peer0.logistic1.medtransfer.com/tls/ca.crt
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.medtransfer.com/users/Admin@logistic1.medtransfer.com/msp
export CORE_PEER_ADDRESS=localhost:7051
export FABRIC_CFG_PATH=$PWD/../config/

Do this for logistic2 and pharmacy


# Switch to logistic1.env
source logistic1.env

# Create Channel
peer channel create -o localhost:7050 -c $CHANNEL_NAME --ordererTLSHostnameOverride orderer.medtransfer.com -f ./channel-artifacts/channel_${CHANNEL_NAME}.tx --outputBlock ./channel-artifacts/${CHANNEL_NAME}.block --tls --cafile $ORDERER_CA 

# Join logistic1 to channel
peer channel join -b ./channel-artifacts/$CHANNEL_NAME.block

# Join logistic2 to channel
source logistic2.env
peer channel join -b ./channel-artifacts/$CHANNEL_NAME.block

# Join pharmacy to channel
source pharmacy.env
peer channel join -b ./channel-artifacts/$CHANNEL_NAME.block

Check the results with peer channel list

# Update anchor peer for each org

source logistic1.env
peer channel update -o localhost:7050 --ordererTLSHostnameOverride orderer.medtransfer.com -c $CHANNEL_NAME -f ./channel-artifacts/logistic1MSPanchors.tx --tls --cafile $ORDERER_CA 

source logistic2.env
peer channel update -o localhost:7050 --ordererTLSHostnameOverride orderer.medtransfer.com -c $CHANNEL_NAME -f ./channel-artifacts/logistic2MSPanchors.tx --tls --cafile $ORDERER_CA 

source pharmacy.env
peer channel update -o localhost:7050 --ordererTLSHostnameOverride orderer.medtransfer.com -c $CHANNEL_NAME -f ./channel-artifacts/pharmacyMSPanchors.tx --tls --cafile $ORDERER_CA 

```

At this point the network and channel configuration are ready.

# Install Chaincode

```bash

# Create folder for chaincode
mkdir chaincode
cp -r ../chaincode/abstore/go/ chaincode/abstore/

# If needed
rm -r  chaincode/abstore/vendor

cd ./chaincode/abstore

# Install (external) go dependencies
GO111MODULE=on go mod vendor

##Fabric Chaincode Lifecycle

# Step 1- Package the chaincode
cd ../../
peer lifecycle chaincode package basic.tar.gz --path ./chaincode/abstore/ --lang golang --label basic_1

# Check the content
tar -tvf basic.tar.gz

# Install CC on peer 0 logistic1
source logistic1.env
peer lifecycle chaincode install basic.tar.gz

# Install CC on peer 0 logistic2
source logistic2.env
peer lifecycle chaincode install basic.tar.gz

# Install CC on peer 0 pharmacy
source pharmacy.env
peer lifecycle chaincode install basic.tar.gz

## basic_1:d44a118ea789f00646aec920719320c9c177a68c59150195ec479f3b42e1a672

# Switch back to logistic1
source logistic1.env
export PKGID=basic_1:d44a118ea789f00646aec920719320c9c177a68c59150195ec479f3b42e1a672

# Approve CC for logistic1
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.medtransfer.com --tls --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name basic --version 1 --package-id $PKGID --sequence 1

# Approve CC for logistic2
source logistic2.env
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.medtransfer.com --tls --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name basic --version 1 --package-id $PKGID --sequence 1

# Approve CC for pharmacy
source pharmacy.env
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.medtransfer.com --tls --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name basic --version 1 --package-id $PKGID --sequence 1

# Check readiness
peer lifecycle chaincode checkcommitreadiness --channelID $CHANNEL_NAME --name basic --version 1 --sequence 1 --tls --cafile $ORDERER_CA --output json

## If the CC has been approved for each org, "true" will be shown next to each org. 

# Commit the CC
source logistic1.env

Note it is important to send the commit statement to at least 2 orgs, because of the default chaincode endorsement lifecyle rule: MAJORITY

peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.medtransfer.com --channelID $CHANNEL_NAME --name basic --version 1 --sequence 1 --tls --cafile $ORDERER_CA --peerAddresses localhost:7051 --tlsRootCertFiles
${PWD}/organizations/peerOrganizations/logistic1.medtransfer.com/peers/peer0.logistic1.medtransfer.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/logistic2.medtransfer.com/peers/peer0.logistic2.medtransfer.com/tls/ca.crt --peerAddresses localhost:10051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/pharmacy.medtransfer.com/peers/peer0.pharmacy.medtransfer.com/tls/ca.crt

# Check the result
peer lifecycle chaincode querycommitted --channelID $CHANNEL_NAME --name basic --cafile $ORDERER_CA
```

# Use the chaincode

```bash
# First Init the Chaincode
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.medtransfer.com --tls --cafile $ORDERER_CA -C $CHANNEL_NAME -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/logistic1.medtransfer.com/peers/peer0.logistic1.medtransfer.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/logistic2.medtransfer.com/peers/peer0.logistic2.medtransfer.com/tls/ca.crt  --peerAddresses localhost:10051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/pharmacy.medtransfer.com/peers/peer0.pharmacy.medtransfer.com/tls/ca.crt -c '{"function":"Init","Args":["account1","1000","account2","10"]}'

# Query the Chaincode
peer chaincode query -C $CHANNEL_NAME -n basic -c '{"function":"Query","Args":["account1"]}'

# Invoke the Chaincode from org3
source org3.env
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.medtransfer.com --tls --cafile $ORDERER_CA -C $CHANNEL_NAME -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/logistic1.medtransfer.com/peers/peer0.logistic1.medtransfer.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/logistic2.medtransfer.com/peers/peer0.logistic2.medtransfer.com/tls/ca.crt  --peerAddresses localhost:10051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/pharmacy.medtransfer.com/peers/peer0.pharmacy.medtransfer.com/tls/ca.crt -c '{"function":"Invoke","Args":["account1","account2","100"]}'
```

# A Persistent Network

```bash
# Stop the network
docker-compose down

# Start the network in the background
docker-compose up -d

# Show the logs
docker-compose logs -f -t

# Notice that your system is persistent and you can start the network as long as you do not clean up the docker volumes.
docker volume ls
docker volume prune

# Check your containers
docker-compose ps

# To see the chaincode container
docker ps
```

















