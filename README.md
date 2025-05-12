# Aztec-sequencer-node
Step by step guide on how to setup/run an aztec sequencer node
# Step-by-Step Guide to Running an AZTEC Node on VPS

This guide will walk you through setting up an AZTEC sequencer node on your VPS running Ubuntu.

## Prerequisites
- A VPS running Ubuntu (preferably Ubuntu 24.04)
- At least 8 CPU cores, 16GB RAM, and 1TB SSD storage
- A stable internet connection (25 Mbps up/down minimum)
- Some Sepolia ETH (for transaction fees)

## Step 1: Connect to Your VPS
```bash
ssh your_username@your_vps_ip
```

## Step 2: Update Your System
```bash
sudo apt update -y && sudo apt upgrade -y
```

## Step 3: Install Docker

```bash
# Remove old versions if they exist
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```


```bash
# Set up the repository
sudo apt-get update
```
```bash
sudo apt-get install -y ca-certificates curl gnupg
```
```bash
sudo install -m 0755 -d /etc/apt/keyrings
```
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
```bash
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
```bash
# Add Docker repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```bash
# Install Docker
sudo apt-get update
```
```bash
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
```bash
# Test Docker installation
sudo docker run hello-world
```
```bash
# Start and enable Docker service
sudo systemctl start docker
```
```bash
sudo systemctl enable docker
```

## Step 4: Install AZTEC CLI

```bash
# Install AZTEC CLI
bash -i <(curl -s https://install.aztec.network)
```
(Press y, then Enter)
```bash
# Add AZTEC to PATH
echo 'export PATH=$PATH:$HOME/.aztec/bin' >> ~/.bashrc
source ~/.bashrc
```
```bash
# Update to testnet version
aztec-up alpha-testnet
```
```bash
# Verify installation
aztec --version
```

## Step 5: Prepare Your Environment Variables

Create a file to store your environment variables:

```bash
nano ~/.aztec-env
```

Add the following content (replace with your actual values):

```bash
export ETHEREUM_HOSTS="https://ethereum-sepolia-rpc.publicnode.com"  # RPC Endpoints Ethereum sepolia
export L1_CONSENSUS_HOST_URLS="https://ethereum-sepolia-beacon-api.publicnode.com"  # RPC Endpoints Ethereum sepolia beacon
export VALIDATOR_PRIVATE_KEY="your_private_key_here" # Your Ethereum wallet private key
export COINBASE="0xYourEthereumAddress"  # Your address to receive rewards
export P2P_IP="$(curl -s api.ipify.org)"  # Your VPS IP address
```

Save and exit (CTRL+X, then Y, then Enter).

Load these variables:

```bash
source ~/.aztec-env
```

## Step 6: Create a Data Directory

```bash
mkdir -p ~/aztec-data
```

## Step 7: Start Your Sequencer Node

```bash
aztec start --node --archiver --sequencer \
  --network alpha-testnet \
  --l1-rpc-urls $ETHEREUM_HOSTS \
  --l1-consensus-host-urls $L1_CONSENSUS_HOST_URLS \
  --sequencer.validatorPrivateKey $VALIDATOR_PRIVATE_KEY \
  --sequencer.coinbase $COINBASE \
  --p2p.p2pIp $P2P_IP \
  --p2p.maxTxPoolSize 1000000000 \
  --data-directory ~/aztec-data
```

## Step 8: Run the Node in Background (Optional)

If you want to keep the node running after closing your SSH session, you can use a tool like `tmux` or `screen`:

```bash
# Install tmux
sudo apt install -y tmux
```
```bash
# Create a new tmux session
tmux new -s aztec-node
```
```bash
# Now run your node command in this session
# (The same command from Step 7)

# To detach from the session, press: CTRL+B, then D
# To reattach later, use: tmux attach -t aztec-node
```

## Step 9: Register as a Validator

Once your node is fully synced, register as a validator:

```bash
aztec add-l1-validator \
  --l1-rpc-urls $ETHEREUM_HOSTS \
  --private-key $VALIDATOR_PRIVATE_KEY \
  --attester $COINBASE \
  --proposer-eoa $COINBASE \
  --staking-asset-handler 0xF739D03e98e23A7B65940848aBA8921fF3bAc4b2 \
  --l1-chain-id 11155111
```

Note: There is a daily quota for validators. If registration fails with `ValidatorQuotaFilledUntil`, try again after the timestamp indicated.

## Step 10: Monitor Your Node

Check your node's logs, if running directly the logs will be displayed in your terminal
```bash
# If running in tmux:
tmux attach -t aztec-node
```
```bash
# To view the logs if you're using data-directory:
ls -la ~/aztec-data
# Look for log files in this directory
```

## Troubleshooting

### If Your Node Fails to Sync

```bash
# Stop the node process
# Delete the current snapshot
rm -rf ~/.aztec/alpha-testnet/data/archiver
```
```bash
# Update to the latest version
aztec-up alpha-testnet

# Restart your node
```

### Checking Your Public IP

```bash
curl api.ipify.org
```

### Port Forwarding

Ensure port 40400 (both TCP and UDP) is open on your VPS firewall:

```bash
# Check if the port is already open
sudo ufw status
```
```bash
# If not, open it
sudo ufw allow 40400/tcp
sudo ufw allow 40400/udp
```

### RPC Rate Limits

If you encounter RPC rate limit issues, consider:
1. Using your own Ethereum node
2. Signing up for an API key with a provider like Alchemy or Infura

## Additional Resources

- Join the AZTEC Discord for community support
- Check the Aztec documentation for updates
- Use `aztec help start` for detailed command options
