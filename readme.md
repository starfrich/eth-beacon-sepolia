# Setup your own ETH Sepolia & Beacon Sepolia RPC

This guide will help you set up your own Ethereum Sepolia testnet node with both execution layer (Geth) and consensus layer (Prysm) clients. This setup provides you with your own dedicated RPC endpoints for Sepolia testing.

## Hardware Requirements
These are the recommended specifications for running a Sepolia node:
- CPU: 4 Core (minimum)
- RAM: 16 GB (minimum)
- Storage: 1 TB SSD (recommended)
- Internet: Stable connection with at least 10 Mbps

### 🖥️ Recommended VDS & Dedicated Server Providers

If you need to purchase a dedicated server, here are some recommended providers:

1. [FiberState](https://billing.fiberstate.com/aff.php?aff=175)
2. [Servarica](https://clients.servarica.com/aff.php?aff=973)

---

## 1. Installing Dependencies
First, we need to install all the necessary system packages and dependencies:

```bash
# Update system package lists
sudo apt update -qy
```

```bash
# Upgrade all installed packages
sudo apt upgrade -qy
```

```bash
# Install essential tools and dependencies
sudo apt install -y curl wget tar unzip git build-essential golang python3 python3-pip
```

## 2. Installing Geth (Execution Client)
Download and set up the Geth Ethereum client:

```bash
# Download Geth binary
wget https://gethstore.blob.core.windows.net/builds/geth-linux-amd64-1.15.11-36b2371c.tar.gz
```

```bash
# Extract the archive
tar -xvf geth-linux-amd64-1.15.11-36b2371c.tar.gz
```

```bash
# Move the Geth binary to system path
sudo mv geth-linux-amd64-1.15.11-36b2371c/geth /usr/bin/geth
```

## 3. Installing Prysm (Consensus Client)
Download and set up the Prysm Beacon Chain client:

```bash
# Download Prysm Beacon Chain binary
wget https://github.com/OffchainLabs/prysm/releases/download/v6.0.1/beacon-chain-v6.0.1-linux-amd64
```

```bash
# Make the binary executable
chmod +x beacon-chain-v6.0.1-linux-amd64
```

```bash
# Move the Prysm binary to system path
mv beacon-chain-v6.0.1-linux-amd64 /usr/bin/prysm
```

## 4. Setting Up Environment
Create a dedicated user and the necessary directories:

```bash
# Create a dedicated user for running Ethereum nodes
sudo useradd -m -s /bin/bash ethereum
```

```bash
# Create and set permissions for Geth data directory
sudo mkdir -p /data/geth-sepolia && sudo chown -R ethereum:ethereum /data/geth-sepolia
```

```bash
# Create and set permissions for Prysm data directory
sudo mkdir -p /data/prysm && sudo chown -R ethereum:ethereum /data/prysm && sudo chmod -R 755 /data/prysm
```

```bash
# Generate JWT secret for secure communication between execution and consensus clients
openssl rand -hex 32 | sudo tee /data/jwt.hex && sudo chown ethereum:ethereum /data/jwt.hex && sudo chmod 600 /data/jwt.hex
```

## 5. Configuring Systemd Service for Geth
Create a systemd service file for Geth to run it as a background service:

```bash
# Create Geth service file
sudo nano /etc/systemd/system/geth.service
```

Add the following content to the file:

```bash
[Unit]
Description=Geth Sepolia Execution Client
After=network.target
[Service]
User=ethereum
Group=ethereum
Type=simple
Restart=always
RestartSec=5
ExecStart=/usr/bin/geth --sepolia --datadir /data/geth-sepolia \
  --http --http.addr "0.0.0.0" --http.port 8545 \
  --http.api "eth,net,web3,personal,engine" \
  --authrpc.addr "127.0.0.1" --authrpc.port 8551 \
  --authrpc.jwtsecret /data/jwt.hex \
  --http.corsdomain "*" \
  --http.vhosts "*"

[Install]
WantedBy=multi-user.target
```

## 6. Configuring Systemd Service for Prysm
Create a systemd service file for Prysm to run it as a background service:

```bash
# Create Prysm service file
sudo nano /etc/systemd/system/prysm.service
```

Add the following content to the file:

```bash
[Unit]
Description=Prysm Beacon Chain Sepolia
After=network.target geth.service
Requires=geth.service

[Service]
User=ethereum
Group=ethereum
Type=simple
Restart=always
RestartSec=5
ExecStart=/usr/bin/prysm --datadir=/data/prysm --sepolia \
  --execution-endpoint=http://localhost:8551 \
  --rpc-host=0.0.0.0 --rpc-port=4000 \
  --grpc-gateway-host=0.0.0.0 --grpc-gateway-port=3500 \
  --jwt-secret=/data/jwt.hex \
  --genesis-beacon-api-url=https://lodestar-sepolia.chainsafe.io \
  --checkpoint-sync-url=https://sepolia.checkpoint-sync.ethpandaops.io \
  --accept-terms-of-use \
  --http-modules=beacon,config,node,validator

[Install]
WantedBy=multi-user.target
```

## 7. Starting the Services
Now we need to reload systemd, enable and start both services:

```bash
# Reload systemd to recognize the new service files
sudo systemctl daemon-reload
```

```bash
# Enable Geth service to start on boot
sudo systemctl enable geth.service
```

```bash
# Enable Prysm service to start on boot
sudo systemctl enable prysm.service
```

```bash
# Start Geth service
sudo systemctl start geth.service
```

```bash
# Start Prysm service
sudo systemctl start prysm.service
```

## 8. Monitoring the Services
You can monitor the logs of both services to verify they are running correctly:

```bash
# Monitor Geth logs
sudo journalctl -fu geth
```

```bash
# Monitor Prysm logs
sudo journalctl -fu prysm
```

# Check Sync Status GETH
```bash
curl -s -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' \
-H "Content-Type: application/json" http://localhost:8545 | jq
```

Output your GETH is synced:

![Image](https://github.com/user-attachments/assets/20a6eadb-9beb-4d1a-9cf7-751dc3359f54)


# Check Sync Status Prysm
```bash
curl -s http://localhost:3500/eth/v1/node/syncing | jq
```

Output your Prysm is synced:

![Image](https://github.com/user-attachments/assets/672ea81d-f8cc-4734-a27f-f6ab976bad54)


# Notes:
- Fully sync took more than 3 hours on a 1Gbps port speed.
- It might be less if you have a 10Gbps port speed.
- Storage used(11/05/2025) around 627 GB.

## 9. RPC Endpoints
Once both services are running and synced, you can use the following RPC endpoints:

- Execution Layer (Geth) HTTP RPC: http://your-server-ip:8545
- Consensus Layer (Prysm) HTTP API: http://your-server-ip:3500

## 10. Troubleshooting
- If services fail to start, check the logs for errors
- Ensure proper network connectivity
- Verify that the JWT secret is accessible by both clients
- Check that you have sufficient disk space for the blockchain data

## 11. Security Considerations
- Consider setting up a firewall to restrict access to your RPC endpoints
```bash
sudo ufw enable
```

```bash
# Reset UFW ke default
sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH (critical)
sudo ufw allow 22/tcp

# Web (Nginx)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Geth
sudo ufw allow 8545/tcp
sudo ufw allow 30303/tcp
sudo ufw allow 30303/udp

# Prysm
sudo ufw allow 4000/tcp
sudo ufw allow 3500/tcp
sudo ufw allow 13000/tcp
sudo ufw allow 12000/udp

# Enable UFW
sudo ufw enable

# Check UFW Status
sudo ufw status verbose
```

- Use strong passwords for your system
- Keep your software updated regularly
- Consider setting up TLS/SSL for secure connections to your RPC endpoints
> Nginx + Cloudflare Proxied + Certbot

# Discussion

If you encounter any issues trying to setup your nodes, please open issue here: https://github.com/starfrich/eth-beacon-sepolia/issues so we can discuss further.