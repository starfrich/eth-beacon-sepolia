# Ethereum Sepolia Node Setup

> **⚠️ WARNING: THIS IS NOT A COPY-PASTE GUIDE - PLEASE READ CAREFULLY!**

This repository provides **two different approaches** for running an Ethereum Sepolia testnet node.

## Available Methods

### Method 1: Systemd + UFW (Native) -> [Here](https://github.com/starfrich/eth-beacon-sepolia/tree/main/systemd)
### Method 2: Docker Compose + Nginx + Certbot (Containerized) -> [Here](https://github.com/starfrich/eth-beacon-sepolia/tree/main/docker) 

## Comparison

| Feature | Systemd + UFW | Docker + Nginx + Certbot |
|---------|---------------|--------------------------|
| **Resource Usage** | Higher (can use more than 16GB+ cache) | Lower (~26GB managed) |
| **Setup Complexity** | Simple | Complex |
| **SSL/HTTPS** | Manual | Automated |
| **Updates** | Manual | `docker compose pull` |
| **Isolation** | System-level | Container-level |
| **External Access** | Basic | Professional |

## Pros & Cons

### Systemd + UFW
**Pros:** Simpler setup, direct system access, no container overhead
**Cons:** Higher memory usage, manual updates, no SSL automation, aggressive caching

### Docker + Nginx + Certbot
**Pros:** Better resource control, easy deployment, automated SSL, professional setup
**Cons:** Complex architecture, learning curve, container dependencies

## Choose Based On

**Systemd** → Simple setup, don't mind high memory usage.
**Docker** → Better resource management, professional deployment, external HTTPS access.

## Prerequisites
- Ubuntu/Debian system
- 1TB SSD (NVMe Preferred)
- 16GB+ RAM (systemd can be very memory hungry)
- Basic Linux administration knowledge
- Domain name (Docker method only)