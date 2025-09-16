# nexus Installation Guide

nexus is a free and open-source repository manager. Sonatype Nexus manages binaries and build artifacts across your software supply chain

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
  - RAM: 4GB minimum (8GB+ recommended)
  - Storage: 100GB for artifacts
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8081 (default nexus port)
  - Docker registry on 5000
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

# Install nexus
sudo dnf install -y nexus

# Enable and start service
sudo systemctl enable --now nexus

# Configure firewall
sudo firewall-cmd --permanent --add-port=8081/tcp
sudo firewall-cmd --reload

# Verify installation
nexus --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install nexus
sudo apt install -y nexus

# Enable and start service
sudo systemctl enable --now nexus

# Configure firewall
sudo ufw allow 8081

# Verify installation
nexus --version
```

### Arch Linux

```bash
# Install nexus
sudo pacman -S nexus

# Enable and start service
sudo systemctl enable --now nexus

# Verify installation
nexus --version
```

### Alpine Linux

```bash
# Install nexus
apk add --no-cache nexus

# Enable and start service
rc-update add nexus default
rc-service nexus start

# Verify installation
nexus --version
```

### openSUSE/SLES

```bash
# Install nexus
sudo zypper install -y nexus

# Enable and start service
sudo systemctl enable --now nexus

# Configure firewall
sudo firewall-cmd --permanent --add-port=8081/tcp
sudo firewall-cmd --reload

# Verify installation
nexus --version
```

### macOS

```bash
# Using Homebrew
brew install nexus

# Start service
brew services start nexus

# Verify installation
nexus --version
```

### FreeBSD

```bash
# Using pkg
pkg install nexus

# Enable in rc.conf
echo 'nexus_enable="YES"' >> /etc/rc.conf

# Start service
service nexus start

# Verify installation
nexus --version
```

### Windows

```bash
# Using Chocolatey
choco install nexus

# Or using Scoop
scoop install nexus

# Verify installation
nexus --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/nexus

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
nexus --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable nexus

# Start service
sudo systemctl start nexus

# Stop service
sudo systemctl stop nexus

# Restart service
sudo systemctl restart nexus

# Check status
sudo systemctl status nexus

# View logs
sudo journalctl -u nexus -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add nexus default

# Start service
rc-service nexus start

# Stop service
rc-service nexus stop

# Restart service
rc-service nexus restart

# Check status
rc-service nexus status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'nexus_enable="YES"' >> /etc/rc.conf

# Start service
service nexus start

# Stop service
service nexus stop

# Restart service
service nexus restart

# Check status
service nexus status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start nexus
brew services stop nexus
brew services restart nexus

# Check status
brew services list | grep nexus
```

### Windows Service Manager

```powershell
# Start service
net start nexus

# Stop service
net stop nexus

# Using PowerShell
Start-Service nexus
Stop-Service nexus
Restart-Service nexus

# Check status
Get-Service nexus
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream nexus_backend {
    server 127.0.0.1:8081;
}

server {
    listen 80;
    server_name nexus.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name nexus.example.com;

    ssl_certificate /etc/ssl/certs/nexus.example.com.crt;
    ssl_certificate_key /etc/ssl/private/nexus.example.com.key;

    location / {
        proxy_pass http://nexus_backend;
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
    ServerName nexus.example.com
    Redirect permanent / https://nexus.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName nexus.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/nexus.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/nexus.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8081/
    ProxyPassReverse / http://127.0.0.1:8081/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend nexus_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/nexus.pem
    redirect scheme https if !{ ssl_fc }
    default_backend nexus_backend

backend nexus_backend
    balance roundrobin
    server nexus1 127.0.0.1:8081 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R nexus:nexus /etc/nexus
sudo chmod 750 /etc/nexus

# Configure firewall
sudo firewall-cmd --permanent --add-port=8081/tcp
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
sudo systemctl status nexus

# View logs
sudo journalctl -u nexus -f

# Monitor resource usage
top -p $(pgrep nexus)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/nexus"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/nexus-backup-$DATE.tar.gz" /etc/nexus /var/lib/nexus

echo "Backup completed: $BACKUP_DIR/nexus-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop nexus

# Restore from backup
tar -xzf /backup/nexus/nexus-backup-*.tar.gz -C /

# Start service
sudo systemctl start nexus
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u nexus -n 100
sudo tail -f /var/log/nexus/nexus.log

# Check configuration
nexus --version

# Check permissions
ls -la /etc/nexus
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8081

# Test connectivity
telnet localhost 8081

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep nexus)

# Check disk I/O
iotop -p $(pgrep nexus)

# Check connections
ss -an | grep 8081
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  nexus:
    image: nexus:latest
    ports:
      - "8081:8081"
    volumes:
      - ./config:/etc/nexus
      - ./data:/var/lib/nexus
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update nexus

# Debian/Ubuntu
sudo apt update && sudo apt upgrade nexus

# Arch Linux
sudo pacman -Syu nexus

# Alpine Linux
apk update && apk upgrade nexus

# openSUSE
sudo zypper update nexus

# FreeBSD
pkg update && pkg upgrade nexus

# Always backup before updates
tar -czf /backup/nexus-pre-update-$(date +%Y%m%d).tar.gz /etc/nexus

# Restart after updates
sudo systemctl restart nexus
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/nexus

# Clean old logs
find /var/log/nexus -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/nexus
```

## Additional Resources

- Official Documentation: https://docs.nexus.org/
- GitHub Repository: https://github.com/nexus/nexus
- Community Forum: https://forum.nexus.org/
- Best Practices Guide: https://docs.nexus.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
