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

## Advanced Configuration Examples

### Complex ACL Examples

```bash
# Time-based access control
acl morning time 08:00-12:00
acl afternoon time 12:00-18:00
acl business_hours time MTWHF 08:00-18:00
acl weekends time SA 00:00-23:59

# Social media blocking during work hours
acl social_media dstdomain .facebook.com .twitter.com .instagram.com .tiktok.com
http_access deny social_media business_hours

# Bandwidth-heavy sites only after hours
acl streaming dstdomain .youtube.com .netflix.com .hulu.com .twitch.tv
http_access deny streaming business_hours
http_access allow streaming

# User-agent based filtering
acl mobile_devices browser -i mobile android iphone ipad
acl download_managers browser -i download wget curl

# Block download managers
http_access deny download_managers

# MIME type filtering
acl video_content rep_mime_type video/.*
acl audio_content rep_mime_type audio/.*

# Block video during work hours
http_reply_access deny video_content business_hours

# IP range based access
acl management_ips src 192.168.1.1-192.168.1.50
acl employee_ips src 192.168.1.51-192.168.1.200
acl guest_ips src 192.168.2.0/24

# Management has unrestricted access
http_access allow management_ips

# Employees have filtered access
acl work_sites dstdomain "/etc/squid/allowed_sites.txt"
http_access allow employee_ips work_sites

# Guests only to specific sites
acl guest_sites dstdomain .google.com .wikipedia.org
http_access allow guest_ips guest_sites

# URL regex examples
acl ads url_regex -i "/etc/squid/ad_block.regex"
acl malware url_regex -i "/etc/squid/malware_domains.regex"
acl webmail url_regex -i mail\.google\.com mail\.yahoo\.com outlook\.live\.com

# Block patterns
http_access deny ads
http_access deny malware

# Port-based ACLs
acl dangerous_ports port 23 telnet
acl database_ports port 3306 5432 1521
http_access deny dangerous_ports
http_access deny CONNECT database_ports

# Request header ACLs
acl no_referer req_header Referer ^$
acl suspicious_agents browser -i "nmap" "masscan" "nikto"
http_access deny suspicious_agents
```

### Bandwidth Management

```bash
# Multiple delay pools for different user groups

# Pool 1: Management (unlimited)
delay_pools 3

# Pool 1 class (per-host)
delay_class 1 2
delay_parameters 1 -1/-1 -1/-1  # Unlimited
delay_access 1 allow management_ips
delay_access 1 deny all

# Pool 2: Employees (limited)
delay_class 2 2
# Network: unlimited, Per-host: 1MB/s (125KB/s)
delay_parameters 2 -1/-1 125000/125000
delay_access 2 allow employee_ips
delay_access 2 deny all

# Pool 3: Guests (heavily limited)
delay_class 3 2
# Network: 5MB/s total, Per-host: 256KB/s
delay_parameters 3 625000/625000 32000/32000
delay_access 3 allow guest_ips
delay_access 3 deny all

# Rate limiting for large downloads
acl large_files url_regex -i \.iso$ \.zip$ \.tar\.gz$ \.exe$ \.dmg$
delay_pools 4
delay_class 4 1
delay_parameters 4 512000/512000  # 512KB/s max for large files
delay_access 4 allow large_files
```

### Content Filtering Rules

```bash
# /etc/squid/squid.conf

# Adult content filtering
acl adult_keywords url_regex -i "/etc/squid/adult_keywords.txt"
http_access deny adult_keywords

# File type restrictions
acl executable_files urlpath_regex -i \.exe$
acl compressed_files urlpath_regex -i \.zip$ \.rar$ \.7z$ \.tar\.gz$
acl installer_files urlpath_regex -i \.msi$ \.dmg$ \.pkg$ \.deb$ \.rpm$

# Block executables for non-IT staff
acl it_staff src 192.168.1.10-192.168.1.20
http_access deny executable_files !it_staff
http_access deny installer_files !it_staff

# Max download size (100MB)
reply_body_max_size 100 MB

# Max upload size (50MB)
request_body_max_size 50 MB

# Allow larger sizes for specific users
acl power_users src "/etc/squid/power_users.txt"
reply_body_max_size 0 power_users
request_body_max_size 0 power_users
```

### Advanced SSL Bumping Configuration

```bash
# /etc/squid/squid.conf

# SSL Bump configuration
http_port 3128 ssl-bump \
  cert=/etc/squid/ssl/myca.pem \
  generate-host-certificates=on \
  dynamic_cert_mem_cache_size=16MB \
  options=NO_SSLv3,NO_TLSv1,NO_TLSv1_1,SINGLE_DH_USE,SINGLE_ECDH_USE \
  cipher=HIGH:MEDIUM:!LOW:!RC4:!SEED:!IDEA:!3DES:!MD5:!EXP:!PSK:!DSS

sslcrtd_program /usr/lib/squid/security_file_certgen -s /var/spool/squid/ssl_db -M 16MB

# SSL bump rules
acl step1 at_step SslBump1
acl step2 at_step SslBump2
acl step3 at_step SslBump3

# Never bump financial sites
acl financial_sites ssl::server_name .bank.com .paypal.com .stripe.com
acl financial_sites ssl::server_name .bankofamerica.com .chase.com .wellsfargo.com

# Never bump sites with client certificates
acl client_cert_sites ssl::server_name .military.gov .client-cert-required.com

# Never bump specific applications
acl apps_no_bump ssl::server_name .apple.com .icloud.com
acl apps_no_bump ssl::server_name .microsoft.com .windowsupdate.com
acl apps_no_bump ssl::server_name .google.com .youtube.com

# Splice (don't bump) specific sites
ssl_bump splice financial_sites
ssl_bump splice client_cert_sites
ssl_bump splice apps_no_bump

# Peek at step 1 to get SNI
ssl_bump peek step1

# Bump everything else
ssl_bump bump all

# SSL errors handling
sslproxy_cert_error allow all
sslproxy_flags DONT_VERIFY_PEER

# SSL options
sslproxy_options NO_SSLv3,NO_TLSv1,NO_TLSv1_1,SINGLE_DH_USE

# OCSP stapling
sslproxy_cert_adapt setValidAfter all
sslproxy_cert_adapt setValidBefore all
```

### Detailed Caching Configuration

```bash
# Multiple cache directories for different content types

# Fast SSD cache for small objects
cache_dir aufs /cache/ssd 50000 32 256 min-size=0 max-size=1048576

# Large HDD cache for big objects
cache_dir aufs /cache/hdd 200000 64 256 min-size=1048577

# Memory cache
cache_mem 2048 MB
maximum_object_size_in_memory 1 MB
memory_cache_mode always

# Cache sizes
maximum_object_size 512 MB
minimum_object_size 0 KB

# Caching rules
refresh_pattern ^ftp:           1440    20%     10080
refresh_pattern ^gopher:        1440    0%      1440
refresh_pattern -i (/cgi-bin/|\?) 0     0%      0
refresh_pattern -i \.(html|htm)$  1440  40%     40320
refresh_pattern -i \.(css|js)$    10080 90%     43200
refresh_pattern -i \.(jpg|jpeg|png|gif|bmp|ico)$ 10080 90% 43200 reload-into-ims
refresh_pattern -i \.(zip|gz|arj|lha|lzh)$ 10080 100% 43200 override-expire override-lastmod ignore-reload
refresh_pattern -i \.(rar|tgz|tar|exe|bin)$ 10080 100% 43200 override-expire override-lastmod ignore-reload
refresh_pattern -i \.(pdf|rtf|doc|docx|xls|xlsx|ppt|pptx)$ 10080 90% 43200 reload-into-ims
refresh_pattern -i \.(mp3|mp4|mpeg|avi|mov|wmv|flv)$ 10080 90% 43200 reload-into-ims ignore-reload override-expire override-lastmod
refresh_pattern .               0       20%     4320

# Cache replacement policies
cache_replacement_policy heap LFUDA
memory_replacement_policy heap GDSF

# Quick abort settings
quick_abort_min 0 KB
quick_abort_max 0 KB
quick_abort_pct 95

# Range offset limit
range_offset_limit 10 MB

# Store directory options
store_dir_select_algorithm least-load

# Cache peer sharing
cache_peer_access deny all

# Don't cache specific content
acl dynamic_content urlpath_regex cgi-bin asp aspx php jsp
acl dynamic_content urlpath_regex \?
cache deny dynamic_content

# Don't cache POST requests
acl POST method POST
cache deny POST

# Cache hierarchy
hierarchy_stoplist cgi-bin ? asp aspx php jsp

# ICP settings (for cache siblings)
icp_port 3130
icp_access allow localnet
icp_access deny all
```

### Comprehensive Logging

```bash
# Access log with all details
logformat detailed %ts.%03tu %6tr %>a %Ss/%03>Hs %<st %rm %ru %[un %Sh/%<a %mt "%{User-Agent}>h" "%{Referer}>h"
access_log daemon:/var/log/squid/access.log detailed

# Separate logs for different purposes
logformat common %>a %[ui %[un [%tl] "%rm %ru HTTP/%rv" %>Hs %<st %Ss:%Sh
logformat combined %>a %[ui %[un [%tl] "%rm %ru HTTP/%rv" %>Hs %<st "%{Referer}>h" "%{User-Agent}>h" %Ss:%Sh

# HTTPS inspection log
logformat ssllog %ts.%03tu %6tr %>a %Ss/%03>Hs %<st %rm %ru %ssl::>cert_subject %ssl::>cert_issuer

# Denied requests log
logformat denied %ts %6tr %>a %Ss/%03>Hs %rm %ru %[un

# Specific logs
access_log daemon:/var/log/squid/https.log ssllog ssl::server_name=*
access_log daemon:/var/log/squid/denied.log denied http_status=403

# Cache log details
cache_log /var/log/squid/cache.log
cache_store_log /var/log/squid/store.log

# Debug logging (temporary)
# debug_options ALL,1 33,2 28,9

# Log rotation
logfile_rotate 30

# MIME table
mime_table /etc/squid/mime.conf

# Error pages customization
error_directory /usr/share/squid/errors/en
err_html_text "Contact IT Support at support@example.com"
```

### Enterprise Authentication Setup

```bash
# Multiple authentication methods

# Kerberos/NTLM for Windows domain
auth_param negotiate program /usr/lib/squid/negotiate_kerberos_auth -s HTTP/squid.example.com@EXAMPLE.COM
auth_param negotiate children 20
auth_param negotiate keep_alive on

# NTLM fallback
auth_param ntlm program /usr/lib/squid/ntlm_fake_auth
auth_param ntlm children 10

# Basic auth for non-domain users
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwords
auth_param basic children 5
auth_param basic realm Squid Proxy Authentication
auth_param basic credentialsttl 8 hours

# LDAP authentication
auth_param basic program /usr/lib/squid/basic_ldap_auth \
  -b "ou=people,dc=example,dc=com" \
  -D "cn=proxyuser,dc=example,dc=com" \
  -W /etc/squid/ldap_password \
  -f "(&(uid=%s)(objectClass=person))" \
  -h ldap.example.com

# External ACL for group membership
external_acl_type ldap_group ttl=3600 negative_ttl=3600 %LOGIN /usr/lib/squid/ext_ldap_group_acl \
  -b "ou=groups,dc=example,dc=com" \
  -D "cn=proxyuser,dc=example,dc=com" \
  -W /etc/squid/ldap_password \
  -f "(&(member=%u)(objectClass=groupOfNames))" \
  -h ldap.example.com

# ACLs for authenticated users
acl authenticated proxy_auth REQUIRED
acl admin_group external ldap_group admins
acl users_group external ldap_group users

# Access rules based on groups
http_access allow admin_group
http_access allow users_group work_sites
http_access deny all
```

### WPAD (Web Proxy Auto-Discovery)

```bash
# DNS WPAD setup
# Add to DNS:
# wpad.example.com. IN A 192.168.1.10

# Create /var/www/html/wpad.dat
function FindProxyForURL(url, host) {
    // Direct connection for local addresses
    if (isInNet(host, "192.168.0.0", "255.255.0.0") ||
        isInNet(host, "10.0.0.0", "255.0.0.0") ||
        isInNet(host, "127.0.0.1", "255.0.0.0") ||
        isPlainHostName(host)) {
        return "DIRECT";
    }

    // Bypass proxy for specific domains
    if (dnsDomainIs(host, ".local") ||
        dnsDomainIs(host, ".internal.company.com")) {
        return "DIRECT";
    }

    // Use proxy with failover
    return "PROXY proxy1.example.com:3128; PROXY proxy2.example.com:3128; DIRECT";
}

# Apache config for WPAD
<VirtualHost *:80>
    ServerName wpad.example.com
    DocumentRoot /var/www/html

    <Location /wpad.dat>
        ForceType application/x-ns-proxy-autoconfig
        Header set Cache-Control "max-age=3600"
    </Location>
</VirtualHost>
```

### Load Balancing Configuration

```bash
# Multiple Squid instances for load balancing

# Master squid.conf
cache_peer proxy1.local sibling 3128 3130 proxy-only
cache_peer proxy2.local sibling 3128 3130 proxy-only
cache_peer proxy3.local sibling 3128 3130 proxy-only

# Round-robin selection
cache_peer_access proxy1.local allow all
cache_peer_access proxy2.local allow all
cache_peer_access proxy3.local allow all

# ICP query for peers
icp_port 3130
icp_access allow localnet

# Peer selection method
cache_peer_domain proxy1.local .com
cache_peer_domain proxy2.local .net .org
cache_peer_domain proxy3.local .edu .gov

# Reliability
dead_peer_timeout 10 seconds
```

### Monitoring Scripts

```bash
#!/bin/bash
# /usr/local/bin/squid-monitor.sh

# Check Squid status
check_squid_status() {
    if ! systemctl is-active --quiet squid; then
        echo "ERROR: Squid is not running"
        systemctl restart squid
    fi
}

# Check cache usage
check_cache_usage() {
    CACHE_USAGE=$(squidclient -p 3128 mgr:storedir | grep "Store Directory" -A 10 | grep "Current Size" | awk '{print $3}')
    CACHE_MAX=100000000  # 100GB in KB

    if [ $CACHE_USAGE -gt $CACHE_MAX ]; then
        echo "WARNING: Cache usage high: $CACHE_USAGE KB"
    fi
}

# Check memory usage
check_memory() {
    MEMORY=$(squidclient -p 3128 mgr:mem | grep "Total accounted" | awk '{print $3}')
    echo "Memory usage: $MEMORY"
}

# Active connections
check_connections() {
    CONNECTIONS=$(squidclient -p 3128 mgr:info | grep "Number of clients" | awk '{print $5}')
    echo "Active connections: $CONNECTIONS"
}

# Hit ratio
check_hit_ratio() {
    squidclient -p 3128 mgr:info | grep -A 2 "Request Hit Ratios"
}

# Run checks
check_squid_status
check_cache_usage
check_memory
check_connections
check_hit_ratio
```

### Log Analysis Examples

```bash
# Top accessed domains
awk '{print $7}' /var/log/squid/access.log | sed 's|http[s]*://||' | cut -d'/' -f1 | sort | uniq -c | sort -rn | head -20

# Top users by bandwidth
awk '{user=$8; bytes=$5; total[user]+=bytes} END {for (u in total) printf "%s\t%d MB\n", u, total[u]/1024/1024}' /var/log/squid/access.log | sort -k2 -rn

# Denied requests
grep "TCP_DENIED" /var/log/squid/access.log | awk '{print $7}' | sort | uniq -c | sort -rn

# HTTP status codes
awk '{print $4}' /var/log/squid/access.log | cut -d'/' -f2 | sort | uniq -c | sort -rn

# Cache hits vs misses
awk '{print $4}' /var/log/squid/access.log | cut -d'/' -f1 | sort | uniq -c

# Bandwidth by hour
awk '{split($1,a,"."); hour=strftime("%H",a[1]); bytes=$5; total[hour]+=bytes} END {for (h in total) printf "%s:00 - %d MB\n", h, total[h]/1024/1024}' /var/log/squid/access.log | sort

# Top file types downloaded
awk '{print $7}' /var/log/squid/access.log | sed 's/.*\.//' | sort | uniq -c | sort -rn | head -20

# Slow requests (>1 second)
awk '{if ($2 > 1000) print $0}' /var/log/squid/access.log

# Failed requests
awk '{if ($4 ~ /TCP_MISS.*4[0-9][0-9]/ || $4 ~ /TCP_MISS.*5[0-9][0-9]/) print $0}' /var/log/squid/access.log
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
