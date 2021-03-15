# IPFS note

## Ref

3. https://medium.com/swlh/ipfs-nodejs-app-8e35f504d440
5. https://github.com/ipfs/js-ipfs
6. https://github.com/ipfs/js-ipfs/tree/master/packages/ipfs-http-client
7. https://github.com/ipfs/js-ipfs/blob/master/docs/core-api/FILES.md#ipfsgetipfspath-options

## IPFS Quick Start

1. https://docs.ipfs.io/how-to/command-line-quick-start/

```sh
# mac
brew install ipfs
ipfs --version
ipfs init
ipfs cat /ipfs/QmQPeNsJPyVWPFDVHb77w8G42Fvo15z4bG2X8D2GhfbSXc/readme
ipfs cat /ipfs/QmQPeNsJPyVWPFDVHb77w8G42Fvo15z4bG2X8D2GhfbSXc/quick-start
```

```sh
# file
echo "hello world" > hello
ipfs add hello
ipfs cat <the-hash-you-got-here>
 
# dir
mkdir foo
mkdir foo/bar
echo "baz" > foo/baz
echo "baz" > foo/bar/baz
ipfs add -r foo

```

```sh
# online
ipfs daemon
ipfs swarm peers
# WebUI
http://127.0.0.1:5001/webui
# Gateway
http://127.0.0.1:8080/ipfs/<the-hash-you-got-here>
```

## IPFS Private Network

1. https://medium.com/@s_van_laar/deploy-a-private-ipfs-network-on-ubuntu-in-5-steps-5aad95f7261b
2. https://stackoverflow.com/questions/40180171/how-to-run-several-ipfs-nodes-on-a-single-machine
3. https://tstrs.me/1459.html
4. https://chunyu-hsiao93.medium.com/ipfs-private-network-私有網路-8d8748cba7b2
5. https://github.com/ipfs/go-ipfs/blob/master/docs/experimental-features.md#private-networks

```sh
mkdir ipfs
cd ipfs

# init
IPFS_PATH=./node1 ipfs init 
IPFS_PATH=./node2 ipfs init
IPFS_PATH=./node1 ipfs bootstrap rm --all
IPFS_PATH=./node2 ipfs bootstrap rm --all

# swarm key
go get -u github.com/Kubuxu/go-ipfs-swarm-key-gen/ipfs-swarm-key-gen
ipfs-swarm-key-gen > ./swarm.key
cp swarm.key ./node1
cp swarm.key ./node2

# change api port / gateway port
sed -i '' 's/5001/6001/g' ./node1/config 
sed -i '' 's/5001/7001/g' ./node2/config 
sed -i '' 's/8080/8081/g' ./node1/config 
sed -i '' 's/8080/8082/g' ./node2/config 

# grep ID
ID1=$(IPFS_PATH=./node1 ipfs id | grep ID | awk '$1=$1' | awk '{split($0, a, "\"")}{print a[4]}') 
ID2=$(IPFS_PATH=./node2 ipfs id | grep ID | awk '$1=$1' | awk '{split($0, a, "\"")}{print a[4]}') 

# node1, node2 add each other
IPFS_PATH=./node1 ipfs bootstrap add /ip4/127.0.0.1/tcp/4001/ipfs/${ID2}
IPFS_PATH=./node2 ipfs bootstrap add /ip4/127.0.0.1/tcp/4001/ipfs/${ID1}

#  start network
IPFS_PATH=./node1 ipfs daemon &
IPFS_PATH=./node2 ipfs daemon &

#  check partners [1] -> success
IPFS_PATH=./node1 ipfs stats bitswap

# kill process
killall ipfs
```

