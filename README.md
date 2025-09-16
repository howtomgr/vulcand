# vulcand Installation Guide

vulcand is a free and open-source HTTP proxy. Vulcand provides programmatic load balancer backed by etcd

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
  - CPU: 1 core minimum
  - RAM: 512MB minimum
  - Storage: 1GB for config
  - Network: HTTP/HTTPS
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8181 (default vulcand port)
  - API on 8182
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

# Install vulcand
sudo dnf install -y vulcand

# Enable and start service
sudo systemctl enable --now vulcand

# Configure firewall
sudo firewall-cmd --permanent --add-port=8181/tcp
sudo firewall-cmd --reload

# Verify installation
vulcand --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install vulcand
sudo apt install -y vulcand

# Enable and start service
sudo systemctl enable --now vulcand

# Configure firewall
sudo ufw allow 8181

# Verify installation
vulcand --version
```

### Arch Linux

```bash
# Install vulcand
sudo pacman -S vulcand

# Enable and start service
sudo systemctl enable --now vulcand

# Verify installation
vulcand --version
```

### Alpine Linux

```bash
# Install vulcand
apk add --no-cache vulcand

# Enable and start service
rc-update add vulcand default
rc-service vulcand start

# Verify installation
vulcand --version
```

### openSUSE/SLES

```bash
# Install vulcand
sudo zypper install -y vulcand

# Enable and start service
sudo systemctl enable --now vulcand

# Configure firewall
sudo firewall-cmd --permanent --add-port=8181/tcp
sudo firewall-cmd --reload

# Verify installation
vulcand --version
```

### macOS

```bash
# Using Homebrew
brew install vulcand

# Start service
brew services start vulcand

# Verify installation
vulcand --version
```

### FreeBSD

```bash
# Using pkg
pkg install vulcand

# Enable in rc.conf
echo 'vulcand_enable="YES"' >> /etc/rc.conf

# Start service
service vulcand start

# Verify installation
vulcand --version
```

### Windows

```bash
# Using Chocolatey
choco install vulcand

# Or using Scoop
scoop install vulcand

# Verify installation
vulcand --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/vulcand

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
vulcand --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable vulcand

# Start service
sudo systemctl start vulcand

# Stop service
sudo systemctl stop vulcand

# Restart service
sudo systemctl restart vulcand

# Check status
sudo systemctl status vulcand

# View logs
sudo journalctl -u vulcand -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add vulcand default

# Start service
rc-service vulcand start

# Stop service
rc-service vulcand stop

# Restart service
rc-service vulcand restart

# Check status
rc-service vulcand status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'vulcand_enable="YES"' >> /etc/rc.conf

# Start service
service vulcand start

# Stop service
service vulcand stop

# Restart service
service vulcand restart

# Check status
service vulcand status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start vulcand
brew services stop vulcand
brew services restart vulcand

# Check status
brew services list | grep vulcand
```

### Windows Service Manager

```powershell
# Start service
net start vulcand

# Stop service
net stop vulcand

# Using PowerShell
Start-Service vulcand
Stop-Service vulcand
Restart-Service vulcand

# Check status
Get-Service vulcand
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream vulcand_backend {
    server 127.0.0.1:8181;
}

server {
    listen 80;
    server_name vulcand.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name vulcand.example.com;

    ssl_certificate /etc/ssl/certs/vulcand.example.com.crt;
    ssl_certificate_key /etc/ssl/private/vulcand.example.com.key;

    location / {
        proxy_pass http://vulcand_backend;
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
    ServerName vulcand.example.com
    Redirect permanent / https://vulcand.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName vulcand.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/vulcand.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/vulcand.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8181/
    ProxyPassReverse / http://127.0.0.1:8181/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend vulcand_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/vulcand.pem
    redirect scheme https if !{ ssl_fc }
    default_backend vulcand_backend

backend vulcand_backend
    balance roundrobin
    server vulcand1 127.0.0.1:8181 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R vulcand:vulcand /etc/vulcand
sudo chmod 750 /etc/vulcand

# Configure firewall
sudo firewall-cmd --permanent --add-port=8181/tcp
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
sudo systemctl status vulcand

# View logs
sudo journalctl -u vulcand -f

# Monitor resource usage
top -p $(pgrep vulcand)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/vulcand"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/vulcand-backup-$DATE.tar.gz" /etc/vulcand /var/lib/vulcand

echo "Backup completed: $BACKUP_DIR/vulcand-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop vulcand

# Restore from backup
tar -xzf /backup/vulcand/vulcand-backup-*.tar.gz -C /

# Start service
sudo systemctl start vulcand
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u vulcand -n 100
sudo tail -f /var/log/vulcand/vulcand.log

# Check configuration
vulcand --version

# Check permissions
ls -la /etc/vulcand
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8181

# Test connectivity
telnet localhost 8181

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep vulcand)

# Check disk I/O
iotop -p $(pgrep vulcand)

# Check connections
ss -an | grep 8181
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  vulcand:
    image: vulcand:latest
    ports:
      - "8181:8181"
    volumes:
      - ./config:/etc/vulcand
      - ./data:/var/lib/vulcand
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update vulcand

# Debian/Ubuntu
sudo apt update && sudo apt upgrade vulcand

# Arch Linux
sudo pacman -Syu vulcand

# Alpine Linux
apk update && apk upgrade vulcand

# openSUSE
sudo zypper update vulcand

# FreeBSD
pkg update && pkg upgrade vulcand

# Always backup before updates
tar -czf /backup/vulcand-pre-update-$(date +%Y%m%d).tar.gz /etc/vulcand

# Restart after updates
sudo systemctl restart vulcand
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/vulcand

# Clean old logs
find /var/log/vulcand -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/vulcand
```

## Additional Resources

- Official Documentation: https://docs.vulcand.org/
- GitHub Repository: https://github.com/vulcand/vulcand
- Community Forum: https://forum.vulcand.org/
- Best Practices Guide: https://docs.vulcand.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
