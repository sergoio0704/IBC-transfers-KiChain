# IBC-transfers-KiChain

### Road map

* [Introduction](#introduction)
* [Installing the Relayer](#installing_the_relayer)
* [Relayer initialization](#relayer_initialization)
* [Creating configuration files](#creating_configuration_files)
* [Light client initialization and working with channels](#light_client)
* [Sending transactions](#sending_transactions)
* [Creating a service file for the Relayer](#creating_service_file_relayer)
* [Launching a Remote Relayer](#launching_remote_relayer)
* [Remote Relayer transaction examples](#remote_relayer_transaction_examples)


## <a name="introduction">Introduction</a>  
Welcome to my guide :smiley:

I would like to tell you how to set up a relayer for cross-broadcasting for 2 networks that are built on Cosmos and use IBC.

Let's take **KiChain** and **Umee** as a basis.

Relayer can be located on the same server as the node, or also located on another server

>In this example, I will be installing the relayer on the same server as the node
***
**To check if your network supports IBC, you need to run the following command**
```javascript
kid q ibc-transfer params #
```
>instead of kid, you can substitute the name of your network

**If the answer looks like this**
```javascript
receive_enabled: 
true send_enabled: true  
```

So you have successfully taken the first step!

## <a name="installing_the_relayer">Installing the Relayer</a>  

1. ```git clone https://github.com/cosmos/relayer.git```
2. ```cd relayer```
3. ```make install```
4. ```cd```
5. ```rly version```

**If you get a reply**
> ```version: 1.0.0-rc1–152-g112205b```

This means the Relayer has been successfully installed.

## <a name="relayer_initialization">Relayer initialization</a>  

> ```rly config init```

## <a name="creating_configuration_files">Creating configuration files</a> 

First, you must create a folder where the network configuration files will be located.

1. ```mkdir rly_config```
2. ```cd rly_config```

**Let's create a file for Kichain**

>```nano kichain-t-4.json```

* **CTRL + O** - save file  
* **CTRL + X** - close file

In the opened editor window, we must insert the following json

```{  "chain-id": "kichain-t-4", "rpc-addr": "https://rpc-challenge.blockchain.ki:443", "account-prefix": "tki", "gas-adjustment": 1.5, "gas-prices": "0.025utki", "trusting-period": "48h" }```

**Now let's create a configuration file for Umee**

>```nano umee-betanet-1.json```

In the opened editor window, we must insert the following json

```{  "chain-id": "umee-betanet-1", "rpc-addr": "http://localhost:26657", "account-prefix": "umee", "gas-adjustment": 1.5, "gas-prices": "0.025uumee", "trusting-period": "48h" }```

**Adding settings to the Relayer configuration**

1. ```rly chains add -f umee-betanet-1.json```
2. ```rly chains add -f kichain-t-4.json```
3. ```cd```

**Create a new wallet**

>```rly keys add umee-betanet-1 YOUR_WALLET_NAME```  
>```rly keys add kichain-t-4 YOUR_WALLET_NAME```

**Restore wallet**

>```rly keys restore umee-betanet-1 YOUR_WALLET_NAME "MNEMONIC PHRASE"```  
>```rly keys restore kichain-t-4 YOUR_WALLET_NAME "MNEMONIC PHRASE"```

**Adding the generated keys to the Relayer configuration**

>```rly chains edit umee-betanet-1 key YOUR_WALLET_NAME```  
>```rly chains edit kichain-t-4 key YOUR_WALLET_NAME```

**Change the wait timeout**

>```nano ~/.relayer/config/config.yaml```

Change the timeout to 10 seconds  
>```timeout: 30s``` -> ```timeout: 10s```

**Both wallets must have coins, check it out**
>```rly q balance umee-betanet-1```  
>```rly q balance kichain-t-4```

If you have coins on your balance, then you can proceed to the next step.

## <a name="light_client">Light client initialization and working with channels files</a> 

**Light client initialization**
>```rly light init umee-betanet-1 -f```  
>```rly light init kichain-t-4 -f```

**Generating a channel between networks**
>```rly paths generate umee-betanet-1 kichain-t-4 transfer --port=transfer```

**Opening a channel for relaying (debug)**
>```rly tx link transfer --debug```

If you see an error

![image](https://user-images.githubusercontent.com/37891709/132264057-114f1789-813e-4e72-9165-198a15e2da1b.png)

You need to go to the config file
>```nano /root/.relayer/config/config.yaml```

And remove the following lines:  
* client-id
* connection-id
* channel-id  

![kichain](https://user-images.githubusercontent.com/37891709/132264224-6c079061-cdc9-46f5-9b22-b5333f6375c6.png)

**Light client reinitialization**
>```rly light init umee-betanet-1 -f```
>```rly light init kichain-t-4 -f```

**Opening the channel (debug)**
>```rly tx link transfer --debug```

We wait until the log appears

![image](https://user-images.githubusercontent.com/37891709/132264435-f287b62b-8952-432a-9d9b-674f8fe7f73d.png)

**Checking**
>```rly paths list -d```

As a result, you should see the following text

![image](https://user-images.githubusercontent.com/37891709/132264483-a16427e9-1225-4b61-b51d-a8a02a7ab6ce.png)

## <a name="sending_transactions">Sending transactions</a> 

Transaction template  
>```rly transact transfer [source-chain] [destination-chain] [amount-token] [destination-wallet-address] [flags]```

Example:
>```rly tx transfer umee-betanet-1 kichain-t-4 1000000uumee tki1v2eghxgmamlkwd4jtsxkev0csgzknhu7lcxwtl --path transfer```


If the transaction was successful, you will see the following text:

![image](https://user-images.githubusercontent.com/37891709/132264808-9c55220d-7cf9-4965-b314-0d96275c9bca.png)

>The details of the transaction can be viewed in the explorer of your network.

## <a name="creating_service_file_relayer">Creating a service file for the Relayer</a> 
```javascript
sudo tee /etc/systemd/system/rlyd.service > /dev/null <<EOF
[Unit]
Description=relayer client
After=network-online.target,starsd.service

[Service]
User=$USER
ExecStart=$(which rly) start transfer
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## <a name="launching_remote_relayer">Launching a Remote Relayer</a> 
1. ```sudo systemctl daemon-reload```
2. ```sudo systemctl enable rlyd```
3. ```sudo systemctl start rlyd```

## <a name="remote_relayer_transaction_examples">Remote Relayer transaction examples</a> 

```kid tx ibc-transfer transfer transfer channel-0 umee16vp4e0cft3052h59zlrfluvmjclaajxt9n4rsj 1000utki --from $kichain_wallet_name --fees=5000utki --gas=auto --chain-id kichain-t-4 --home $HOME/kichain/kid```

* Find out the channel number
> ```rly paths show transfer --yaml```
