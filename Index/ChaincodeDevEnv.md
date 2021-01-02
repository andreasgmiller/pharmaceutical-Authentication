# Fabric 2.2 Chaincode DevMode Environment
These are the instructions to install the necessary devmode environment for chaincode development. 

# Set up environment
```bash
mkdir fabricDev
cd fabricDev
git clone https://github.com/hyperledger/fabric.git

cd fabric

# Run the following command to build the binaries for the orderer, peer, and configtxgen
make orderer peer configtxgen

# If you do not have the make command or GCC command installed, use this:
apt install make

apt install gcc

# Set the PATH environment variable to include orderer and peer binaries:
export PATH=$(pwd)/build/bin:$PATH

# Set the FABRIC_CFG_PATH environment variable to point to the sampleconfig folder
export FABRIC_CFG_PATH=$(pwd)/sampleconfig

# Generate the genesis block for the ordering service
configtxgen -profile SampleDevModeSolo -channelID syschannel -outputBlock genesisblock -configPath $FABRIC_CFG_PATH -outputBlock $(pwd)/sampleconfig/genesisblock
```

# Start the orderer
```bash
# In terminal 1
ORDERER_GENERAL_GENESISPROFILE=SampleDevModeSolo orderer
```

# Start peer in DevMode
```bash
# In terminal 2

# Make sure to modify the core.yaml file, and change the port to 10443
operations:
  # host and port for the operations server
  listenAddress: 127.0.0.1:10443

# Set the paths
export PATH=$(pwd)/build/bin:$PATH
export FABRIC_CFG_PATH=$(pwd)/sampleconfig

FABRIC_LOGGING_SPEC=chaincode=debug CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052 peer node start --peer-chaincodedev=true
```

# Create the channel
```bash
# In terminal 3
# Set the paths
export PATH=$(pwd)/build/bin:$PATH
export FABRIC_CFG_PATH=$(pwd)/sampleconfig

configtxgen -channelID ch1 -outputCreateChannelTx ch1.tx -profile SampleSingleMSPChannel -configPath $FABRIC_CFG_PATH

# Create the channel
peer channel create -o 127.0.0.1:7050 -c ch1 -f ch1.tx

# Join the channel
peer channel join -b ch1.block
```

# Build the chaincode
```bash
# Use the simple chaincode from the fabric/integration/chaincode directory to demonstrate how to run a chaincode package in DevMode. 
go build -o simpleChaincode ./integration/chaincode/simple/cmd

# Add command for your own chaincode later!

# Start the chaincode
CORE_CHAINCODE_LOGLEVEL=debug CORE_PEER_TLS_ENABLED=false CORE_CHAINCODE_ID_NAME=mycc:1.0 ./simpleChaincode -peer.address 127.0.0.1:7052
```

# Approve and commit the chaincode
```bash
# In terminal 4
# Set the paths
export PATH=$(pwd)/build/bin:$PATH
export FABRIC_CFG_PATH=$(pwd)/sampleconfig

peer lifecycle chaincode approveformyorg  -o 127.0.0.1:7050 --channelID ch1 --name mycc --version 1.0 --sequence 1 --init-required --signature-policy "OR ('SampleOrg.member')" --package-id mycc:1.0

peer lifecycle chaincode checkcommitreadiness -o 127.0.0.1:7050 --channelID ch1 --name mycc --version 1.0 --sequence 1 --init-required --signature-policy "OR ('SampleOrg.member')"

peer lifecycle chaincode commit -o 127.0.0.1:7050 --channelID ch1 --name mycc --version 1.0 --sequence 1 --init-required --signature-policy "OR ('SampleOrg.member')" --peerAddresses 127.0.0.1:7051
```

# Test the chaincode
```bash
# In terminal 4
CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n mycc -c '{"Args":["init","a","100","b","200"]}' --isInit
CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n mycc -c '{"Args":["invoke","a","b","10"]}'
CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n mycc -c '{"Args":["query","a"]}'

CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode invoke -o 127.0.0.1:7050 -C ch1 -n mycc -c '{"Args":["InitLedger"]}' --isInit
CORE_PEER_ADDRESS=127.0.0.1:7051 peer chaincode query -o 127.0.0.1:7050 -C ch1 -n mycc -c '{"Args":["ReadAsset","asset1"]}' | jq .
```





