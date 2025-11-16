# Squid Proxy Server

A comprehensive guide to Squid - a full-featured caching proxy server supporting HTTP, HTTPS, FTP, and more.

## Table of Contents

- [Introduction](#introduction)
- [Installation](#installation)
- [Basic Configuration](#basic-configuration)
- [Access Control](#access-control)
- [Caching](#caching)
- [SSL/TLS](#ssltls)
- [Authentication](#authentication)
- [Performance Tuning](#performance-tuning)
- [Monitoring & Logging](#monitoring--logging)
- [Common Use Cases](#common-use-cases)

## Introduction

### What is Squid?

Squid is a caching and forwarding HTTP web proxy. It reduces bandwidth and improves response times by caching and reusing frequently-requested web pages.

### Features

- **HTTP/HTTPS Proxy**: Full proxy functionality
- **Caching**: Reduces bandwidth usage
- **Access Control**: Fine-grained ACLs
- **SSL Bumping**: HTTPS inspection
- **Authentication**: Multiple auth methods
- **Content Filtering**: URL filtering
- **Reverse Proxy**: Load balancing and caching
- **Transparent Proxy**: Invisible to clients

## Installation

### Ubuntu/Debian

```bash
# Update package list
sudo apt update

# Install Squid
sudo apt install squid -y

# Check version
squid -v

# Check service status
sudo systemctl status squid

# Start Squid
sudo systemctl start squid

# Enable on boot
sudo systemctl enable squid
```

### RHEL/CentOS

```bash
# Install Squid
sudo yum install squid -y

# Or with DNF
sudo dnf install squid -y

# Start and enable
sudo systemctl start squid
sudo systemctl enable squid
```

### From Source

```bash
# Install dependencies
sudo apt install build-essential libssl-dev

# Download source
wget http://www.squid-cache.org/Versions/v5/squid-5.9.tar.gz
tar -xzf squid-5.9.tar.gz
cd squid-5.9

# Configure and compile
./configure --prefix=/usr \
  --enable-ssl \
  --enable-ssl-crtd \
  --with-openssl
make
sudo make install
```

## Basic Configuration

### Main Configuration File

```bash
# Location: /etc/squid/squid.conf

# HTTP Port
http_port 3128

# Visible hostname
visible_hostname squid.example.com

# Cache directory
cache_dir ufs /var/spool/squid 10000 16 256
# Format: cache_dir TYPE DIRECTORY SIZE L1 L2
# SIZE in MB, L1 and L2 are subdirectories

# Access log
access_log /var/log/squid/access.log squid

# Cache log
cache_log /var/log/squid/cache.log

# Memory cache size
cache_mem 256 MB

# Maximum object size in cache
maximum_object_size 50 MB

# Maximum object size in memory
maximum_object_size_in_memory 512 KB

# Coredump directory
coredump_dir /var/spool/squid
```

### Minimal Working Configuration

```bash
# /etc/squid/squid.conf

# Port and visible hostname
http_port 3128
visible_hostname proxy.example.com

# Recommended minimum ACL
acl localnet src 10.0.0.0/8
acl localnet src 172.16.0.0/12
acl localnet src 192.168.0.0/16

acl SSL_ports port 443
acl Safe_ports port 80          # http
acl Safe_ports port 21          # ftp
acl Safe_ports port 443         # https
acl Safe_ports port 70          # gopher
acl Safe_ports port 210         # wais
acl Safe_ports port 1025-65535  # unregistered ports
acl Safe_ports port 280         # http-mgmt
acl Safe_ports port 488         # gss-http
acl Safe_ports port 591         # filemaker
acl Safe_ports port 777         # multiling http
acl CONNECT method CONNECT

# Deny requests to unsafe ports
http_access deny !Safe_ports

# Deny CONNECT to other than SSL ports
http_access deny CONNECT !SSL_ports

# Allow localhost
http_access allow localhost manager
http_access deny manager

# Allow local network
http_access allow localnet
http_access allow localhost

# Deny all other access
http_access deny all

# Cache directory
cache_dir ufs /var/spool/squid 10000 16 256
```

## Access Control

### ACL Types

```bash
# Source IP/Network
acl office_network src 192.168.1.0/24
acl admin_ip src 192.168.1.100

# Destination Domain
acl allowed_domains dstdomain .example.com
acl blocked_domains dstdomain .facebook.com .twitter.com

# Destination URL
acl download_files url_regex -i \.exe$ \.zip$ \.rar$

# Time-based ACL
acl business_hours time MTWHF 08:00-18:00

# Destination port
acl http_port port 80
acl https_port port 443

# Method
acl post_method method POST
acl get_method method GET

# URL path regex
acl streaming url_regex -i youtube\.com\/watch

# Request header
acl mobile_devices req_header User-Agent -i mobile|android|iphone
```

### Access Rules

```bash
# Block specific domains
acl social_media dstdomain .facebook.com .twitter.com .instagram.com
http_access deny social_media

# Block during work hours
acl work_hours time MTWHF 09:00-17:00
acl streaming url_regex -i youtube\.com netflix\.com
http_access deny streaming work_hours

# Allow only specific IPs
acl allowed_ips src 192.168.1.0/24
http_access allow allowed_ips
http_access deny all

# Block file downloads
acl exe_files url_regex -i \.exe$
http_access deny exe_files

# Bandwidth limiting per IP
delay_pools 1
delay_class 1 2
delay_parameters 1 -1/-1 8000/8000
delay_access 1 allow all
```

### URL Filtering

```bash
# Create blacklist file
# /etc/squid/blacklist.txt
facebook.com
twitter.com
youtube.com
*.gambling.*

# In squid.conf
acl blacklist dstdomain "/etc/squid/blacklist.txt"
http_access deny blacklist

# Whitelist
acl whitelist dstdomain "/etc/squid/whitelist.txt"
http_access allow whitelist
http_access deny all
```

## Caching

### Cache Configuration

```bash
# Cache directory (Type Directory MaxSize L1Dirs L2Dirs)
cache_dir ufs /var/spool/squid 10000 16 256

# Memory cache
cache_mem 256 MB

# Object size limits
maximum_object_size 100 MB
minimum_object_size 0 KB
maximum_object_size_in_memory 512 KB

# Cache replacement policy
cache_replacement_policy heap LFUDA
memory_replacement_policy heap GDSF

# Refresh patterns
refresh_pattern ^ftp:           1440    20%     10080
refresh_pattern ^gopher:        1440    0%      1440
refresh_pattern -i (/cgi-bin/|\?) 0     0%      0
refresh_pattern .               0       20%     4320

# Don't cache certain content
acl cgi-bin url_regex cgi-bin
cache deny cgi-bin

# Force caching of certain content
acl static_content url_regex -i \.(jpg|jpeg|png|gif|css|js)$
cache allow static_content
```

### Cache Management

```bash
# Initialize cache directory
sudo squid -z

# Clear cache
sudo systemctl stop squid
sudo rm -rf /var/spool/squid/*
sudo squid -z
sudo systemctl start squid

# Check cache stats
squidclient -p 3128 mgr:info
squidclient -p 3128 mgr:storedir

# Cache manager
http_port 3128
cache_mgr admin@example.com
```

## SSL/TLS

### HTTPS Proxy

```bash
# Enable HTTPS proxy
http_port 3129 ssl-bump \
  cert=/etc/squid/ssl/squid.pem \
  generate-host-certificates=on \
  dynamic_cert_mem_cache_size=4MB

# SSL bump configuration
sslcrtd_program /usr/lib/squid/security_file_certgen -s /var/spool/squid/ssl_db -M 4MB

# Initialize SSL database
sudo /usr/lib/squid/security_file_certgen -c -s /var/spool/squid/ssl_db
sudo chown -R proxy:proxy /var/spool/squid/ssl_db

# SSL bump ACL
acl step1 at_step SslBump1
ssl_bump peek step1
ssl_bump bump all

# Don't bump certain sites
acl nobump_domains ssl::server_name .bank.com .secure.example.com
ssl_bump splice nobump_domains
ssl_bump bump all
```

### Generate SSL Certificate

```bash
# Create directory
sudo mkdir -p /etc/squid/ssl
cd /etc/squid/ssl

# Generate private key and certificate
sudo openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 \
  -keyout squid.pem -out squid.pem

# Set permissions
sudo chmod 400 squid.pem
sudo chown proxy:proxy squid.pem
```

## Authentication

### Basic Authentication

```bash
# Install htpasswd
sudo apt install apache2-utils

# Create password file
sudo htpasswd -c /etc/squid/passwords username

# Add more users
sudo htpasswd /etc/squid/passwords another_user

# Configure Squid
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwords
auth_param basic children 5
auth_param basic realm Squid Proxy Server
auth_param basic credentialsttl 2 hours

# Require authentication
acl authenticated proxy_auth REQUIRED
http_access allow authenticated
http_access deny all
```

### LDAP Authentication

```bash
# Install LDAP helper
sudo apt install squid-ldap

# Configure LDAP auth
auth_param basic program /usr/lib/squid/basic_ldap_auth \
  -b "dc=example,dc=com" \
  -D "cn=squid,dc=example,dc=com" \
  -w "password" \
  -f "uid=%s" \
  -h ldap.example.com

auth_param basic children 5
auth_param basic realm Squid Proxy
auth_param basic credentialsttl 2 hours

acl ldap_users proxy_auth REQUIRED
http_access allow ldap_users
```

### Active Directory Authentication

```bash
# Kerberos/NTLM authentication
auth_param negotiate program /usr/lib/squid/negotiate_kerberos_auth \
  -s HTTP/squid.example.com@EXAMPLE.COM

auth_param negotiate children 10
auth_param negotiate keep_alive on

# NTLM fallback
auth_param ntlm program /usr/lib/squid/ntlm_auth
auth_param ntlm children 5

acl ad_users proxy_auth REQUIRED
http_access allow ad_users
```

## Performance Tuning

### Memory and CPU

```bash
# Memory cache
cache_mem 512 MB

# File descriptors
max_filedescriptors 4096

# CPU affinity (for multi-core)
cpu_affinity_map process_numbers=1,2,3,4 cores=1,2,3,4

# Worker processes (Squid 3.5+)
workers 4
```

### Network Tuning

```bash
# Connection settings
client_persistent_connections on
server_persistent_connections on

# Timeouts
connect_timeout 30 seconds
request_timeout 5 minutes
pconn_timeout 1 minute
read_timeout 15 minutes
```

### Cache Tuning

```bash
# Cache size and dirs
cache_dir aufs /var/spool/squid 20000 32 256

# Async I/O threads
cache_dir aufs /var/spool/squid 20000 32 256 max-size=1048576

# Memory pools
memory_pools on
memory_pools_limit 512 MB
```

## Monitoring & Logging

### Access Logs

```bash
# Standard log format
access_log /var/log/squid/access.log squid

# Combined log format
access_log /var/log/squid/access.log combined

# Custom log format
logformat custom %ts.%03tu %6tr %>a %Ss/%03>Hs %<st %rm %ru %[un %Sh/%<a %mt
access_log /var/log/squid/access.log custom

# Log rotation
logfile_rotate 10
```

### Monitoring Tools

```bash
# Real-time monitoring
tail -f /var/log/squid/access.log

# Cache stats
squidclient -p 3128 mgr:info

# Active connections
squidclient -p 3128 mgr:filedescriptors

# Memory usage
squidclient -p 3128 mgr:mem

# Cache statistics
squidclient -p 3128 mgr:storedir

# Access via browser
# http://localhost:3128/squid-internal-mgr/
```

### SNMP Monitoring

```bash
# Enable SNMP
snmp_port 3401
acl snmppublic snmp_community public
snmp_access allow snmppublic localhost
snmp_access deny all

# Query SNMP
snmpwalk -v2c -c public localhost:3401 .1.3.6.1.4.1.3495
```

## Common Use Cases

### Transparent Proxy

```bash
# Squid configuration
http_port 3129 intercept

# iptables rules
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 3129
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-port 3129
```

### Reverse Proxy

```bash
# Squid as reverse proxy
http_port 80 accel defaultsite=www.example.com
cache_peer backend.example.com parent 8080 0 no-query originserver

acl our_sites dstdomain .example.com
http_access allow our_sites
cache_peer_access backend.example.com allow our_sites
```

### Parent Proxy

```bash
# Forward to upstream proxy
cache_peer parent.proxy.com parent 3128 0 no-query default
never_direct allow all
```

## Best Practices

### Security

1. **Restrict Access**: Use ACLs to limit access
2. **Authentication**: Require authentication
3. **SSL Inspection**: Use SSL bumping carefully
4. **Updates**: Keep Squid updated
5. **Logging**: Monitor access logs

### Performance

1. **Cache Size**: Allocate sufficient cache
2. **Memory**: Provide adequate RAM
3. **Workers**: Use multiple workers on multi-core systems
4. **Async I/O**: Use aufs cache_dir
5. **Connection Pooling**: Enable persistent connections

### Maintenance

```bash
# Regular tasks
- Monitor logs
- Rotate logs
- Check cache utilization
- Review ACLs
- Update blacklists
- Test backup/restore
```

## Additional Resources

- [Official Squid Documentation](http://www.squid-cache.org/Doc/)
- [Squid Wiki](https://wiki.squid-cache.org/)
- [Squid Configuration Manual](http://www.squid-cache.org/Doc/config/)

---

*Last updated: 2025-11-16*
