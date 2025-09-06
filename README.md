# RustDesk Server - Self-Hosted Remote Desktop Solution

🚀 **Complete Docker Compose setup for RustDesk server - Your own secure, open-source TeamViewer alternative!**

[![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![RustDesk](https://img.shields.io/badge/RustDesk-Server-orange?style=for-the-badge)](https://rustdesk.com/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge)](LICENSE)

## 📋 Overview

This repository provides a complete, production-ready Docker Compose setup for hosting your own RustDesk relay server. RustDesk is a secure, high-performance remote desktop application that serves as an excellent open-source alternative to TeamViewer.

### ✨ Features

- 🔒 **Self-hosted & Secure**: Complete control over your remote desktop infrastructure
- 🐳 **Docker Compose**: Easy deployment and management
- 🌐 **Cross-Platform**: Works with Windows, macOS, Linux, iOS, and Android clients
- ⚡ **High Performance**: Low latency remote desktop experience
- 📱 **Mobile Support**: Control your devices from anywhere
- 🔐 **Encrypted Connections**: End-to-end encryption for all sessions
- 🚀 **Easy Setup**: Get running in minutes with our configuration

## 🛠 Quick Setup

### Prerequisites

- Docker and Docker Compose installed
- A server with public IP (VPS/dedicated server)
- Ports 21115-21119 available and accessible

### 1. Clone Repository

```bash
git clone https://github.com/a-ariff/rustdesk-server.git
cd rustdesk-server
```

### 2. Configure Environment

```bash
cp .env.example .env
# Edit .env file with your settings
```

### 3. Deploy Server

```bash
docker-compose up -d
```

### 4. Get Your Key

```bash
# Get the public key for client configuration
docker-compose logs hbbs | grep "Key:"
```

## 🔧 Configuration

### Environment Variables (.env)

```bash
# Server Configuration
SERVER_IP=your.server.ip.address
RELAY_DOMAIN=your-domain.com

# Container Settings
HBBS_PORT=21115
HBBR_PORT=21116
HBBR_PORT_WS=21117

# Security
ENCRYPTED_ONLY=Y
KEY_FILE=/root/key.pem

# Logging
RUST_LOG=info
```

### Docker Compose Services

#### HBBS (RustDesk Relay Server)
- **Port 21115**: Main relay service
- **Port 21116**: NAT traversal
- Handles client connections and authentication

#### HBBR (RustDesk Bridge Server)
- **Port 21116**: Bridge service
- **Port 21117**: WebSocket support
- Manages peer-to-peer connections

## 🌐 Required Ports & Firewall

### Port Configuration

| Port  | Protocol | Service | Description |
|-------|----------|---------|-------------|
| 21115 | TCP      | HBBS    | Main relay server |
| 21116 | TCP      | HBBR    | Bridge server |
| 21116 | UDP      | HBBS    | NAT traversal |
| 21117 | TCP      | HBBR    | WebSocket |
| 21118 | TCP      | HBBS    | Web console (optional) |
| 21119 | TCP      | HBBR    | Alternative port |

### Firewall Rules

#### UFW (Ubuntu/Debian)
```bash
sudo ufw allow 21115:21119/tcp
sudo ufw allow 21116/udp
sudo ufw reload
```

#### Firewalld (RHEL/CentOS)
```bash
sudo firewall-cmd --permanent --add-port=21115-21119/tcp
sudo firewall-cmd --permanent --add-port=21116/udp
sudo firewall-cmd --reload
```

#### iptables
```bash
sudo iptables -A INPUT -p tcp --dport 21115:21119 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 21116 -j ACCEPT
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

## 📱 Client Configuration

### Desktop Clients (Windows/macOS/Linux)

1. **Download RustDesk** from [official website](https://rustdesk.com/)
2. **Open Settings** → Network
3. **Configure server settings**:
   ```
   ID Server: your.server.ip.address:21115
   Relay Server: your.server.ip.address:21116
   Key: [paste your server key here]
   ```
4. **Apply settings** and restart RustDesk

### Mobile Clients (iOS/Android)

1. **Download RustDesk** from App Store/Google Play
2. **Go to Settings** → Advanced
3. **Set server configuration**:
   - Server: `your.server.ip.address`
   - Key: `[your server key]`
4. **Save and restart** the app

### Custom Clients (Web)

Access the web client at: `https://your.server.ip.address:21118`

## 🔐 Security Best Practices

### SSL/TLS Configuration

```yaml
# Add to docker-compose.yml for HTTPS
services:
  nginx:
    image: nginx:alpine
    ports:
      - "443:443"
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/ssl/certs
```

### Key Management

```bash
# Generate new encryption key
openssl genpkey -algorithm RSA -out key.pem -pkcs8

# Set proper permissions
chmod 600 key.pem
chown root:root key.pem
```

### Access Control

```bash
# Restrict access by IP (optional)
docker-compose exec hbbs rustdesk-server --allow-ip "192.168.1.0/24,10.0.0.0/8"
```

## 📊 Monitoring & Maintenance

### Health Checks

```bash
# Check service status
docker-compose ps

# View logs
docker-compose logs -f hbbs
docker-compose logs -f hbbr

# Monitor connections
docker-compose exec hbbs netstat -tuln
```

### Backup Configuration

```bash
#!/bin/bash
# backup.sh
DATE=$(date +%Y%m%d_%H%M%S)
mkdir -p backups
cp .env backups/.env_$DATE
cp docker-compose.yml backups/docker-compose_$DATE.yml
echo "Backup completed: $DATE"
```

### Updates

```bash
# Update containers
docker-compose pull
docker-compose down
docker-compose up -d

# Clean old images
docker system prune -f
```

## 🚀 Advanced Configuration

### Custom Domain Setup

```bash
# Add to your DNS:
# A record: rustdesk.yourdomain.com → your.server.ip.address

# Update .env:
RELAY_DOMAIN=rustdesk.yourdomain.com
```

### Load Balancing

```yaml
services:
  hbbs-1:
    image: rustdesk/rustdesk-server:latest
    command: hbbs -r hbbr-1:21117
    
  hbbs-2:
    image: rustdesk/rustdesk-server:latest
    command: hbbs -r hbbr-2:21117
    
  nginx-lb:
    image: nginx:alpine
    # Load balancer configuration
```

## 🐛 Troubleshooting

### Common Issues

#### Connection Failed
```bash
# Check if ports are open
telnet your.server.ip.address 21115

# Verify firewall rules
sudo ufw status

# Check container logs
docker-compose logs hbbs
```

#### Key Errors
```bash
# Regenerate server key
docker-compose down
docker-compose run --rm hbbs rustdesk-server --gen-key
docker-compose up -d
```

#### Performance Issues
```bash
# Monitor resource usage
docker stats

# Check network latency
ping -c 10 your.server.ip.address
```

### Support

- 📖 **Documentation**: [RustDesk Official Docs](https://rustdesk.com/docs/)
- 💬 **Community**: [RustDesk Discord](https://discord.gg/nDceKgxnkV)
- 🐛 **Issues**: [Report Issues](https://github.com/a-ariff/rustdesk-server/issues)

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## ⭐ Support This Project

If this repository helped you set up your RustDesk server, please consider:

- ⭐ **Starring this repository**
- 🐛 **Reporting issues**
- 💡 **Suggesting improvements**
- 🤝 **Contributing code**

---

**Made with ❤️ by [Ariff Mohamed](https://github.com/a-ariff)**

*Self-hosted remote desktop solution for ultimate privacy and control*
