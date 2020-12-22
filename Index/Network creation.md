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











