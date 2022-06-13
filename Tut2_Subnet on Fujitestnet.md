# Subnet on Fuji Testnet
In this tutorial we will create a subnet on FUji testnet "amafanssubnet". 
You should have a node connected and synced to Fuji testnet. You can run your validator on the same node or on another node also. For the sake of simplicity, I am going to run the validator node on the same fuji node.
Please follow the step by step guide.

##   Download GoLang

```
curl -OL https://golang.org/dl/go1.18.3.linux-amd64.tar.gz=
sudo tar -C /usr/local -xvf go1.18.3.linux-amd64.tar.gz
```
####  Add Go paths in your ~/.bashrc
Add these path variables in your ~/.bashrc file and reload it

```
export GOPATH=$HOME/go
export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin
```
Reload your bashrc for these changes to take effect on the same window.
```
source ~/.bashrc
```
Now, Check go version to see if everything is upto the mark.
```
go version
output: go version go1.18.3 linux/amd64
```

##   Install subnet-cli

```
git clone https://github.com/ava-labs/subnet-cli.git;
cd subnet-cli;
go install -v .;
```

##  Create your validator keys 

Use this command to create your private key and fund it using fuji faucet.

I created a new directoty "keys" under my home directory and then ran this command to generate a private key.
```
cd keys
subnet-cli create key
```
A new file .subnet-cli.pk will be created with a private key. Copy the private key.
By default, subnet-cli uses the key specified in file .subnet-cli.pk on the P-Chain to pay for the transaction fee,
If you want to use some other key at some other location on this server, use --private-key-path later to overwrite. Please make sure that you have enough fund on this P-Chain address to pay for transactions.

####  To fund this private key on Fuji TestNet P-chain. 

##### Address corresponding to this private key 
Use your private key in the .subnet-cli.pk file on the web wallet to access this https://wallet.avax.network/. (Private Key is the first option on the web wallet). And pick Fuji on the top right corner as the network and locate your C-Chain address which starts with 0x. Copy this address.
##### Request funds from fujitestnet faucet.
Request funds from the faucet using your C-Chain address at https://faucet.avax.network/. Wait for the transaction to get confirmed.

##### Move funds from C-chain to P-chain
Move the test funds from the C-Chain to the P-Chain by clicking on the Cross Chain on the left side of the https://wallet.avax.network/
![C-chain to P-chain](/assets/ChaintoPchain.png)

After following these 3 steps, your test key should now have a balance on the P-Chain on Fuji Testnet.

##  Create VMID
First, you'll need to compile the subnet-evm into a binary that AvalancheGo can interact with.
```
git clone https://github.com/ava-labs/subnet-evm.git
cd subnet-evm/
subnet-cli create VMID amafanssubnetevm
```
Create a VMID with string `amafanssubnetevm` which you can change to whatever you like. This command is used to generate a valid VMID based on some string to uniquely identify a VM. This should stay the same for all versions of the VM, so it should be based on a word rather than the hash of some code.
Output will be:
```
created a new VMID juedWvBi73uAL3JgXYPJsD6d2YyVPAg4xv4RRqdCzaWpTsjVj from amafanssubnetevm
```
Now issue this command to build
```
./scripts/build.sh build/juedWvBi73uAL3JgXYPJsD6d2YyVPAg4xv4RRqdCzaWpTsjVj
```
This command will create a binary in the build folder
```
ubuntu@ip-172-31-90-23:~/subnet-evm$ ls build/
juedWvBi73uAL3JgXYPJsD6d2YyVPAg4xv4RRqdCzaWpTsjVj
```

## Move Binary

Choose option based on how you had built your "avalanchego".
##### Build from source manually
Once the `subnet-evm` binary is built, you'll need to move it to AvalancheGo's plugin directory (within the [--build-dir](https://docs.avax.network/nodes/maintain/avalanchego-config-flags#--build-dir-string)) 
so it can be run by your node. When building avalanchego from source (see  [Run an Avalanche Node Manually ](https://docs.avax.network/nodes/build/run-avalanche-node-manually)), 
this defaults to avalanchego/build/plugins in which avalanchego is the directory where you have checked out AvalancheGo project. This build directory is structured as:

build-dir
|_avalanchego (note: this is the AvalancheGo binary, not a directory)
|_plugins
  |_evm

To put the subnet-evm binary in the right place, run the following command (assuming the avalanchego and subnet-evm repos are in the same folder):

mv ./subnet-evm/build/juedWvBi73uAL3JgXYPJsD6d2YyVPAg4xv4RRqdCzaWpTsjVj ./avalanchego/build/plugins;


#### using the Install Script 

avalanche-node
|_avalanchego (note: this is the AvalancheGo binary, not a directory)
|_plugins
  |_evm

mv subnet-evm/build/juedWvBi73uAL3JgXYPJsD6d2YyVPAg4xv4RRqdCzaWpTsjVj avalanche-node/plugins/


## Genesis file
Create a Custom Subnet Configuration
```
avalanche subnet create amafanssubnet
```
It will present you several options, Let us explore them one by one.
1. Choose EVM, For our case, use SubnetEVM
```
ubuntu@ip-172-31-90-23:~$ avalanche subnet create amafanssubnet
Use the arrow keys to navigate: ↓ ↑ → ←
? Choose your VM:
  ▸ SubnetEVM
    Custom
```
2. Choose chainID, 
Choose chainid which is not being used by any existing EVM chains 
Check here: https://chainlist.org/
```
Enter your subnet's ChainId. It can be any positive integer.
✗ ChainId: █
```

3. Choose Fee
```
? How would you like to set fees:
    Low disk use    / Low Throughput    1.5 mil gas/s (C-Chain's setting)
  ▸ Medium disk use / Medium Throughput 2 mil   gas/s
    High disk use   / High Throughput   5 mil   gas/s
    Customize fee config
```

4. Airdrop Tokens in the beginning of the chain
```
? How would you like to distribute funds:
    Airdrop 1 million tokens to the default address (do not use in production)
  ▸ Customize your airdrop
```
Enter your address and the tokens amount.
If you want to add more airdrop, Keep clicking on Yes
```
? Would you like to airdrop more tokens?:
  ▸ No
    Yes
```
5. Adding a custom precompile
We will revist this option in later tutorials, Lets continue with "No" option.
```
? Advanced: Would you like to add a custom precompile to modify the EVM?:
  ▸ No
    Yes

```
Click on No and your subnet will be created.
##### Check your subnet 
```
ubuntu@ip-172-31-90-23:~$ avalanche subnet list

+---------------+---------------+-----------+
|    SUBNET     |     CHAIN     |   TYPE    |
+---------------+---------------+-----------+
| amafanssubnet | amafanssubnet | SubnetEVM |
+---------------+---------------+-----------+
```

##### Setting a Custom Fee Recipient
By default, all fees are burned (sent to the blackhole address with "allowFeeRecipients": false). However, 
it is possible to enable block producers to set a fee recipient (who will get compensated for blocks they produce).
To enable this feature, you'll need to add the following to your genesis file (under the "config" key):
```
{
  "config": {
    "allowFeeRecipients": true
  }
}
```
Fee Recipient
With allowFeeRecipients enabled, your validators can specify their addresses to collect fees. 
They need to update their EVM chain config with the following to specify where the fee should be sent to.
```
{
  "feeRecipient": "<YOUR 0x-ADDRESS>"
}
```

##### Get your genesis file for the subnet 
```
avalanche subnet describe firstsubnet --genesis
{
    "config": {
        "chainId": 13665,
        "homesteadBlock": 0,
        "eip150Block": 0,
        "eip150Hash": "0x2086799aeebeae135c246c65021c82b4e15a2c451340993aacfd2751886514f0",
        "eip155Block": 0,
        "eip158Block": 0,
        "byzantiumBlock": 0,
        "constantinopleBlock": 0,
        "petersburgBlock": 0,
        "istanbulBlock": 0,
        "muirGlacierBlock": 0,
        "subnetEVMTimestamp": 0,
        "feeConfig": {
            "gasLimit": 8000000,
            "targetBlockRate": 2,
            "minBaseFee": 25000000000,
            "targetGas": 20000000,
            "baseFeeChangeDenominator": 36,
            "minBlockGasCost": 0,
            "maxBlockGasCost": 1000000,
            "blockGasCostStep": 200000
        },
        "contractDeployerAllowListConfig": {
            "blockTimestamp": null,
            "adminAddresses": null
        },
        "contractNativeMinterConfig": {
            "blockTimestamp": null,
            "adminAddresses": null
        },
        "txAllowListConfig": {
            "blockTimestamp": null,
            "adminAddresses": null
        }
    },
    "nonce": "0x0",
    "timestamp": "0x0",
    "extraData": "0x",
    "gasLimit": "0x7a1200",
    "difficulty": "0x0",
    "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "coinbase": "0x0000000000000000000000000000000000000000",
    "alloc": {
        "3213ef068d8def86a068d0cbbcbb3c00664894ad": {
            "balance": "0x8ac7230489e80000"
        },
        "9c4c630fab38192bbcc53464a156a59337d75a24": {
            "balance": "0x5af3107a4000"
        }
    },
    "airdropHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "airdropAmount": null,
    "number": "0x0",
    "gasUsed": "0x0",
    "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "baseFeePerGas": null
}
```

save this file in your home directory with 
amafanssubnet_genesis.json

## Run your subnet EVM

Step 1: 
You must have a fuji node up and running, Refer to this artcile to deploy a fuji node on AWS.
Get your nodeID by this command
```
ubuntu@ip-172-31-90-23:~$ curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"info.getNodeID"
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info

ubuntu@ip-172-31-90-23:~$ {"jsonrpc":"2.0","result":{"nodeID":"NodeID-167TkRiC6RVLboo5U7TpiB6Sk4UfxN8w2"},"id":1}
```
Make note of this node-id

Step 2: 

```
subnet-cli wizard \
--node-ids=NodeID-167TkRiC6RVLboo5U7TpiB6Sk4UfxN8w2 \
--vm-genesis-path=amafanssubnet_genesis.json \
--vm-id=juedWvBi73uAL3JgXYPJsD6d2YyVPAg4xv4RRqdCzaWpTsjVj \
--chain-name=amafanssubnet \
--private-key-path keys/subnet-cli.pk
```

If you see this error 
```
subnet-cli failed insufficient funds: on [P-fuji1j38ncp5r6dpfjlt64et8cnqvcnp4yum4yfxyz6] (expected=1201000000, have=0)
```
Your address that correponds to the private key in .subnet-cli.pk doesnt have sufficient funds. Please refer to the section
"Create Keys" above to fund your address.

If everything goes fine, You will see this output

```
subnet-cli wizard --node-ids=NodeID-167TkRiC6RVLboo5U7TpiB6Sk4UfxN8w2\
         --vm-genesis-path=amafanssubnet_genesis.json \
         --vm-id=juedWvBi73uAL3JgXYPJsD6d2YyVPAg4xv4RRqdCzaWpTsjVj \
          --chain-name=amafanssubnet \
          --private-key-path keys/subnet-cli.pk

2022-06-13T18:20:33.498Z        info    client/client.go:81     fetching X-Chain id
2022-06-13T18:20:33.543Z        info    client/client.go:87     fetched X-Chain id      {"id": "2JVSBoinj9C2J33VntvzYtVJNZdN2NKiwwKjcumHUWEb5DbBrm"}
2022-06-13T18:20:33.543Z        info    client/client.go:96     fetching AVAX asset id  {"uri": "https://api.avax-test.network"}
2022-06-13T18:20:33.564Z        info    client/client.go:105    fetched AVAX asset id   {"id": "U8iRqJoiJm8xZHAacmvYyZVwqQx6uDNtQeP3CQ6fcgQk3JqnK"}
2022-06-13T18:20:33.564Z        info    client/client.go:107    fetching network information
2022-06-13T18:20:33.590Z        info    client/client.go:116    fetched network information     {"networkId": 5, "networkName": "fuji"}

Ready to run wizard, should we continue?
*--------------------------------*---------------------------------------------------*
| PRIMARY P-CHAIN ADDRESS        | P-fuji1j38ncp5r6dpfjlt64et8cnqvcnp4yum4yfxyz6     |
*--------------------------------*---------------------------------------------------*
| TOTAL P-CHAIN BALANCE          | 1.9952981 $AVAX                                   |
*--------------------------------*---------------------------------------------------*
| TX FEE                         | 0.201 $AVAX                                       |
*--------------------------------*---------------------------------------------------*
| EACH STAKE AMOUNT              | 1.000 $AVAX                                       |
*--------------------------------*---------------------------------------------------*
| TOTAL STAKE AMOUNT             | 1.000 $AVAX                                       |
*--------------------------------*---------------------------------------------------*
| REQUIRED BALANCE               | 1.201 $AVAX                                       |
*--------------------------------*---------------------------------------------------*
| URI                            | https://api.avax-test.network                     |
*--------------------------------*---------------------------------------------------*
| NETWORK NAME                   | fuji                                              |
*--------------------------------*---------------------------------------------------*
| NEW PRIMARY NETWORK VALIDATORS | [167TkRiC6RVLboo5U7TpiB6Sk4UfxN8w2]               |
*--------------------------------*---------------------------------------------------*
| VALIDATE END                   | 2023-04-09T18:20:33Z                              |
*--------------------------------*---------------------------------------------------*
| STAKE AMOUNT                   | 1.000 $AVAX                                       |
*--------------------------------*---------------------------------------------------*
| VALIDATE REWARD FEE            | 2.000 %                                           |
*--------------------------------*---------------------------------------------------*
| REWARD ADDRESS                 | EXBvhXRVVJhehvXTz7BJvzUtsxsWMrcdy                 |
*--------------------------------*---------------------------------------------------*
| CHANGE ADDRESS                 | EXBvhXRVVJhehvXTz7BJvzUtsxsWMrcdy                 |
*--------------------------------*---------------------------------------------------*
| NEW SUBNET VALIDATORS          | [167TkRiC6RVLboo5U7TpiB6Sk4UfxN8w2]               |
*--------------------------------*---------------------------------------------------*
| SUBNET VALIDATION WEIGHT       | 1,000                                             |
*--------------------------------*---------------------------------------------------*
| CHAIN NAME                     | amafanssubnet                                     |
*--------------------------------*---------------------------------------------------*
| VM ID                          | juedWvBi73uAL3JgXYPJsD6d2YyVPAg4xv4RRqdCzaWpTsjVj |
*--------------------------------*---------------------------------------------------*
| VM GENESIS PATH                | amafanssubnet_genesis.json                        |
*--------------------------------*---------------------------------------------------*
Use the arrow keys to navigate: ↓ ↑ → ←
  ▸ Yes, let's create! I agree to pay the fee!
    No, stop it!
```
Click on Yes. and you will see the following output

```

2022-06-13T18:21:03.709Z        info    client/p.go:405 adding validator        {"start": "2022-06-13T18:21:33.678Z", "end": "2023-04-09T18:20:33.000Z", "stakeAmount": 1000000000, "rewardAddress": "EXBvhXRVVJhehvXTz7BJvzUtsxsWMrcdy", "changeAddress": "EXBvhXRVVJhehvXTz7BJvzUtsxsWMrcdy"}
2022-06-13T18:21:03.812Z        info    platformvm/checker.go:48        polling P-Chain tx      {"txId": "4MQfGNN5o7HTTZL562P6NxKoSfVKGiodbh2ehm7oqZ2oavemB", "expectedStatus": "Committed"}
2022-06-13T18:21:03.812Z        info    poll/poll.go:42 start polling   {"internal": "1s"}
2022-06-13T18:21:05.948Z        info    poll/poll.go:66 poll confirmed  {"took": "2.136140586s"}
added 167TkRiC6RVLboo5U7TpiB6Sk4UfxN8w2 to primary network validator set (took 2.136140586s)

waiting for validator 167TkRiC6RVLboo5U7TpiB6Sk4UfxN8w2 to start validating 11111111111111111111111111111111LpoYY...(could take a few minutes)


2022-06-13T18:23:36.737Z        info    client/p.go:131 creating subnet {"dryMode": false, "assetId": "U8iRqJoiJm8xZHAacmvYyZVwqQx6uDNtQeP3CQ6fcgQk3JqnK", "createSubnetTxFee": 100000000}
2022-06-13T18:23:36.850Z        info    platformvm/checker.go:74        polling subnet  {"subnetId": "QWNhddjn6AcebaE8QRwpkL9tgiRmmdFxfHBuWqoKN5pHRWRKV"}
2022-06-13T18:23:36.850Z        info    platformvm/checker.go:48        polling P-Chain tx      {"txId": "QWNhddjn6AcebaE8QRwpkL9tgiRmmdFxfHBuWqoKN5pHRWRKV", "expectedStatus": "Committed"}
2022-06-13T18:23:36.850Z        info    poll/poll.go:42 start polling   {"internal": "1s"}
2022-06-13T18:23:38.966Z        info    poll/poll.go:66 poll confirmed  {"took": "2.11540627s"}
2022-06-13T18:23:38.966Z        info    platformvm/checker.go:88        finding subnets {"subnetId": "QWNhddjn6AcebaE8QRwpkL9tgiRmmdFxfHBuWqoKN5pHRWRKV"}
2022-06-13T18:23:38.966Z        info    poll/poll.go:42 start polling   {"internal": "1s"}
2022-06-13T18:23:39.028Z        info    poll/poll.go:66 poll confirmed  {"took": "62.183781ms"}
created subnet "QWNhddjn6AcebaE8QRwpkL9tgiRmmdFxfHBuWqoKN5pHRWRKV" (took 2.177590051s)



Now, time for some config changes on your node(s).
Set --whitelisted-subnets=QWNhddjn6AcebaE8QRwpkL9tgiRmmdFxfHBuWqoKN5pHRWRKV and move the compiled VM juedWvBi73uAL3JgXYPJsD6d2YyVPAg4xv4RRqdCzaWpTsjVj to <build-dir>/plugins/juedWvBi73uAL3JgXYPJsD6d2YyVPAg4xv4RRqdCzaWpTsjVj.
When you're finished, restart your node.
Use the arrow keys to navigate: ↓ ↑ → ←
  ▸ Yes, let's continue! I've updated --whitelisted-subnets, built my VM, and restarted my node(s)!
```

 {"subnetId": "QWNhddjn6AcebaE8QRwpkL9tgiRmmdFxfHBuWqoKN5pHRWRKV"}

 ##### Adding your subnetid to your Fuji code config.json

open config.json present at  ~/.avalanchego/configs/node.json on your fuji node
and add following details, Replace whitelisted-subnets with the subnet Id  from above.
```
 {
    "http-host": "",
    "network-id": "fuji",
    "public-ip": "44.205.74.92",
    "health-check-frequency": "2s",
    "log-display-level": "DEBUG",
    "log-level": "DEBUG",
    "whitelisted-subnets": "QWNhddjn6AcebaE8QRwpkL9tgiRmmdFxfHBuWqoKN5pHRWRKV"

}
```
Restart your avalanchego service.

##### Your subnet details and Blockchain details.
Go back to your console where you ran the subnet wizard command.
You will see the following lines after continutation 

```
2022-06-13T18:39:03.126Z        info    client/p.go:497 creating blockchain     {"subnetId": "QWNhddjn6AcebaE8QRwpkL9tgiRmmdFxfHBuWqoKN5pHRWRKV", "chainName": "amafanssubnet", "vmId": "juedWvBi73uAL3JgXYPJsD6d2YyVPAg4xv4RRqdCzaWpTsjVj", "createBlockchainTxFee": 100000000}
created blockchain "j6TJTXGmMWWBvYHsyp1bZrkiSDJVoG4FpaWz4LrTjSjpE6zxN" (took 118.540399ms)

*----------------------------*---------------------------------------------------*
| PRIMARY P-CHAIN ADDRESS    | P-fuji1j38ncp5r6dpfjlt64et8cnqvcnp4yum4yfxyz6     |
*----------------------------*---------------------------------------------------*
| TOTAL P-CHAIN BALANCE      | 0.8942981 $AVAX                                   |
*----------------------------*---------------------------------------------------*
| URI                        | https://api.avax-test.network                     |
*----------------------------*---------------------------------------------------*
| NETWORK NAME               | fuji                                              |
*----------------------------*---------------------------------------------------*
| PRIMARY NETWORK VALIDATORS | [167TkRiC6RVLboo5U7TpiB6Sk4UfxN8w2]               |
*----------------------------*---------------------------------------------------*
| SUBNET VALIDATORS          | [167TkRiC6RVLboo5U7TpiB6Sk4UfxN8w2]               |
*----------------------------*---------------------------------------------------*
| SUBNET ID                  | QWNhddjn6AcebaE8QRwpkL9tgiRmmdFxfHBuWqoKN5pHRWRKV |
*----------------------------*---------------------------------------------------*
| BLOCKCHAIN ID              | j6TJTXGmMWWBvYHsyp1bZrkiSDJVoG4FpaWz4LrTjSjpE6zxN |
*----------------------------*---------------------------------------------------*
| CHAIN NAME                 | amafanssubnet                                     |
*----------------------------*---------------------------------------------------*
| VM ID                      | juedWvBi73uAL3JgXYPJsD6d2YyVPAg4xv4RRqdCzaWpTsjVj |
*----------------------------*---------------------------------------------------*
| VM GENESIS PATH            | amafanssubnet_genesis.json                        |
*----------------------------*---------------------------------------------------*
```
This shows that your subnet QWNhddjn6AcebaE8QRwpkL9tgiRmmdFxfHBuWqoKN5pHRWRKV
and Blockchain id is j6TJTXGmMWWBvYHsyp1bZrkiSDJVoG4FpaWz4LrTjSjpE6zxN

you can check this url https://explorer-xp.avax-test.network/blockchain/j6TJTXGmMWWBvYHsyp1bZrkiSDJVoG4FpaWz4LrTjSjpE6zxN 
to see the blockchain and subnet.

 ![Subnet details](/assets/subnetdetails.png)


## Make your rpc url available
Install nginx for proxy forward port 80 to  port 9650
##### Install nginx
sudo apt-get install nginx

##### Edit /etc/nginx/site-enabled/default and put this config.
```
server {

        etag off;
        ssl_protocols TLSv1.2;
        ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256';
        ssl_prefer_server_ciphers on;

        listen 80;
        location / {
                proxy_pass   http://localhost:9650/;
                real_ip_header CF-Connecting-IP;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                add_header X-Frame-Options 'deny' always;
                add_header X-Content-Type-Options "nosniff";
                add_header X-XSS-Protection '1; mode=block' always;
                add_header Cache-Control 'must-revalidate';
                add_header Strict-Transport-Security "max-age=31536000; includeSubdomains; preload";

                proxy_redirect off;

        #limit_except GET POST PUT { deny all; }

        }
        location /healthcheck {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                #add_header Access-Control-Allow-Origin *;
                return 200;
        }

        location /home/ubuntu/ {
                deny all;
                return 403;
        }

}

```

##### Open port 80 on the security group of your validator ec2 instance.

##### Access the RPC
Now access your RPC at  http://<your validator elastic ip>/ext/bc/j6TJTXGmMWWBvYHsyp1bZrkiSDJVoG4FpaWz4LrTjSjpE6zxN/rpc

