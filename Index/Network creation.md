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








