# Red Hat Identity Management (IDM)

A comprehensive guide to Red Hat Identity Management - an integrated identity and authentication solution for Linux/Unix environments.

## Table of Contents

- [Introduction](#introduction)
- [Installation](#installation)
- [User Management](#user-management)
- [Group Management](#group-management)
- [Host Management](#host-management)
- [DNS Management](#dns-management)
- [Certificate Management](#certificate-management)
- [Authentication](#authentication)
- [RBAC & Delegation](#rbac--delegation)
- [Replication & HA](#replication--ha)

## Introduction

### What is Red Hat IDM?

Red Hat Identity Management (IdM) is an integrated solution combining:
- **389 Directory Server**: LDAP backend
- **MIT Kerberos**: Authentication
- **Dogtag Certificate System**: PKI/Certificates
- **SSSD**: Client authentication daemon
- **DNS**: Integrated DNS (optional)
- **NTP**: Time synchronization

### Features

- Centralized user/group management
- Single Sign-On (SSO) with Kerberos
- Certificate management and PKI
- Host-based access control (HBAC)
- Sudo rules
- SELinux user mapping
- Two-factor authentication
- Trusted Active Directory integration

## Installation

### Server Installation (RHEL/CentOS)

```bash
# Install packages
sudo dnf install -y ipa-server ipa-server-dns

# Configure hostname
sudo hostnamectl set-hostname idm.example.com

# Ensure DNS resolution
echo "192.168.1.100 idm.example.com idm" | sudo tee -a /etc/hosts

# Run installation
sudo ipa-server-install \
  --realm EXAMPLE.COM \
  --domain example.com \
  --ds-password DMPassword123 \
  --admin-password AdminPass123 \
  --hostname idm.example.com \
  --setup-dns \
  --forwarder 8.8.8.8 \
  --forwarder 8.8.4.4 \
  --no-ntp \
  --unattended

# Start services
sudo systemctl enable --now ipa

# Check status
sudo ipactl status

# Obtain Kerberos ticket
kinit admin
# Enter password: AdminPass123

# Verify
ipa user-find admin
```

### Installation Without DNS

```bash
sudo ipa-server-install \
  --realm EXAMPLE.COM \
  --domain example.com \
  --ds-password DMPassword123 \
  --admin-password AdminPass123 \
  --hostname idm.example.com \
  --no-ntp \
  --unattended
```

### Replica Installation

```bash
# On replica server
sudo dnf install -y ipa-server

# Configure hostname
sudo hostnamectl set-hostname idm-replica.example.com

# Join as replica
sudo ipa-replica-install \
  --principal admin \
  --admin-password AdminPass123 \
  --setup-ca \
  --setup-dns \
  --forwarder 8.8.8.8 \
  --unattended
```

### Client Installation

```bash
# Install client packages
sudo dnf install -y ipa-client

# Join to IdM domain
sudo ipa-client-install \
  --domain example.com \
  --realm EXAMPLE.COM \
  --server idm.example.com \
  --principal admin \
  --password AdminPass123 \
  --mkhomedir \
  --unattended

# Verify
id admin@example.com
klist
```

## User Management

### CLI User Management

```bash
# Authenticate
kinit admin

# Add user
ipa user-add jdoe \
  --first=John \
  --last=Doe \
  --email=jdoe@example.com \
  --password

# List users
ipa user-find

# Show user details
ipa user-show jdoe

# Modify user
ipa user-mod jdoe \
  --title="Senior Engineer" \
  --phone="+1-555-0100"

# Disable user
ipa user-disable jdoe

# Enable user
ipa user-enable jdoe

# Delete user
ipa user-del jdoe

# Set password
ipa passwd jdoe

# Force password change on next login
ipa user-mod jdoe --password-expiration=20251201000000Z

# Add SSH public key
ipa user-mod jdoe --sshpubkey="ssh-rsa AAAAB3..."
```

### Batch User Creation

```bash
# Create users from CSV
# users.csv format: username,firstname,lastname,email

while IFS=, read -r username first last email; do
  ipa user-add "$username" \
    --first="$first" \
    --last="$last" \
    --email="$email" \
    --password
done < users.csv
```

## Group Management

### Group Operations

```bash
# Create group
ipa group-add engineers \
  --desc="Engineering Team"

# Add users to group
ipa group-add-member engineers \
  --users=jdoe,asmith

# List groups
ipa group-find

# Show group details
ipa group-show engineers

# Remove user from group
ipa group-remove-member engineers \
  --users=jdoe

# Delete group
ipa group-del engineers

# Create group with GID
ipa group-add developers \
  --gid=10000 \
  --desc="Developers"

# Add nested groups
ipa group-add-member managers \
  --groups=engineers
```

### Group Types

```bash
# POSIX group (default)
ipa group-add posixgroup

# Non-POSIX group
ipa group-add --nonposix externalgroup

# External group (for AD users)
ipa group-add --external adgroup
```

## Host Management

### Host Operations

```bash
# Add host
ipa host-add server1.example.com \
  --ip-address=192.168.1.101

# List hosts
ipa host-find

# Show host details
ipa host-show server1.example.com

# Modify host
ipa host-mod server1.example.com \
  --desc="Web Server" \
  --location="Data Center 1"

# Delete host
ipa host-del server1.example.com

# Generate OTP for enrollment
ipa host-add server2.example.com \
  --password --random

# Add host to group
ipa hostgroup-add webservers
ipa hostgroup-add-member webservers \
  --hosts=server1.example.com,server2.example.com
```

### Host-Based Access Control (HBAC)

```bash
# Create HBAC rule
ipa hbacrule-add allow_ssh_to_servers \
  --desc="Allow SSH to servers"

# Add users
ipa hbacrule-add-user allow_ssh_to_servers \
  --users=jdoe,asmith

# Add hosts
ipa hbacrule-add-host allow_ssh_to_servers \
  --hosts=server1.example.com

# Add service
ipa hbacrule-add-service allow_ssh_to_servers \
  --hbacsvcs=sshd

# Enable rule
ipa hbacrule-enable allow_ssh_to_servers

# Test HBAC
ipa hbactest \
  --user=jdoe \
  --host=server1.example.com \
  --service=sshd

# List rules
ipa hbacrule-find

# Disable default allow_all rule
ipa hbacrule-disable allow_all
```

## DNS Management

### DNS Operations

```bash
# Add DNS zone
ipa dnszone-add example.com

# Add A record
ipa dnsrecord-add example.com server1 \
  --a-rec=192.168.1.101

# Add CNAME record
ipa dnsrecord-add example.com www \
  --cname-rec=server1.example.com.

# Add MX record
ipa dnsrecord-add example.com @ \
  --mx-rec="10 mail.example.com."

# Add TXT record
ipa dnsrecord-add example.com @ \
  --txt-rec="v=spf1 mx ~all"

# Show DNS record
ipa dnsrecord-show example.com server1

# Delete DNS record
ipa dnsrecord-del example.com server1

# Add reverse zone
ipa dnszone-add 1.168.192.in-addr.arpa

# Add PTR record
ipa dnsrecord-add 1.168.192.in-addr.arpa 101 \
  --ptr-rec=server1.example.com.
```

## Certificate Management

### Certificate Operations

```bash
# Request certificate for host
ipa-getcert request \
  -f /etc/pki/tls/certs/server.crt \
  -k /etc/pki/tls/private/server.key \
  -K host/server1.example.com \
  -D server1.example.com \
  -N CN=server1.example.com

# List certificates
ipa-getcert list

# Check certificate status
ipa-getcert status -i <request-id>

# Resubmit certificate request
ipa-getcert resubmit -i <request-id>

# Revoke certificate
ipa-getcert stop-tracking -i <request-id>

# Get CA certificate
ipa-getcert list-cas

# Request service certificate
ipa service-add HTTP/server1.example.com
ipa-getcert request \
  -f /etc/pki/tls/certs/http.crt \
  -k /etc/pki/tls/private/http.key \
  -K HTTP/server1.example.com
```

### CA Management

```bash
# Show CA configuration
ipa ca-show ipa

# Issue certificate manually
ipa cert-request cert.csr --principal host/server1.example.com

# Show certificate
ipa cert-show <serial-number>

# Revoke certificate
ipa cert-revoke <serial-number> --revocation-reason=4
```

## Authentication

### Kerberos

```bash
# Get Kerberos ticket
kinit username

# Get ticket with specific lifetime
kinit -l 7d username

# List tickets
klist

# Destroy tickets
kdestroy

# Renew ticket
kinit -R

# Check Kerberos configuration
ipa krb5-show-config

# Create keytab
ipa-getkeytab -s idm.example.com \
  -p host/server1.example.com \
  -k /etc/krb5.keytab

# Test Kerberos auth
kinit -kt /etc/krb5.keytab host/server1.example.com
```

### Two-Factor Authentication

```bash
# Enable OTP for user
ipa user-mod jdoe --user-auth-type=otp

# Add OTP token
ipa otptoken-add \
  --owner=jdoe \
  --type=totp \
  --desc="Mobile App"

# Show QR code for token
ipa otptoken-add --owner=jdoe --type=totp --qr-code

# List tokens
ipa otptoken-find --user=jdoe

# Disable token
ipa otptoken-disable <token-uuid>

# Delete token
ipa otptoken-del <token-uuid>

# Login with OTP
# Password: regularpassword123456 (password + OTP code)
```

### Password Policies

```bash
# Show default policy
ipa pwpolicy-show

# Create password policy for group
ipa pwpolicy-add engineers \
  --minlength=12 \
  --minclasses=3 \
  --maxlife=90 \
  --minlife=1 \
  --history=5 \
  --priority=10

# Modify password policy
ipa pwpolicy-mod engineers \
  --maxfail=3 \
  --failinterval=300 \
  --lockouttime=600
```

## RBAC & Delegation

### Role-Based Access Control

```bash
# Create role
ipa role-add helpdesk \
  --desc="Helpdesk Team"

# Add privileges to role
ipa role-add-privilege helpdesk \
  --privileges="User Administrators"

# Assign role to user
ipa role-add-member helpdesk \
  --users=jdoe

# Create custom privilege
ipa privilege-add "Reset Passwords" \
  --desc="Can reset user passwords"

# Add permission to privilege
ipa privilege-add-permission "Reset Passwords" \
  --permissions="System: Change User password"

# List roles
ipa role-find

# Show role details
ipa role-show helpdesk
```

### Sudo Rules

```bash
# Create sudo rule
ipa sudorule-add allow_admin_sudo \
  --desc="Allow admins full sudo"

# Add users to sudo rule
ipa sudorule-add-user allow_admin_sudo \
  --users=jdoe

# Add hosts
ipa sudorule-add-host allow_admin_sudo \
  --hosts=server1.example.com

# Add commands
ipa sudorule-add-allow-command allow_admin_sudo \
  --sudocmds=ALL

# Enable sudo rule
ipa sudorule-enable allow_admin_sudo

# Test sudo rule
sudo -l -U jdoe
```

## Replication & HA

### Managing Replicas

```bash
# List replication agreements
ipa-replica-manage list

# Check replication status
ipa-replica-manage list --verbose

# Initialize replica
ipa-replica-manage re-initialize \
  --from idm.example.com

# Force sync
ipa-replica-manage force-sync \
  --from idm.example.com

# Remove replica
ipa-replica-manage del idm-replica.example.com

# List CA replicas
ipa-csreplica-manage list
```

### Topology Management

```bash
# Show replication topology
ipa topologysegment-find domain

# Add topology segment
ipa topologysegment-add domain \
  idm-to-replica \
  --left idm.example.com \
  --right idm-replica.example.com

# Delete topology segment
ipa topologysegment-del domain idm-to-replica
```

## Additional Resources

- [Red Hat IDM Documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/planning_identity_management/)
- [FreeIPA (Upstream)](https://www.freeipa.org/page/Documentation)
- [RHEL Identity Management Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_identity_management/)

---

*Last updated: 2025-11-16*
