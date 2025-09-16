# headphones Installation Guide

headphones is a free and open-source automated music downloader. Headphones automates music library management and acquisition from various sources

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
  - Storage: 500MB for app
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8181 (default headphones port)
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

# Install headphones
sudo dnf install -y headphones

# Enable and start service
sudo systemctl enable --now headphones

# Configure firewall
sudo firewall-cmd --permanent --add-port=8181/tcp
sudo firewall-cmd --reload

# Verify installation
headphones --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install headphones
sudo apt install -y headphones

# Enable and start service
sudo systemctl enable --now headphones

# Configure firewall
sudo ufw allow 8181

# Verify installation
headphones --version
```

### Arch Linux

```bash
# Install headphones
sudo pacman -S headphones

# Enable and start service
sudo systemctl enable --now headphones

# Verify installation
headphones --version
```

### Alpine Linux

```bash
# Install headphones
apk add --no-cache headphones

# Enable and start service
rc-update add headphones default
rc-service headphones start

# Verify installation
headphones --version
```

### openSUSE/SLES

```bash
# Install headphones
sudo zypper install -y headphones

# Enable and start service
sudo systemctl enable --now headphones

# Configure firewall
sudo firewall-cmd --permanent --add-port=8181/tcp
sudo firewall-cmd --reload

# Verify installation
headphones --version
```

### macOS

```bash
# Using Homebrew
brew install headphones

# Start service
brew services start headphones

# Verify installation
headphones --version
```

### FreeBSD

```bash
# Using pkg
pkg install headphones

# Enable in rc.conf
echo 'headphones_enable="YES"' >> /etc/rc.conf

# Start service
service headphones start

# Verify installation
headphones --version
```

### Windows

```bash
# Using Chocolatey
choco install headphones

# Or using Scoop
scoop install headphones

# Verify installation
headphones --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/headphones

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
headphones --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable headphones

# Start service
sudo systemctl start headphones

# Stop service
sudo systemctl stop headphones

# Restart service
sudo systemctl restart headphones

# Check status
sudo systemctl status headphones

# View logs
sudo journalctl -u headphones -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add headphones default

# Start service
rc-service headphones start

# Stop service
rc-service headphones stop

# Restart service
rc-service headphones restart

# Check status
rc-service headphones status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'headphones_enable="YES"' >> /etc/rc.conf

# Start service
service headphones start

# Stop service
service headphones stop

# Restart service
service headphones restart

# Check status
service headphones status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start headphones
brew services stop headphones
brew services restart headphones

# Check status
brew services list | grep headphones
```

### Windows Service Manager

```powershell
# Start service
net start headphones

# Stop service
net stop headphones

# Using PowerShell
Start-Service headphones
Stop-Service headphones
Restart-Service headphones

# Check status
Get-Service headphones
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream headphones_backend {
    server 127.0.0.1:8181;
}

server {
    listen 80;
    server_name headphones.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name headphones.example.com;

    ssl_certificate /etc/ssl/certs/headphones.example.com.crt;
    ssl_certificate_key /etc/ssl/private/headphones.example.com.key;

    location / {
        proxy_pass http://headphones_backend;
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
    ServerName headphones.example.com
    Redirect permanent / https://headphones.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName headphones.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/headphones.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/headphones.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8181/
    ProxyPassReverse / http://127.0.0.1:8181/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend headphones_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/headphones.pem
    redirect scheme https if !{ ssl_fc }
    default_backend headphones_backend

backend headphones_backend
    balance roundrobin
    server headphones1 127.0.0.1:8181 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R headphones:headphones /etc/headphones
sudo chmod 750 /etc/headphones

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
sudo systemctl status headphones

# View logs
sudo journalctl -u headphones -f

# Monitor resource usage
top -p $(pgrep headphones)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/headphones"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/headphones-backup-$DATE.tar.gz" /etc/headphones /var/lib/headphones

echo "Backup completed: $BACKUP_DIR/headphones-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop headphones

# Restore from backup
tar -xzf /backup/headphones/headphones-backup-*.tar.gz -C /

# Start service
sudo systemctl start headphones
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u headphones -n 100
sudo tail -f /var/log/headphones/headphones.log

# Check configuration
headphones --version

# Check permissions
ls -la /etc/headphones
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
top -p $(pgrep headphones)

# Check disk I/O
iotop -p $(pgrep headphones)

# Check connections
ss -an | grep 8181
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  headphones:
    image: headphones:latest
    ports:
      - "8181:8181"
    volumes:
      - ./config:/etc/headphones
      - ./data:/var/lib/headphones
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update headphones

# Debian/Ubuntu
sudo apt update && sudo apt upgrade headphones

# Arch Linux
sudo pacman -Syu headphones

# Alpine Linux
apk update && apk upgrade headphones

# openSUSE
sudo zypper update headphones

# FreeBSD
pkg update && pkg upgrade headphones

# Always backup before updates
tar -czf /backup/headphones-pre-update-$(date +%Y%m%d).tar.gz /etc/headphones

# Restart after updates
sudo systemctl restart headphones
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/headphones

# Clean old logs
find /var/log/headphones -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/headphones
```

## Additional Resources

- Official Documentation: https://docs.headphones.org/
- GitHub Repository: https://github.com/headphones/headphones
- Community Forum: https://forum.headphones.org/
- Best Practices Guide: https://docs.headphones.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
