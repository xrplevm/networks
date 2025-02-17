# Launch Ceremony

The following guide outlines the necessary steps for the Validator Advisory Board to launch the XRPL EVM testnet.

## Validator provisioning

### 1. Infrastructure Provision
Provision a new machine that meets the minimum specifications as outlined in the [XRPL EVM documentation](https://docs.xrplevm.org/pages/operators/getting-started/system-requirements). Ensure the machine has stable network connectivity and sufficient resources to support the node.

### 2. Node Installation
Download and install the XRPL EVM node binary by following the official installation guide: [Installing the Node](https://docs.xrplevm.org/pages/operators/getting-started/installing-the-node).

### 3. Configure the Node
Set up the node configuration with the correct chain ID:
```sh
exrpd config set client chain-id xrplevm_1449000-1
```

Select a secure keyring backend (either `file` or `os`) and configure the node accordingly:
```sh
exrpd config set client keyring-backend <keyring>
```

Configure additional settings based on best practices:

- **Pruning**: Set to default for optimal storage efficiency.

- **API Access**: Disable all APIs except Tendermint RPC, which should be restricted to localhost.

- **Peer Exchange**: Maintain default settings unless specific network configurations are required.

For more details, refer to the [node configuration documentation](https://docs.xrplevm.org/pages/operators/advanced/node-configuration-options) as well as the [node configuration reference](https://docs.xrplevm.org/pages/operators/resources/configuration-reference).


### 4. Create a New Key and Secure It
Generate a new key for the validator.
```sh
exrpd keys add <key_name> --key-type eth_secp256k1
```
**Ensure that you back up the mnemonic securely**. [Read more](https://docs.xrplevm.org/pages/operators/validators/managing-keys).


### 5. Initialize the Node
Initialize the node with a moniker (a unique name for your validator):
```sh
exrpd init <moniker> --chain-id xrplevm_1449000-1
```

**Ensure that you back up the node keys securely**. [Read more](https://docs.xrplevm.org/pages/operators/validators/managing-keys).
```sh
~/.exrpd/config/node_key.json
~/.exrpd/config/priv_validator_key.json
```

### 6. Download the Base Genesis File
Download the base genesis file required for initialization:
```sh
wget -O ~/.exrpd/config/genesis.json https://raw.githubusercontent.com/xrplevm/networks/refs/heads/main/testnet/base-genesis.json
```

### 7. Add Your Validator Account to the Genesis
Add your validator account to the genesis block:
```sh
exrpd add-genesis-account "$(exrpd keys show <key_name> -a)" 1000000poa,40000000000000000axrp
```

Generate the validator transaction:
```sh
exrpd gentx <key_name> 1000000poa --fees 40000000000000000axrp --chain-id xrplevm_1449000-1 --commission-rate 0 --commission-max-rate 0 --commission-max-change-rate 0
```

Collect all genesis transactions:
```sh
exrpd collect-gentxs
```

### 8. Submit the Genesis File
Send the generated genesis file located at `~/.exrpd/config/genesis.json` to **#advisory-board** Discord channel for finalization and deployment.

---

After following these steps, we will collect the genesis files from all the members of the Validator Advisory Board. Once all the files are collected, we will generate the final genesis file that will be used for the final deployment and launch of the XRPL EVM testnet.

---

##  Launching the Testnet

### 1. Download final genesis File
Download the final genesis file of the Testnet:
```sh
wget -O ~/.exrpd/config/genesis.json https://raw.githubusercontent.com/xrplevm/networks/refs/heads/main/testnet/genesis.json
```

### 2. Add Seeds to the Node Configuration
Add seeds to the node configuration that will allow connecting to the rest of the node. Modify the file `~/.exrpd/config/config.toml` to include the following seed node:
```sh
PEERS=`curl -sL https://raw.githubusercontent.com/xrplevm/networks/main/testnet/peers.txt | sort -R | head -n 10 | awk '{print $1}' | paste -s -d, -`
sed -i.bak -e "s/^seeds *=.*/seeds = \"$PEERS\"/" ~/.exrpd/config/config.toml
cat ~/.exrpd/config/config.toml | grep seeds
```

### 3. Initialize the Node
Initialize the node with the final genesis file:
```sh
exrpd start
```

### 4. Testnet start
Monitor the node status and ensure that it is syncing with the network. Once the node connects to the majority of the validators, the network will start producing blocks. 
