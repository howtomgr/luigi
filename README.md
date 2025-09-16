# luigi Installation Guide

luigi is a free and open-source pipeline framework. Luigi helps build complex pipelines of batch jobs

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
  - RAM: 2GB minimum
  - Storage: 5GB for state
  - Network: HTTP interface
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8082 (default luigi port)
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

# Install luigi
sudo dnf install -y luigi

# Enable and start service
sudo systemctl enable --now luigi

# Configure firewall
sudo firewall-cmd --permanent --add-port=8082/tcp
sudo firewall-cmd --reload

# Verify installation
luigi --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install luigi
sudo apt install -y luigi

# Enable and start service
sudo systemctl enable --now luigi

# Configure firewall
sudo ufw allow 8082

# Verify installation
luigi --version
```

### Arch Linux

```bash
# Install luigi
sudo pacman -S luigi

# Enable and start service
sudo systemctl enable --now luigi

# Verify installation
luigi --version
```

### Alpine Linux

```bash
# Install luigi
apk add --no-cache luigi

# Enable and start service
rc-update add luigi default
rc-service luigi start

# Verify installation
luigi --version
```

### openSUSE/SLES

```bash
# Install luigi
sudo zypper install -y luigi

# Enable and start service
sudo systemctl enable --now luigi

# Configure firewall
sudo firewall-cmd --permanent --add-port=8082/tcp
sudo firewall-cmd --reload

# Verify installation
luigi --version
```

### macOS

```bash
# Using Homebrew
brew install luigi

# Start service
brew services start luigi

# Verify installation
luigi --version
```

### FreeBSD

```bash
# Using pkg
pkg install luigi

# Enable in rc.conf
echo 'luigi_enable="YES"' >> /etc/rc.conf

# Start service
service luigi start

# Verify installation
luigi --version
```

### Windows

```bash
# Using Chocolatey
choco install luigi

# Or using Scoop
scoop install luigi

# Verify installation
luigi --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/luigi

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
luigi --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable luigi

# Start service
sudo systemctl start luigi

# Stop service
sudo systemctl stop luigi

# Restart service
sudo systemctl restart luigi

# Check status
sudo systemctl status luigi

# View logs
sudo journalctl -u luigi -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add luigi default

# Start service
rc-service luigi start

# Stop service
rc-service luigi stop

# Restart service
rc-service luigi restart

# Check status
rc-service luigi status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'luigi_enable="YES"' >> /etc/rc.conf

# Start service
service luigi start

# Stop service
service luigi stop

# Restart service
service luigi restart

# Check status
service luigi status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start luigi
brew services stop luigi
brew services restart luigi

# Check status
brew services list | grep luigi
```

### Windows Service Manager

```powershell
# Start service
net start luigi

# Stop service
net stop luigi

# Using PowerShell
Start-Service luigi
Stop-Service luigi
Restart-Service luigi

# Check status
Get-Service luigi
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream luigi_backend {
    server 127.0.0.1:8082;
}

server {
    listen 80;
    server_name luigi.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name luigi.example.com;

    ssl_certificate /etc/ssl/certs/luigi.example.com.crt;
    ssl_certificate_key /etc/ssl/private/luigi.example.com.key;

    location / {
        proxy_pass http://luigi_backend;
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
    ServerName luigi.example.com
    Redirect permanent / https://luigi.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName luigi.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/luigi.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/luigi.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8082/
    ProxyPassReverse / http://127.0.0.1:8082/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend luigi_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/luigi.pem
    redirect scheme https if !{ ssl_fc }
    default_backend luigi_backend

backend luigi_backend
    balance roundrobin
    server luigi1 127.0.0.1:8082 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R luigi:luigi /etc/luigi
sudo chmod 750 /etc/luigi

# Configure firewall
sudo firewall-cmd --permanent --add-port=8082/tcp
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
sudo systemctl status luigi

# View logs
sudo journalctl -u luigi -f

# Monitor resource usage
top -p $(pgrep luigi)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/luigi"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/luigi-backup-$DATE.tar.gz" /etc/luigi /var/lib/luigi

echo "Backup completed: $BACKUP_DIR/luigi-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop luigi

# Restore from backup
tar -xzf /backup/luigi/luigi-backup-*.tar.gz -C /

# Start service
sudo systemctl start luigi
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u luigi -n 100
sudo tail -f /var/log/luigi/luigi.log

# Check configuration
luigi --version

# Check permissions
ls -la /etc/luigi
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8082

# Test connectivity
telnet localhost 8082

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep luigi)

# Check disk I/O
iotop -p $(pgrep luigi)

# Check connections
ss -an | grep 8082
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  luigi:
    image: luigi:latest
    ports:
      - "8082:8082"
    volumes:
      - ./config:/etc/luigi
      - ./data:/var/lib/luigi
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update luigi

# Debian/Ubuntu
sudo apt update && sudo apt upgrade luigi

# Arch Linux
sudo pacman -Syu luigi

# Alpine Linux
apk update && apk upgrade luigi

# openSUSE
sudo zypper update luigi

# FreeBSD
pkg update && pkg upgrade luigi

# Always backup before updates
tar -czf /backup/luigi-pre-update-$(date +%Y%m%d).tar.gz /etc/luigi

# Restart after updates
sudo systemctl restart luigi
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/luigi

# Clean old logs
find /var/log/luigi -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/luigi
```

## Additional Resources

- Official Documentation: https://docs.luigi.org/
- GitHub Repository: https://github.com/luigi/luigi
- Community Forum: https://forum.luigi.org/
- Best Practices Guide: https://docs.luigi.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
