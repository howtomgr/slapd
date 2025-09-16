# openldap Installation Guide

openldap is a free and open-source open source LDAP directory server. OpenLDAP provides directory services for user authentication and authorization, serving as an alternative to Microsoft Active Directory or Oracle Directory Server

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
  - RAM: 512MB minimum (2GB+ recommended)
  - Storage: 1GB for directory data
  - Network: LDAP and LDAPS protocols
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 389 (default openldap port)
  - Port 636 for LDAPS
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

# Install openldap
sudo dnf install -y slapd

# Enable and start service
sudo systemctl enable --now slapd

# Configure firewall
sudo firewall-cmd --permanent --add-port=389/tcp
sudo firewall-cmd --reload

# Verify installation
slapd -V
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install openldap
sudo apt install -y slapd

# Enable and start service
sudo systemctl enable --now slapd

# Configure firewall
sudo ufw allow 389

# Verify installation
slapd -V
```

### Arch Linux

```bash
# Install openldap
sudo pacman -S slapd

# Enable and start service
sudo systemctl enable --now slapd

# Verify installation
slapd -V
```

### Alpine Linux

```bash
# Install openldap
apk add --no-cache slapd

# Enable and start service
rc-update add slapd default
rc-service slapd start

# Verify installation
slapd -V
```

### openSUSE/SLES

```bash
# Install openldap
sudo zypper install -y slapd

# Enable and start service
sudo systemctl enable --now slapd

# Configure firewall
sudo firewall-cmd --permanent --add-port=389/tcp
sudo firewall-cmd --reload

# Verify installation
slapd -V
```

### macOS

```bash
# Using Homebrew
brew install slapd

# Start service
brew services start slapd

# Verify installation
slapd -V
```

### FreeBSD

```bash
# Using pkg
pkg install slapd

# Enable in rc.conf
echo 'slapd_enable="YES"' >> /etc/rc.conf

# Start service
service slapd start

# Verify installation
slapd -V
```

### Windows

```bash
# Using Chocolatey
choco install slapd

# Or using Scoop
scoop install slapd

# Verify installation
slapd -V
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/slapd

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
slapd -V
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable slapd

# Start service
sudo systemctl start slapd

# Stop service
sudo systemctl stop slapd

# Restart service
sudo systemctl restart slapd

# Check status
sudo systemctl status slapd

# View logs
sudo journalctl -u slapd -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add slapd default

# Start service
rc-service slapd start

# Stop service
rc-service slapd stop

# Restart service
rc-service slapd restart

# Check status
rc-service slapd status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'slapd_enable="YES"' >> /etc/rc.conf

# Start service
service slapd start

# Stop service
service slapd stop

# Restart service
service slapd restart

# Check status
service slapd status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start slapd
brew services stop slapd
brew services restart slapd

# Check status
brew services list | grep slapd
```

### Windows Service Manager

```powershell
# Start service
net start slapd

# Stop service
net stop slapd

# Using PowerShell
Start-Service slapd
Stop-Service slapd
Restart-Service slapd

# Check status
Get-Service slapd
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream slapd_backend {
    server 127.0.0.1:389;
}

server {
    listen 80;
    server_name slapd.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name slapd.example.com;

    ssl_certificate /etc/ssl/certs/slapd.example.com.crt;
    ssl_certificate_key /etc/ssl/private/slapd.example.com.key;

    location / {
        proxy_pass http://slapd_backend;
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
    ServerName slapd.example.com
    Redirect permanent / https://slapd.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName slapd.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/slapd.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/slapd.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:389/
    ProxyPassReverse / http://127.0.0.1:389/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend slapd_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/slapd.pem
    redirect scheme https if !{ ssl_fc }
    default_backend slapd_backend

backend slapd_backend
    balance roundrobin
    server slapd1 127.0.0.1:389 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R slapd:slapd /etc/slapd
sudo chmod 750 /etc/slapd

# Configure firewall
sudo firewall-cmd --permanent --add-port=389/tcp
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
sudo systemctl status slapd

# View logs
sudo journalctl -u slapd -f

# Monitor resource usage
top -p $(pgrep slapd)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/slapd"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/slapd-backup-$DATE.tar.gz" /etc/slapd /var/lib/slapd

echo "Backup completed: $BACKUP_DIR/slapd-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop slapd

# Restore from backup
tar -xzf /backup/slapd/slapd-backup-*.tar.gz -C /

# Start service
sudo systemctl start slapd
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u slapd -n 100
sudo tail -f /var/log/slapd/slapd.log

# Check configuration
slapd -V

# Check permissions
ls -la /etc/slapd
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 389

# Test connectivity
telnet localhost 389

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep slapd)

# Check disk I/O
iotop -p $(pgrep slapd)

# Check connections
ss -an | grep 389
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  slapd:
    image: slapd:latest
    ports:
      - "389:389"
    volumes:
      - ./config:/etc/slapd
      - ./data:/var/lib/slapd
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update slapd

# Debian/Ubuntu
sudo apt update && sudo apt upgrade slapd

# Arch Linux
sudo pacman -Syu slapd

# Alpine Linux
apk update && apk upgrade slapd

# openSUSE
sudo zypper update slapd

# FreeBSD
pkg update && pkg upgrade slapd

# Always backup before updates
tar -czf /backup/slapd-pre-update-$(date +%Y%m%d).tar.gz /etc/slapd

# Restart after updates
sudo systemctl restart slapd
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/slapd

# Clean old logs
find /var/log/slapd -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/slapd
```

## Additional Resources

- Official Documentation: https://docs.slapd.org/
- GitHub Repository: https://github.com/slapd/slapd
- Community Forum: https://forum.slapd.org/
- Best Practices Guide: https://docs.slapd.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
