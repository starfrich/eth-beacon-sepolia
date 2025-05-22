# Setup your own ETH Sepolia & Beacon Sepolia RPC

## 🚨 IMPORTANT NOTICE
> **⚠️ WARNING: THIS IS NOT A COPY-PASTE GUIDE - PLEASE READ CAREFULLY!**

This guide will help you set up your own Ethereum Sepolia testnet node with both execution layer (Geth) and consensus layer (Prysm) clients. This setup provides you with your own dedicated RPC endpoints for Sepolia(ETH & Beacon).

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

## RPC

Most free RPC services have limitations but don’t worry, we’ve got you covered. We have few packages for rent starts from $15/week, DM me on Telegram: [starfish](https://t.me/starfishprerich) or [robapuros](https://t.me/Robapuros) and we’ll work something out.
 
> Current availability: **2 IPs left**

> Once the all slots are filled, you’ll need to wait **3–7 days** for the next available RPC. 

> 📌 **First Come, First Served** — no booking system. 


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
sudo apt install -y curl wget tar unzip git build-essential golang python3 python3-pip nginx certbot python3-certbot-nginx
```


**Nginx:**
```bash
# Start and enable Nginx

sudo systemctl start nginx
sudo systemctl enable nginx

# Check status
sudo systemctl status nginx
```

### 2. Setup Ethereum Node

**Directory**
```bash
mkdir rpc && cd rpc
```

```
# create docker-compose.yml
nano docker-compose.yml
```
> Copy from docker-compose.yml from this repo and save.

**Create the JWT secret file:**
```bash
mkdir -p data
openssl rand -hex 32 > data/jwt.hex
```

**Start the services:**
```bash
docker compose up -d
```

**Monitor the logs:**
```bash
# View Geth logs
docker compose logs -f geth

# View Prysm logs
docker compose logs -f prysm
```

### 3. Configure Nginx Reverse Proxy

**Create Nginx configuration:**
```bash
sudo nano /etc/nginx/sites-available/sepolia-rpc
```

**Add the following configuration:**
```nginx
server {
    server_name your.domain.com;

    access_log /var/log/nginx/sepolia_rpc_access.log;
    error_log /var/log/nginx/sepolia_rpc_error.log;

    location / {
        # Allow Docker subnet
        allow 172.18.0.0/16;

        # Allow specific IP addresses
        allow 123.123.123.123;
        allow 456.456.456.456;
        
        # Deny all other traffic
        deny all;

        # Proxy configuration
        proxy_pass http://127.0.0.1:8545;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        # Timeout settings
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```
> This is just for sepolia config, for beacon make new config looks like this with specific port and sub-domain.

**Enable the site:**
```bash
# Create symbolic link
sudo ln -s /etc/nginx/sites-available/sepolia-rpc /etc/nginx/sites-enabled/

# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

### 4. Setup SSL with Certbot

**Obtain SSL certificate:**
```bash
# Replace your.domain.com with your actual domain
sudo certbot --nginx -d your.domain.com

# Follow the prompts to configure SSL
```

**Test certificate renewal:**
```bash
sudo certbot renew --dry-run
```

## Services Configuration

### Geth (Execution Client)
- **Image:** `ethereum/client-go:v1.15.11`
- **Network:** Sepolia testnet
- **Sync Mode:** Snap sync (faster initial sync)
- **RPC Endpoint:** `http://localhost:8545`
- **Available APIs:** eth, net, web3
- **Memory:** 8-16GB allocated

### Prysm (Consensus Client)
- **Image:** `gcr.io/prysmaticlabs/prysm/beacon-chain:v6.0.1`
- **Network:** Sepolia testnet
- **HTTP API:** `http://localhost:3500`
- **RPC:** `http://localhost:4000`
- **Memory:** 6-10GB allocated
- **Checkpoint Sync:** Enabled for faster sync

## Port Configuration

| Service | Port | Protocol | Purpose |
|---------|------|----------|---------|
| Geth | 8545 | HTTP | JSON-RPC API (localhost only) |
| Geth | 30303 | TCP/UDP | P2P networking |
| Prysm | 3500 | HTTP | Beacon API (localhost only) |
| Prysm | 4000 | RPC | gRPC API (localhost only) |
| Prysm | 13000 | TCP | P2P networking |
| Prysm | 12000 | UDP | P2P networking |

## Data Persistence

All blockchain data is persisted in the `./data` directory:
- `./data/geth-sepolia/` - Geth blockchain data
- `./data/prysm/` - Prysm beacon chain data
- `./data/jwt.hex` - JWT secret for client authentication

## Usage Examples

### Direct Access (Local)
```bash
# Check Geth sync status
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' \
  http://localhost:8545

# Check latest block
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  http://localhost:8545
```

### Through Nginx Proxy (Remote)
```bash
# Using your domain with SSL
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' \
  https://your.domain.com

# Check beacon chain status
curl https://your.domain.com/beacon/eth/v1/node/health
```

## Monitoring and Maintenance

### Check Service Status
```bash
docker ps -a
```

### View Resource Usage
```bash
docker stats geth-sepolia prysm-sepolia
```

### Restart Services
```bash
# Restart all services
docker compose restart

# Restart individual service
docker compose restart geth
docker compose restart prysm
```

### Update Services
```bash
# Pull latest images
docker compose pull

# Restart with new images
docker compose up -d
```

## Sync Time Expectations

- **Initial Sync:** 2-6 hours depending on hardware and network
- **Geth Snap Sync:** Usually completes faster than full sync
- **Prysm Checkpoint Sync:** Starts from recent checkpoint, much faster than genesis sync

## Troubleshooting

### Common Issues

1. **Port conflicts:** Ensure ports 8545, 3500, 4000, 30303, 13000, 12000 are available
2. **Insufficient memory:** Increase Docker memory limits if containers are killed
3. **Slow sync:** Check disk I/O performance and network connectivity
4. **JWT authentication errors:** Ensure jwt.hex file exists and is readable
5. **Nginx 502 Bad Gateway:** Check if Docker containers are running and accessible
6. **SSL certificate issues:** Verify domain DNS and Certbot configuration
7. **Access denied (403):** Check IP whitelist in Nginx configuration

### Useful Commands

```bash
# Check if services are responding
curl http://localhost:8545 -d '{"jsonrpc":"2.0","method":"web3_clientVersion","id":1}'
curl http://localhost:3500/eth/v1/node/health

# Check through Nginx proxy
curl https://your.domain.com -d '{"jsonrpc":"2.0","method":"web3_clientVersion","id":1}'

# Check Nginx status and logs
sudo systemctl status nginx
sudo tail -f /var/log/nginx/sepolia_rpc_access.log
sudo tail -f /var/log/nginx/sepolia_rpc_error.log

# Check SSL certificate status
sudo certbot certificates

# Test Nginx configuration
sudo nginx -t

# Reload Nginx after configuration changes
sudo systemctl reload nginx

# Clear data and restart (WARNING: This will delete all synced data)
docker compose down
sudo rm -rf data/geth-sepolia data/prysm
docker compose up -d
```

### Docker Network Issues
```bash
# Check Docker networks
docker network ls

# Inspect Docker network
docker network inspect bridge

# Check container connectivity
docker exec -it geth-sepolia ping prysm-sepolia
```

### Certificate Renewal
```bash
# Manual certificate renewal
sudo certbot renew

# Check certificate expiry
sudo certbot certificates

# Test automatic renewal
sudo certbot renew --dry-run
```

## Security Considerations

### Network Security
- RPC endpoints are bound to localhost only for direct access
- Nginx proxy provides controlled external access with IP whitelisting
- SSL/TLS encryption for all external traffic
- P2P ports are exposed only for blockchain network connectivity

### Access Control Configuration
The Nginx configuration includes several layers of security:

1. **IP Whitelisting**: Only specified IP addresses can access the RPC
   ```nginx
   # Allow Docker subnet
   allow 172.18.0.0/16;
   
   # Allow specific IP addresses
   allow 123.123.123.123;
   allow 456.456.456.456;
   
   # Deny all others
   deny all;
   ```

2. **SSL/TLS Encryption**: All external traffic is encrypted via HTTPS

3. **Reverse Proxy Headers**: Proper forwarding of client information

### Additional Security Tips
- Regularly update Docker images and system packages
- Use strong passwords and SSH key authentication
- Consider implementing fail2ban for additional protection
- Monitor access logs regularly: `/var/log/nginx/sepolia_rpc_access.log`
- Backup your node data and SSL certificates regularly