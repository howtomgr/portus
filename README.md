# portus Installation Guide

portus is a free and open-source Docker registry frontend. Portus provides authorization service and frontend for Docker Registry

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
  - Storage: 5GB for data
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 3000 (default portus port)
  - None
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

# Install portus
sudo dnf install -y portus

# Enable and start service
sudo systemctl enable --now portus

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload

# Verify installation
portus --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install portus
sudo apt install -y portus

# Enable and start service
sudo systemctl enable --now portus

# Configure firewall
sudo ufw allow 3000

# Verify installation
portus --version
```

### Arch Linux

```bash
# Install portus
sudo pacman -S portus

# Enable and start service
sudo systemctl enable --now portus

# Verify installation
portus --version
```

### Alpine Linux

```bash
# Install portus
apk add --no-cache portus

# Enable and start service
rc-update add portus default
rc-service portus start

# Verify installation
portus --version
```

### openSUSE/SLES

```bash
# Install portus
sudo zypper install -y portus

# Enable and start service
sudo systemctl enable --now portus

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload

# Verify installation
portus --version
```

### macOS

```bash
# Using Homebrew
brew install portus

# Start service
brew services start portus

# Verify installation
portus --version
```

### FreeBSD

```bash
# Using pkg
pkg install portus

# Enable in rc.conf
echo 'portus_enable="YES"' >> /etc/rc.conf

# Start service
service portus start

# Verify installation
portus --version
```

### Windows

```bash
# Using Chocolatey
choco install portus

# Or using Scoop
scoop install portus

# Verify installation
portus --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/portus

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
portus --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable portus

# Start service
sudo systemctl start portus

# Stop service
sudo systemctl stop portus

# Restart service
sudo systemctl restart portus

# Check status
sudo systemctl status portus

# View logs
sudo journalctl -u portus -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add portus default

# Start service
rc-service portus start

# Stop service
rc-service portus stop

# Restart service
rc-service portus restart

# Check status
rc-service portus status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'portus_enable="YES"' >> /etc/rc.conf

# Start service
service portus start

# Stop service
service portus stop

# Restart service
service portus restart

# Check status
service portus status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start portus
brew services stop portus
brew services restart portus

# Check status
brew services list | grep portus
```

### Windows Service Manager

```powershell
# Start service
net start portus

# Stop service
net stop portus

# Using PowerShell
Start-Service portus
Stop-Service portus
Restart-Service portus

# Check status
Get-Service portus
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream portus_backend {
    server 127.0.0.1:3000;
}

server {
    listen 80;
    server_name portus.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name portus.example.com;

    ssl_certificate /etc/ssl/certs/portus.example.com.crt;
    ssl_certificate_key /etc/ssl/private/portus.example.com.key;

    location / {
        proxy_pass http://portus_backend;
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
    ServerName portus.example.com
    Redirect permanent / https://portus.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName portus.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/portus.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/portus.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend portus_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/portus.pem
    redirect scheme https if !{ ssl_fc }
    default_backend portus_backend

backend portus_backend
    balance roundrobin
    server portus1 127.0.0.1:3000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R portus:portus /etc/portus
sudo chmod 750 /etc/portus

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
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
sudo systemctl status portus

# View logs
sudo journalctl -u portus -f

# Monitor resource usage
top -p $(pgrep portus)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/portus"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/portus-backup-$DATE.tar.gz" /etc/portus /var/lib/portus

echo "Backup completed: $BACKUP_DIR/portus-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop portus

# Restore from backup
tar -xzf /backup/portus/portus-backup-*.tar.gz -C /

# Start service
sudo systemctl start portus
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u portus -n 100
sudo tail -f /var/log/portus/portus.log

# Check configuration
portus --version

# Check permissions
ls -la /etc/portus
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 3000

# Test connectivity
telnet localhost 3000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep portus)

# Check disk I/O
iotop -p $(pgrep portus)

# Check connections
ss -an | grep 3000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  portus:
    image: portus:latest
    ports:
      - "3000:3000"
    volumes:
      - ./config:/etc/portus
      - ./data:/var/lib/portus
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update portus

# Debian/Ubuntu
sudo apt update && sudo apt upgrade portus

# Arch Linux
sudo pacman -Syu portus

# Alpine Linux
apk update && apk upgrade portus

# openSUSE
sudo zypper update portus

# FreeBSD
pkg update && pkg upgrade portus

# Always backup before updates
tar -czf /backup/portus-pre-update-$(date +%Y%m%d).tar.gz /etc/portus

# Restart after updates
sudo systemctl restart portus
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/portus

# Clean old logs
find /var/log/portus -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/portus
```

## Additional Resources

- Official Documentation: https://docs.portus.org/
- GitHub Repository: https://github.com/portus/portus
- Community Forum: https://forum.portus.org/
- Best Practices Guide: https://docs.portus.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
