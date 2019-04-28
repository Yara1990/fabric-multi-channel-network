# fabric-multi-channel-network

#Setup a Multi-Channel Network in Hyperledger Fabric


#Steps Involved:

$ mkdir hyperledgerfabric13

$ cd hyperledgefabric13

$ curl -sSL http://bit.ly/2ysbOFE | bash -s 1.3.0

$ mkdir multi-channel-network

$ cd multi-channel-network

$  ../fabric-samples/bin/cryptogen generate --config=./crypto-config.yaml

$ mkdir channel-artifacts

$ ../bin/configtxgen -profile OrdererGenesis -outputBlock ./channel-artifacts/genesis.block

$ export CHANNEL_ONE_NAME=channelall

$ export CHANNEL_ONE_PROFILE=ChannelAll

$ export CHANNEL_TWO_NAME=channel12

$ export CHANNEL_TWO_PROFILE=Channel12

$ ../bin/configtxgen -profile ${CHANNEL_ONE_PROFILE} -outputCreateChannelTx ./channel-artifacts/${CHANNEL_ONE_NAME}.tx -channelID $CHANNEL_ONE_NAME

$ ../bin/configtxgen -profile ${CHANNEL_TWO_PROFILE} -outputCreateChannelTx ./channel-artifacts/${CHANNEL_TWO_NAME}.tx -channelID $CHANNEL_TWO_NAME

$ ../bin/configtxgen -profile ${CHANNEL_ONE_PROFILE} -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors_${CHANNEL_ONE_NAME}.tx -channelID $CHANNEL_ONE_NAME -asOrg Org1MSP

$ ../bin/configtxgen -profile ${CHANNEL_ONE_PROFILE} -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors_${CHANNEL_ONE_NAME}.tx -channelID $CHANNEL_ONE_NAME -asOrg Org2MSP

$ ../bin/configtxgen -profile ${CHANNEL_ONE_PROFILE} -outputAnchorPeersUpdate ./channel-artifacts/Org3MSPanchors_${CHANNEL_ONE_NAME}.tx -channelID $CHANNEL_ONE_NAME -asOrg Org3MSP

$ ../bin/configtxgen -profile ${CHANNEL_TWO_PROFILE} -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors_${CHANNEL_TWO_NAME}.tx -channelID $CHANNEL_TWO_NAME -asOrg Org1MSP

$ ../bin/configtxgen -profile ${CHANNEL_TWO_PROFILE} -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors_${CHANNEL_TWO_NAME}.tx -channelID $CHANNEL_TWO_NAME -asOrg Org2MSP

$ docker-compose -f docker-compose.yaml up -d

# designate three new terminals for Org1, Org2 and Org3.

For Org1 (default)

$ docker exec -it cli bash

For Org2 (specifying the environment variables for Org2)

$ docker exec -e "CORE_PEER_LOCALMSPID=Org2MSP" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp" -e "CORE_PEER_ADDRESS=peer0.org2.example.com:7051" -it cli bash

For Org3 (specifying the environment variables for Org3)

$ docker exec -e "CORE_PEER_LOCALMSPID=Org3MSP" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.example.com/users/Admin@org3.example.com/msp" -e "CORE_PEER_ADDRESS=peer0.org3.example.com:7051" -it cli bash

For easy access, export this parameter ORDERER_CA in all three terminals.

export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem


Create the channel block file (any Terminal)

peer channel create -o orderer.example.com:7050 -c channelall -f /opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts/channelall.tx --tls --cafile $ORDERER_CA


Org1 Terminal

 peer channel join -b channelall.block --tls --cafile $ORDERER_CA
 
peer channel update -o orderer.example.com:7050 -c channelall -f /opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts/Org1MSPanchors_channelall.tx --tls --cafile $ORDERER_CA

Org2 Terminal

 peer channel join -b channelall.block --tls --cafile $ORDERER_CA
 
 peer channel update -o orderer.example.com:7050 -c channelall -f /opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts/Org2MSPanchors_channelall.tx --tls --cafile $ORDERER_CA
 
Org3 Terminal


 peer channel join -b channelall.block --tls --cafile $ORDERER_CA
 

 peer channel update -o orderer.example.com:7050 -c channelall -f /opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts/Org3MSPanchors_channelall.tx --tls --cafile $ORDERER_CA
 
Create the channel block file (any Terminal)

 peer channel create -o orderer.example.com:7050 -c channel12 -f /opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts/channel12.tx --tls --cafile $ORDERER_CA

Now a file channel12.block is created.
Join the two peers to this channel and update the anchor peer for each peer.


Org1 Terminal

 peer channel join -b channel12.block --tls --cafile $ORDERER_CA
 
 peer channel update -o orderer.example.com:7050 -c channel12 -f /opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts/Org1MSPanchors_channel12.tx --tls --cafile $ORDERER_CA
 
Org2 Terminal

peer channel join -b channel12.block --tls --cafile $ORDERER_CA

peer channel update -o orderer.example.com:7050 -c channel12 -f /opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts/Org2MSPanchors_channel12.tx --tls --cafile $ORDERER_CA

Install Simple Asset Chaincode (SACC)


To install the chaincode, on each terminal,

 peer chaincode install -n sacc -p github.com/chaincode/sacc -v 1.0
On Org1 Terminal, we instantiate the code on ChannelAll.

 peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile $ORDERER_CA -C channelall -c '{"Args":["a", "100"]}' -n sacc -v 1.0 -P "OR('Org1MSP.peer', 'Org2MSP.peer', 'Org3MSP.peer')"


On Org1 Terminal,

peer chaincode query -C channelall -n sacc -c '{"Args":["get","a"]}'

Now on Org2 Terminal,

peer chaincode query -C channelall -n sacc -c '{"Args":["get","a"]}'

And on Org3 Terminal,

# peer chaincode query -C channelall -n sacc -c '{"Args":["get","a"]}'

Instantiate and Interact with Chaincode on Channel12
Now we instantiate the same SACC on Channel12.
On Org1 Terminal,

 peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile $ORDERER_CA -C channel12 -c '{"Args":["b", "200"]}' -n sacc -v 1.0 -P "OR('Org1MSP.peer', 'Org2MSP.peer')"


Then again we begin with Org1 Terminal.

 peer chaincode query -C channel12 -n sacc -c '{"Args":["get","b"]}'
Then on Org2 Terminal.

peer chaincode query -C channel12 -n sacc -c '{"Args":["get","b"]}'


in Org1 or Org2 Terminal,
 peer chaincode query -C channel12 -n sacc -c '{"Args":["get","a"]}'
