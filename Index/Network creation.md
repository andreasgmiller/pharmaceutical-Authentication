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

# Copy files a few files from the test-network
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
configtxgen -profile ThreeOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP

configtxgen -profile ThreeOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP

configtxgen -profile ThreeOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org3MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org3MSP

Note: Remember to replace each orgMSP with the name of your organization. Eg. Change Org1MSP to logistic1MSP.

# Check your work using the tree tool
tree ./organizations -L 2
tree ./channel-artifacts
tree ./system-genesis-block
```

# Start Network
Modify the docker-compose.yaml file. This includes making changes to the CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE docker variable and creating a .env file. 

```bash
- CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_own-network

vi .env
COMPOSE_PROJECT_NAME=own-network
IMAGE_TAG=latest
SYS_CHANNEL=system-channel

# Remember to change the network names to own-network
networks:
  - own-network
  
# Start network in terminal 1
docker-compose up
```

# Create Channel

```bash
# Terminal 2
cd fabric/fabric-samples/own-network

# Set some env vars
export FABRIC_CFG_PATH=$PWD/../config/
export CHANNEL_NAME=channel1 
export ORDERER_CA=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

Note (Only if you changed org and container names): Make sure to change the example domain name with your own domain name when exporting the ORDERER_CA variable. Eg. example.com ---> logistic1.com

# Create env files for each organization

- create org1.env
- create org2.env
- create org3.env

vi org1.env
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
export FABRIC_CFG_PATH=$PWD/../config/

Do this for org2 and org3


# Switch to org1.env
source org1.env

# Create Channel
peer channel create -o localhost:7050 -c $CHANNEL_NAME --ordererTLSHostnameOverride orderer.example.com -f ./channel-artifacts/channel_${CHANNEL_NAME}.tx --outputBlock ./channel-artifacts/${CHANNEL_NAME}.block --tls --cafile $ORDERER_CA 

# Join org1 to channel
peer channel join -b ./channel-artifacts/$CHANNEL_NAME.block

# Join org2 to channel
source org2.env
peer channel join -b ./channel-artifacts/$CHANNEL_NAME.block

# Join org3 to channel
source org3.env
peer channel join -b ./channel-artifacts/$CHANNEL_NAME.block

Check the results with peer channel list

# Update anchor peer for each org

source org1.env
peer channel update -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com -c $CHANNEL_NAME -f ./channel-artifacts/Org1MSPanchors.tx --tls --cafile $ORDERER_CA 

source org2.env
peer channel update -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com -c $CHANNEL_NAME -f ./channel-artifacts/Org2MSPanchors.tx --tls --cafile $ORDERER_CA 

source org3.env
peer channel update -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com -c $CHANNEL_NAME -f ./channel-artifacts/Org3MSPanchors.tx --tls --cafile $ORDERER_CA 

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

Fabric Chaincode Lifecycle

# Step 1- Package the chaincode
cd ../../
peer lifecycle chaincode package basic.tar.gz --path ./chaincode/abstore/ --lang golang --label basic_1

# Check the content
tar -tvf basic.tar.gz

# Install CC on peer 0 Org1
source org1.env
peer lifecycle chaincode install basic.tar.gz

# Install CC on peer 0 Org2
source org2.env
peer lifecycle chaincode install basic.tar.gz

# Install CC on peer 0 Org3
source org3.env
peer lifecycle chaincode install basic.tar.gz

basic_1:d44a118ea789f00646aec920719320c9c177a68c59150195ec479f3b42e1a672

# Switch back to org1
source org1.env
export PKGID=basic_1:d44a118ea789f00646aec920719320c9c177a68c59150195ec479f3b42e1a672

# Approve CC for org1
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name basic --version 1 --package-id $PKGID --sequence 1

# Approve CC for org2
source org2.env
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name basic --version 1 --package-id $PKGID --sequence 1

# Approve CC for org3
source org3.env
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name basic --version 1 --package-id $PKGID --sequence 1

# Check readiness
peer lifecycle chaincode checkcommitreadiness --channelID $CHANNEL_NAME --name basic --version 1 --sequence 1 --tls --cafile $ORDERER_CA --output json

If the CC has been approved for each org, "true" will be shown next to each org. 

# Commit the CC
source org1.env

Note it is important to send the commit statement to at least 2 orgs, because of the default chaincode endorsement lifecyle rule: MAJORITY

peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID $CHANNEL_NAME --name basic --version 1 --sequence 1 --tls --cafile $ORDERER_CA --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt --peerAddresses localhost:10051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt

# Check the result
peer lifecycle chaincode querycommitted --channelID $CHANNEL_NAME --name basic --cafile $ORDERER_CA
```

















