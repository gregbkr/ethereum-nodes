## Ethereum nodes ##

![Ethereum-nodes.png](https://github.com/gregbkr/ethereum-nodes-docker/raw/master/Ethereum-nodes1.PNG)

In few commands: (thx to docker compose ;)
- run both clients (parity & geth)
- check accounts and status
- make a transaction in python (transfert test eth between the nodes)

I use this setup for production
- Data stays on host volumes: container nodes can be destroy without lossing the data --> easy version migration
- Auto unlock wallets when container is starting (please use firewall to protect your prod nodes!)
 
More info: you can find an overview of that setup on my blog: https://greg.satoshi.tech/

# 0. Prerequisit:

- Ubuntu or other flavor
- Docker
- Docker-compose

Accounts info:
- gethtest: 0x881bce7131857c9d24effdbe39ea974fd3c43e50
- paritytest: 0x9e6c69b73f5808a25404edcf7eac7bf8ee935568
- password: notsecure

# 1. Setup git

    git clone https://github.com/gregbkr/ethereum-nodes-docker eth && cd eth

# 2. Setup ethereum Nodes:

Setup local volumes for blockchains

    docker volume create --name eth_gethtest
    docker volume create --name eth_paritytest

Use already existing account and password (nice so you don't have to load ethtest)

    cp -r gethtest/* /var/lib/docker/volumes/eth_gethtest/_data/
    cp -r paritytest/* /var/lib/docker/volumes/eth_paritytest/_data/

OR if you want to use a fresh new wallet

    docker run --rm -it -v eth_paritytest:/root/.parity ethcore/parity --testnet account new
    ls /var/lib/docker/volumes/eth_paritytest/_data/testnet_keys/  <-- your test wallet is here

    docker run --rm -it -v eth_gethtest:/root/.ethereum ethereum/client-go --testnet account new
    ls /var/lib/docker/volumes/eth_gethtest/_data/testnet/keystore/  <-- your test wallet is here

OR in a running container

  docker run -d --name geth0 -v eth_gethtest:/root/.ethereum ethereum/client-go --testnet
  docker exec -it geth0 bash -c "geth --testnet --password <(echo -n notsecure) account new"
  docker stop geth0 && docker rm geth0

And setup password:

    echo -n 'notsecure' > /var/lib/docker/volumes/eth_paritytest/_data/testnet_keys/mypass     <-- unsure that there is no space or new line in file: don't use nano, it leave a new line!!


# 3. Run containers

    docker-compose up -d

# 4. Checks 

Accounts:

    docker exec eth_paritytest_1 /build/parity/target/release/parity --testnet account list
    docker exec eth_gethtest_1 geth --testnet account list

Use the python script to check data and to send eth between the two node
	
	apt-get install -y python3-pip
	pip3 install --upgrade pip
	pip3 install web3

	python3 checkWeb3.py

To make queries on blockchain, your nodes need to be sync? 
    Syncing?        : No 				<-- you are sync
    Syncing?        : {'highestBlock': 1743086,...  <-- need to wait more...

Use faucet to get eth-test credit: http://www.etherfaucet.net/

Or classic curl: https://github.com/ethereum/wiki/wiki/JSON-RPC

    curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0x0037a6b811ffeb6e072da21179d11b1406371c63", "latest"],"id":1}' http://127.0.0.1:8545

# 5. Geth javascript console

You can attach to the geth console to run javascript command: https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console

    docker exec -it eth_gethtest_1 geth attach ipc:/root/.ethereum/testnet/geth.ipc
    admin
    admin.peers
    eth
    eth.accounts

Or directly:

    docker exec -it eth_gethtest_1 geth --exec 'admin.nodeInfo.name' attach ipc:/root/.ethereum/testnet/geth.ipc
    docker exec -it eth_gethtest_1 geth --exec 'personal.listAccounts' attach ipc:/root/.ethereum/testnet/geth.ipc
    docker exec -it eth_gethtest_1 geth --exec 'eth.getBalance(eth.coinbase)' attach ipc:/root/.ethereum/testnet/geth.ipc

# 6. Continuous delivery

You can fix the version of the nodes in:  

    nano docker-compose.yml 
    image:ethcore/parity:latest   <-- here

Or when a new version of nodes are available in hub.docker.com

    docker-compose pull
    docker-compose up -d  <-- Your containers are at the latest version !
    
# help
    
    docker run -ti ethcore/parity --help
	
