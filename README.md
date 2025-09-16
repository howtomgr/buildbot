# buildbot Installation Guide

buildbot is a free and open-source CI framework. Buildbot provides Python-based CI framework for automating builds

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 2+ cores
  - RAM: 2GB minimum
  - Storage: 10GB for builds
  - Network: HTTP/Git access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8010 (default buildbot port)
  - Worker on 9989
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install buildbot
sudo dnf install -y buildbot

# Enable and start service
sudo systemctl enable --now buildbot

# Configure firewall
sudo firewall-cmd --permanent --add-port=8010/tcp
sudo firewall-cmd --reload

# Verify installation
buildbot --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install buildbot
sudo apt install -y buildbot

# Enable and start service
sudo systemctl enable --now buildbot

# Configure firewall
sudo ufw allow 8010

# Verify installation
buildbot --version
```

### Arch Linux

```bash
# Install buildbot
sudo pacman -S buildbot

# Enable and start service
sudo systemctl enable --now buildbot

# Verify installation
buildbot --version
```

### Alpine Linux

```bash
# Install buildbot
apk add --no-cache buildbot

# Enable and start service
rc-update add buildbot default
rc-service buildbot start

# Verify installation
buildbot --version
```

### openSUSE/SLES

```bash
# Install buildbot
sudo zypper install -y buildbot

# Enable and start service
sudo systemctl enable --now buildbot

# Configure firewall
sudo firewall-cmd --permanent --add-port=8010/tcp
sudo firewall-cmd --reload

# Verify installation
buildbot --version
```

### macOS

```bash
# Using Homebrew
brew install buildbot

# Start service
brew services start buildbot

# Verify installation
buildbot --version
```

### FreeBSD

```bash
# Using pkg
pkg install buildbot

# Enable in rc.conf
echo 'buildbot_enable="YES"' >> /etc/rc.conf

# Start service
service buildbot start

# Verify installation
buildbot --version
```

### Windows

```bash
# Using Chocolatey
choco install buildbot

# Or using Scoop
scoop install buildbot

# Verify installation
buildbot --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/buildbot

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
buildbot --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable buildbot

# Start service
sudo systemctl start buildbot

# Stop service
sudo systemctl stop buildbot

# Restart service
sudo systemctl restart buildbot

# Check status
sudo systemctl status buildbot

# View logs
sudo journalctl -u buildbot -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add buildbot default

# Start service
rc-service buildbot start

# Stop service
rc-service buildbot stop

# Restart service
rc-service buildbot restart

# Check status
rc-service buildbot status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'buildbot_enable="YES"' >> /etc/rc.conf

# Start service
service buildbot start

# Stop service
service buildbot stop

# Restart service
service buildbot restart

# Check status
service buildbot status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start buildbot
brew services stop buildbot
brew services restart buildbot

# Check status
brew services list | grep buildbot
```

### Windows Service Manager

```powershell
# Start service
net start buildbot

# Stop service
net stop buildbot

# Using PowerShell
Start-Service buildbot
Stop-Service buildbot
Restart-Service buildbot

# Check status
Get-Service buildbot
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream buildbot_backend {
    server 127.0.0.1:8010;
}

server {
    listen 80;
    server_name buildbot.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name buildbot.example.com;

    ssl_certificate /etc/ssl/certs/buildbot.example.com.crt;
    ssl_certificate_key /etc/ssl/private/buildbot.example.com.key;

    location / {
        proxy_pass http://buildbot_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName buildbot.example.com
    Redirect permanent / https://buildbot.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName buildbot.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/buildbot.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/buildbot.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8010/
    ProxyPassReverse / http://127.0.0.1:8010/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend buildbot_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/buildbot.pem
    redirect scheme https if !{ ssl_fc }
    default_backend buildbot_backend

backend buildbot_backend
    balance roundrobin
    server buildbot1 127.0.0.1:8010 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R buildbot:buildbot /etc/buildbot
sudo chmod 750 /etc/buildbot

# Configure firewall
sudo firewall-cmd --permanent --add-port=8010/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status buildbot

# View logs
sudo journalctl -u buildbot -f

# Monitor resource usage
top -p $(pgrep buildbot)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/buildbot"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/buildbot-backup-$DATE.tar.gz" /etc/buildbot /var/lib/buildbot

echo "Backup completed: $BACKUP_DIR/buildbot-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop buildbot

# Restore from backup
tar -xzf /backup/buildbot/buildbot-backup-*.tar.gz -C /

# Start service
sudo systemctl start buildbot
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u buildbot -n 100
sudo tail -f /var/log/buildbot/buildbot.log

# Check configuration
buildbot --version

# Check permissions
ls -la /etc/buildbot
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8010

# Test connectivity
telnet localhost 8010

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep buildbot)

# Check disk I/O
iotop -p $(pgrep buildbot)

# Check connections
ss -an | grep 8010
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  buildbot:
    image: buildbot:latest
    ports:
      - "8010:8010"
    volumes:
      - ./config:/etc/buildbot
      - ./data:/var/lib/buildbot
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update buildbot

# Debian/Ubuntu
sudo apt update && sudo apt upgrade buildbot

# Arch Linux
sudo pacman -Syu buildbot

# Alpine Linux
apk update && apk upgrade buildbot

# openSUSE
sudo zypper update buildbot

# FreeBSD
pkg update && pkg upgrade buildbot

# Always backup before updates
tar -czf /backup/buildbot-pre-update-$(date +%Y%m%d).tar.gz /etc/buildbot

# Restart after updates
sudo systemctl restart buildbot
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/buildbot

# Clean old logs
find /var/log/buildbot -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/buildbot
```

## Additional Resources

- Official Documentation: https://docs.buildbot.org/
- GitHub Repository: https://github.com/buildbot/buildbot
- Community Forum: https://forum.buildbot.org/
- Best Practices Guide: https://docs.buildbot.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
