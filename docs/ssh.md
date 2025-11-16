# SSH (Secure Shell)

A comprehensive guide to SSH - secure network protocol for remote system access and management.

## Table of Contents

- [Introduction](#introduction)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [SSH Keys](#ssh-keys)
- [Configuration](#configuration)
- [Port Forwarding](#port-forwarding)
- [ProxyJump & Bastion](#proxyjump--bastion)
- [File Transfer](#file-transfer)
- [Security](#security)
- [Troubleshooting](#troubleshooting)

## Introduction

### What is SSH?

SSH (Secure Shell) is a cryptographic network protocol for secure data communication, remote command execution, and network services over an unsecured network.

### Features

- **Encrypted Communication**: All data encrypted
- **Authentication**: Password and public key authentication
- **Port Forwarding**: Tunnel other protocols through SSH
- **File Transfer**: SCP, SFTP
- **X11 Forwarding**: Run GUI applications remotely
- **Agent Forwarding**: Use local keys on remote servers

## Installation

### Linux

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install openssh-client openssh-server

# RHEL/CentOS/Fedora
sudo dnf install openssh-clients openssh-server

# Enable and start SSH server
sudo systemctl enable sshd
sudo systemctl start sshd

# Check status
sudo systemctl status sshd
```

### macOS

```bash
# SSH client pre-installed

# Enable SSH server
sudo systemsetup -setremotelogin on

# Disable SSH server
sudo systemsetup -setremotelogin off
```

### Windows

```powershell
# Windows 10/11 (OpenSSH Client)
# Add-WindowsCapability -Online -Name OpenSSH.Client

# OpenSSH Server
Add-WindowsCapability -Online -Name OpenSSH.Server
Start-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'
```

## Basic Usage

### Connect to Remote Host

```bash
# Basic connection
ssh username@hostname

# Specify port
ssh -p 2222 username@hostname

# Specify identity file
ssh -i ~/.ssh/id_rsa username@hostname

# Run command remotely
ssh username@hostname 'ls -la'

# Interactive shell with command
ssh -t username@hostname 'sudo command'

# Verbose output (debugging)
ssh -v username@hostname
ssh -vv username@hostname  # More verbose
ssh -vvv username@hostname # Maximum verbosity

# Disconnect (type in SSH session)
~.
```

### First Connection

```bash
# First time connecting shows fingerprint
The authenticity of host 'example.com (192.168.1.100)' can't be established.
ED25519 key fingerprint is SHA256:xxxxx.
Are you sure you want to continue connecting (yes/no)?

# Verify fingerprint on server
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

## SSH Keys

### Generate SSH Key Pair

```bash
# RSA key (2048-bit)
ssh-keygen -t rsa -b 2048

# RSA key (4096-bit)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Ed25519 key (recommended)
ssh-keygen -t ed25519 -C "your_email@example.com"

# ECDSA key
ssh-keygen -t ecdsa -b 521

# Specify file location
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_custom

# Generate without passphrase (not recommended)
ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_ed25519
```

### Copy Public Key to Server

```bash
# Using ssh-copy-id
ssh-copy-id username@hostname

# Specify key file
ssh-copy-id -i ~/.ssh/id_ed25519.pub username@hostname

# Specify port
ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 2222 username@hostname

# Manual copy
cat ~/.ssh/id_ed25519.pub | ssh username@hostname 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'

# Set permissions
ssh username@hostname 'chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys'
```

### Manage SSH Keys

```bash
# List keys in ssh-agent
ssh-add -l

# Add key to ssh-agent
ssh-add ~/.ssh/id_ed25519

# Add with timeout (1 hour)
ssh-add -t 3600 ~/.ssh/id_ed25519

# Remove key from agent
ssh-add -d ~/.ssh/id_ed25519

# Remove all keys
ssh-add -D

# Change key passphrase
ssh-keygen -p -f ~/.ssh/id_ed25519

# Show public key from private key
ssh-keygen -y -f ~/.ssh/id_ed25519

# Convert key formats
ssh-keygen -i -f old_key > new_key  # Import
ssh-keygen -e -f key > key.pem      # Export
```

## Configuration

### Client Configuration (~/.ssh/config)

```bash
# Host-specific configuration
Host myserver
    HostName 192.168.1.100
    User admin
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

# Wildcard matching
Host *.example.com
    User deployment
    IdentityFile ~/.ssh/deploy_key

# Default settings
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    Compression yes
    ForwardAgent no
    AddKeysToAgent yes

# Multiple hosts
Host server1 server2 server3
    User admin
    IdentityFile ~/.ssh/admin_key

# Use configuration
ssh myserver  # Connects using config above
```

### Common Configuration Options

```bash
# ~/.ssh/config options

Host myhost
    # Connection
    HostName example.com
    Port 22
    User username

    # Authentication
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
    PubkeyAuthentication yes
    PasswordAuthentication no

    # Connection management
    ServerAliveInterval 60
    ServerAliveCountMax 3
    TCPKeepAlive yes

    # Forwarding
    ForwardAgent yes
    ForwardX11 yes
    LocalForward 8080 localhost:80
    RemoteForward 9090 localhost:3000
    DynamicForward 1080

    # Proxy/Jump
    ProxyJump bastion
    ProxyCommand ssh -W %h:%p bastion

    # Ciphers and algorithms
    Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
    MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
    KexAlgorithms curve25519-sha256,diffie-hellman-group-exchange-sha256

    # Misc
    Compression yes
    LogLevel INFO
    StrictHostKeyChecking ask
    UserKnownHostsFile ~/.ssh/known_hosts
```

### Server Configuration (/etc/ssh/sshd_config)

```bash
# Basic settings
Port 22
ListenAddress 0.0.0.0
Protocol 2

# Authentication
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no
UsePAM yes

# Key files
AuthorizedKeysFile .ssh/authorized_keys
HostKey /etc/ssh/ssh_host_ed25519_key
HostKey /etc/ssh/ssh_host_rsa_key

# Security
MaxAuthTries 3
MaxSessions 10
LoginGraceTime 60
ClientAliveInterval 300
ClientAliveCountMax 2

# Forwarding
AllowTcpForwarding yes
X11Forwarding yes
AllowAgentForwarding yes

# Logging
SyslogFacility AUTH
LogLevel VERBOSE

# Subsystems
Subsystem sftp /usr/lib/openssh/sftp-server

# Restart after changes
sudo systemctl restart sshd
```

## Port Forwarding

### Local Port Forwarding

```bash
# Forward local port to remote
ssh -L local_port:remote_host:remote_port username@ssh_server

# Example: Access remote database
ssh -L 5432:localhost:5432 username@db-server
# Connect to localhost:5432 locally

# Multiple forwards
ssh -L 8080:web:80 -L 5432:db:5432 username@server

# Keep connection open
ssh -N -L 8080:localhost:80 username@server
```

### Remote Port Forwarding

```bash
# Forward remote port to local
ssh -R remote_port:local_host:local_port username@ssh_server

# Example: Share local web server
ssh -R 8080:localhost:3000 username@remote-server
# Access via remote-server:8080

# Bind to all interfaces on remote
ssh -R 0.0.0.0:8080:localhost:3000 username@server
```

### Dynamic Port Forwarding (SOCKS Proxy)

```bash
# Create SOCKS proxy
ssh -D 1080 username@server

# Configure browser to use SOCKS proxy:
# - SOCKS Host: localhost
# - Port: 1080
# - SOCKS v5

# With autossh (auto-reconnect)
autossh -M 0 -D 1080 -N username@server
```

## ProxyJump & Bastion

### ProxyJump (SSH Jump Host)

```bash
# Jump through bastion
ssh -J bastion-user@bastion.example.com target-user@target-server

# Multiple jumps
ssh -J jump1,jump2,jump3 username@target

# In config file
Host target
    HostName target-server
    User target-user
    ProxyJump bastion-user@bastion.example.com

# Then simply:
ssh target
```

### ProxyCommand

```bash
# Using ProxyCommand
Host target
    HostName 10.0.1.100
    User admin
    ProxyCommand ssh -W %h:%p bastion

# With netcat
Host target
    ProxyCommand ssh bastion nc %h %p

# Through SOCKS proxy
Host target
    ProxyCommand nc -X 5 -x localhost:1080 %h %p
```

## File Transfer

### SCP (Secure Copy)

```bash
# Copy file to remote
scp file.txt username@server:/remote/path/

# Copy from remote
scp username@server:/remote/file.txt /local/path/

# Copy directory recursively
scp -r directory/ username@server:/remote/path/

# Specify port
scp -P 2222 file.txt username@server:/path/

# Preserve attributes
scp -p file.txt username@server:/path/

# Limit bandwidth (KB/s)
scp -l 1024 file.txt username@server:/path/

# Through jump host
scp -J bastion file.txt username@target:/path/

# Copy between remote hosts
scp user1@host1:/file user2@host2:/path/
```

### SFTP (SSH File Transfer Protocol)

```bash
# Connect to SFTP server
sftp username@hostname

# SFTP commands
sftp> ls                 # List remote files
sftp> lls                # List local files
sftp> cd /remote/path    # Change remote directory
sftp> lcd /local/path    # Change local directory
sftp> get file.txt       # Download file
sftp> get -r directory/  # Download directory
sftp> put file.txt       # Upload file
sftp> put -r directory/  # Upload directory
sftp> mkdir newdir       # Create remote directory
sftp> rm file.txt        # Delete remote file
sftp> rmdir directory    # Remove remote directory
sftp> pwd                # Print remote working directory
sftp> lpwd               # Print local working directory
sftp> quit               # Exit

# Batch mode
sftp -b commands.txt username@server

# commands.txt:
cd /remote/path
get file1.txt
put file2.txt
quit
```

### rsync over SSH

```bash
# Sync directory to remote
rsync -avz -e ssh /local/path/ username@server:/remote/path/

# Sync from remote
rsync -avz -e ssh username@server:/remote/path/ /local/path/

# Delete files on destination that don't exist on source
rsync -avz --delete -e ssh /local/path/ username@server:/remote/path/

# Show progress
rsync -avz --progress -e ssh /local/path/ username@server:/remote/path/

# Dry run
rsync -avz --dry-run -e ssh /local/path/ username@server:/remote/path/

# Specify port
rsync -avz -e "ssh -p 2222" /local/path/ username@server:/remote/path/

# Through jump host
rsync -avz -e "ssh -J bastion" /local/path/ username@target:/remote/path/
```

## Security

### SSH Hardening

```bash
# /etc/ssh/sshd_config

# Disable root login
PermitRootLogin no

# Disable password authentication
PasswordAuthentication no
PubkeyAuthentication yes

# Limit users
AllowUsers alice bob
DenyUsers eve

# Limit groups
AllowGroups sshusers

# Change default port
Port 2222

# Disable X11 forwarding (if not needed)
X11Forwarding no

# Disable agent forwarding
AllowAgentForwarding no

# Use strong ciphers only
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
KexAlgorithms curve25519-sha256,diffie-hellman-group-exchange-sha256

# Rate limiting
MaxAuthTries 3
MaxStartups 10:30:60

# Idle timeout
ClientAliveInterval 300
ClientAliveCountMax 2
```

### Fail2Ban

```bash
# Install fail2ban
sudo apt install fail2ban

# Configure for SSH
sudo nano /etc/fail2ban/jail.local

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600

# Restart fail2ban
sudo systemctl restart fail2ban

# Check status
sudo fail2ban-client status sshd
```

### Two-Factor Authentication

```bash
# Install Google Authenticator
sudo apt install libpam-google-authenticator

# Configure for user
google-authenticator

# Edit /etc/pam.d/sshd
auth required pam_google_authenticator.so

# Edit /etc/ssh/sshd_config
ChallengeResponseAuthentication yes
AuthenticationMethods publickey,keyboard-interactive

# Restart SSH
sudo systemctl restart sshd
```

## Troubleshooting

### Common Issues

```bash
# Permission denied (publickey)
# Fix permissions:
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_*
chmod 644 ~/.ssh/id_*.pub

# Connection timeout
# Check firewall
sudo ufw status
sudo ufw allow 22/tcp

# Check SSH service
sudo systemctl status sshd

# Check listening ports
sudo netstat -tulpn | grep ssh

# Too many authentication failures
# Limit keys tried
ssh -o IdentitiesOnly=yes -i ~/.ssh/specific_key user@host

# Host key verification failed
# Remove old host key
ssh-keygen -R hostname

# Or edit known_hosts
nano ~/.ssh/known_hosts

# Debug connection
ssh -vvv username@hostname

# Check server logs
sudo tail -f /var/log/auth.log  # Debian/Ubuntu
sudo tail -f /var/log/secure    # RHEL/CentOS
```

### Performance Issues

```bash
# Disable DNS lookup (server-side)
UseDNS no

# Enable compression
Compression yes

# Use faster cipher
Ciphers chacha20-poly1305@openssh.com

# Disable GSSAPI
GSSAPIAuthentication no

# Reuse connections
ControlMaster auto
ControlPath ~/.ssh/sockets/%r@%h-%p
ControlPersist 600
```

## Additional Resources

- [OpenSSH Documentation](https://www.openssh.com/manual.html)
- [SSH Academy](https://www.ssh.com/academy/ssh)
- [Mozilla SSH Guidelines](https://infosec.mozilla.org/guidelines/openssh)
- [SSH Audit](https://github.com/jtesta/ssh-audit)

---

*Last updated: 2025-11-16*
