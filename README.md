# emqx Installation Guide

emqx is a free and open-source MQTT broker. EMQX provides scalable MQTT broker for IoT

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
  - RAM: 4GB minimum
  - Storage: 10GB for data
  - Network: MQTT protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 1883 (default emqx port)
  - Dashboard on 18083
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

# Install emqx
sudo dnf install -y emqx

# Enable and start service
sudo systemctl enable --now emqx

# Configure firewall
sudo firewall-cmd --permanent --add-port=1883/tcp
sudo firewall-cmd --reload

# Verify installation
emqx --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install emqx
sudo apt install -y emqx

# Enable and start service
sudo systemctl enable --now emqx

# Configure firewall
sudo ufw allow 1883

# Verify installation
emqx --version
```

### Arch Linux

```bash
# Install emqx
sudo pacman -S emqx

# Enable and start service
sudo systemctl enable --now emqx

# Verify installation
emqx --version
```

### Alpine Linux

```bash
# Install emqx
apk add --no-cache emqx

# Enable and start service
rc-update add emqx default
rc-service emqx start

# Verify installation
emqx --version
```

### openSUSE/SLES

```bash
# Install emqx
sudo zypper install -y emqx

# Enable and start service
sudo systemctl enable --now emqx

# Configure firewall
sudo firewall-cmd --permanent --add-port=1883/tcp
sudo firewall-cmd --reload

# Verify installation
emqx --version
```

### macOS

```bash
# Using Homebrew
brew install emqx

# Start service
brew services start emqx

# Verify installation
emqx --version
```

### FreeBSD

```bash
# Using pkg
pkg install emqx

# Enable in rc.conf
echo 'emqx_enable="YES"' >> /etc/rc.conf

# Start service
service emqx start

# Verify installation
emqx --version
```

### Windows

```bash
# Using Chocolatey
choco install emqx

# Or using Scoop
scoop install emqx

# Verify installation
emqx --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/emqx

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
emqx --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable emqx

# Start service
sudo systemctl start emqx

# Stop service
sudo systemctl stop emqx

# Restart service
sudo systemctl restart emqx

# Check status
sudo systemctl status emqx

# View logs
sudo journalctl -u emqx -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add emqx default

# Start service
rc-service emqx start

# Stop service
rc-service emqx stop

# Restart service
rc-service emqx restart

# Check status
rc-service emqx status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'emqx_enable="YES"' >> /etc/rc.conf

# Start service
service emqx start

# Stop service
service emqx stop

# Restart service
service emqx restart

# Check status
service emqx status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start emqx
brew services stop emqx
brew services restart emqx

# Check status
brew services list | grep emqx
```

### Windows Service Manager

```powershell
# Start service
net start emqx

# Stop service
net stop emqx

# Using PowerShell
Start-Service emqx
Stop-Service emqx
Restart-Service emqx

# Check status
Get-Service emqx
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream emqx_backend {
    server 127.0.0.1:1883;
}

server {
    listen 80;
    server_name emqx.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name emqx.example.com;

    ssl_certificate /etc/ssl/certs/emqx.example.com.crt;
    ssl_certificate_key /etc/ssl/private/emqx.example.com.key;

    location / {
        proxy_pass http://emqx_backend;
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
    ServerName emqx.example.com
    Redirect permanent / https://emqx.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName emqx.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/emqx.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/emqx.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:1883/
    ProxyPassReverse / http://127.0.0.1:1883/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend emqx_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/emqx.pem
    redirect scheme https if !{ ssl_fc }
    default_backend emqx_backend

backend emqx_backend
    balance roundrobin
    server emqx1 127.0.0.1:1883 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R emqx:emqx /etc/emqx
sudo chmod 750 /etc/emqx

# Configure firewall
sudo firewall-cmd --permanent --add-port=1883/tcp
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
sudo systemctl status emqx

# View logs
sudo journalctl -u emqx -f

# Monitor resource usage
top -p $(pgrep emqx)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/emqx"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/emqx-backup-$DATE.tar.gz" /etc/emqx /var/lib/emqx

echo "Backup completed: $BACKUP_DIR/emqx-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop emqx

# Restore from backup
tar -xzf /backup/emqx/emqx-backup-*.tar.gz -C /

# Start service
sudo systemctl start emqx
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u emqx -n 100
sudo tail -f /var/log/emqx/emqx.log

# Check configuration
emqx --version

# Check permissions
ls -la /etc/emqx
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 1883

# Test connectivity
telnet localhost 1883

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep emqx)

# Check disk I/O
iotop -p $(pgrep emqx)

# Check connections
ss -an | grep 1883
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  emqx:
    image: emqx:latest
    ports:
      - "1883:1883"
    volumes:
      - ./config:/etc/emqx
      - ./data:/var/lib/emqx
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update emqx

# Debian/Ubuntu
sudo apt update && sudo apt upgrade emqx

# Arch Linux
sudo pacman -Syu emqx

# Alpine Linux
apk update && apk upgrade emqx

# openSUSE
sudo zypper update emqx

# FreeBSD
pkg update && pkg upgrade emqx

# Always backup before updates
tar -czf /backup/emqx-pre-update-$(date +%Y%m%d).tar.gz /etc/emqx

# Restart after updates
sudo systemctl restart emqx
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/emqx

# Clean old logs
find /var/log/emqx -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/emqx
```

## Additional Resources

- Official Documentation: https://docs.emqx.org/
- GitHub Repository: https://github.com/emqx/emqx
- Community Forum: https://forum.emqx.org/
- Best Practices Guide: https://docs.emqx.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
