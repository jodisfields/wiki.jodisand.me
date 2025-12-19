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

## Advanced Configuration Examples

### Complex HBAC Rules

```bash
# Create comprehensive HBAC rule for web servers
ipa hbacrule-add webserver_ssh_access \
  --desc="SSH access to web servers for ops team during business hours"

# Add specific users
ipa hbacrule-add-user webserver_ssh_access \
  --users=jdoe,asmith

# Add user groups
ipa hbacrule-add-user webserver_ssh_access \
  --groups=ops-team,sre-team

# Add specific hosts
ipa hbacrule-add-host webserver_ssh_access \
  --hosts=web1.example.com,web2.example.com

# Add host groups
ipa hbacrule-add-host webserver_ssh_access \
  --hostgroups=webservers,frontends

# Add services
ipa hbacrule-add-service webserver_ssh_access \
  --hbacsvcs=sshd,sudo

# Enable the rule
ipa hbacrule-enable webserver_ssh_access

# Test the rule
ipa hbactest \
  --user=jdoe \
  --host=web1.example.com \
  --service=sshd \
  --rules=webserver_ssh_access

# Create time-based access rule (requires external script)
# Example: Only allow access during business hours
# This is implemented via sudo rules and cron
```

### Advanced Sudo Rules Configuration

```bash
# Create sudo rule for emergency root access
ipa sudorule-add emergency_root \
  --desc="Emergency full root access for senior admins"

# Add users
ipa sudorule-add-user emergency_root \
  --users=admin1,admin2

# Add user groups
ipa sudorule-add-user emergency_root \
  --groups=senior-admins

# Add hosts (all hosts)
ipa sudorule-add-host emergency_root \
  --hosts=ALL

# Allow all commands
ipa sudorule-add-allow-command emergency_root \
  --sudocmds=ALL

# Set run-as user
ipa sudorule-mod emergency_root \
  --runasusercat=all \
  --runasgroupcat=all

# Require sudo password
ipa sudorule-add-option emergency_root \
  --sudooption='!authenticate'

---

# Create restricted sudo rule for web server restart
ipa sudorule-add restart_apache \
  --desc="Allow web admins to restart Apache"

# Add group
ipa sudorule-add-user restart_apache \
  --groups=web-admins

# Add hostgroup
ipa sudorule-add-host restart_apache \
  --hostgroups=webservers

# Create sudo command
ipa sudocmd-add '/usr/bin/systemctl restart httpd'
ipa sudocmd-add '/usr/bin/systemctl status httpd'
ipa sudocmd-add '/usr/bin/systemctl reload httpd'

# Create sudo command group
ipa sudocmdgroup-add apache_management
ipa sudocmdgroup-add-member apache_management \
  --sudocmds='/usr/bin/systemctl restart httpd' \
  --sudocmds='/usr/bin/systemctl status httpd' \
  --sudocmds='/usr/bin/systemctl reload httpd'

# Add command group to rule
ipa sudorule-add-allow-command restart_apache \
  --sudocmdgroups=apache_management

# No password required for these commands
ipa sudorule-add-option restart_apache \
  --sudooption='!authenticate'

---

# Create sudo rule for package management
ipa sudorule-add package_management \
  --desc="Allow sysadmins to manage packages"

ipa sudorule-add-user package_management \
  --groups=sysadmins

ipa sudorule-add-host package_management \
  --hostgroups=production-servers

# Add DNF/YUM commands
ipa sudocmd-add '/usr/bin/dnf *'
ipa sudocmd-add '/usr/bin/yum *'
ipa sudocmd-add '/usr/bin/rpm *'

ipa sudocmdgroup-add package_mgmt_cmds
ipa sudocmdgroup-add-member package_mgmt_cmds \
  --sudocmds='/usr/bin/dnf *' \
  --sudocmds='/usr/bin/yum *' \
  --sudocmds='/usr/bin/rpm *'

ipa sudorule-add-allow-command package_management \
  --sudocmdgroups=package_mgmt_cmds

# Test sudo rules
sudo -l -U jdoe
```

### Multi-Tier User Provisioning

```bash
# Script for bulk user creation with different roles
#!/bin/bash

# Create organizational groups
ipa group-add --desc="Engineering Department" engineering
ipa group-add --desc="Operations Department" operations
ipa group-add --desc="DevOps Team" devops
ipa group-add --desc="Security Team" security
ipa group-add --desc="Database Administrators" dba

# Create role-based groups
ipa group-add --desc="Senior Engineers" senior-engineers
ipa group-add --desc="Junior Engineers" junior-engineers
ipa group-add --desc="Team Leads" team-leads

# Create users with department assignment
create_user() {
  local username=$1
  local first=$2
  local last=$3
  local email=$4
  local department=$5
  local role=$6
  local manager=$7

  ipa user-add "$username" \
    --first="$first" \
    --last="$last" \
    --email="$email" \
    --title="$role" \
    --manager="$manager" \
    --orgunit="$department" \
    --phone="+1-555-0100" \
    --street="123 Main St" \
    --city="San Francisco" \
    --state="CA" \
    --postalcode="94105" \
    --password

  # Add to department group
  ipa group-add-member "$department" --users="$username"

  # Set password expiration policy
  if [ "$role" == "Senior Engineer" ]; then
    ipa pwpolicy-add senior-engineers \
      --maxlife=180 \
      --minlife=1 \
      --history=10 \
      --minlength=16
    ipa group-add-member senior-engineers --users="$username"
  fi
}

# Create users
create_user "jdoe" "John" "Doe" "jdoe@example.com" "engineering" "Senior Engineer" "manager@example.com"
create_user "asmith" "Alice" "Smith" "asmith@example.com" "operations" "Operations Lead" "director@example.com"
create_user "bwilson" "Bob" "Wilson" "bwilson@example.com" "devops" "DevOps Engineer" "jdoe@example.com"

# Set SSH keys in bulk
for user in jdoe asmith bwilson; do
  ipa user-mod "$user" \
    --sshpubkey="$(cat /path/to/${user}_rsa.pub)"
done

# Configure account expiration for contractors
ipa user-mod contractor1 \
  --principal-expiration=20251231235959Z

# Lock/unlock accounts
ipa user-disable contractor1  # Temporary disable
ipa user-enable contractor1   # Re-enable
```

### Advanced DNS Configuration

```bash
# Create DNS zones with DNSSEC
ipa dnszone-add example.com \
  --name-server=ns1.example.com. \
  --admin-email=admin.example.com. \
  --dynamic-update=TRUE \
  --allow-sync-ptr=TRUE

# Enable DNSSEC
ipa dnszone-mod example.com --dnssec=TRUE

# Create reverse zone
ipa dnszone-add 1.168.192.in-addr.arpa \
  --name-server=ns1.example.com.

# Add various DNS records
# A records
ipa dnsrecord-add example.com web1 --a-rec=192.168.1.10
ipa dnsrecord-add example.com web2 --a-rec=192.168.1.11
ipa dnsrecord-add example.com web3 --a-rec=192.168.1.12

# Load balancer with multiple A records
ipa dnsrecord-add example.com lb \
  --a-rec=192.168.1.20 \
  --a-rec=192.168.1.21

# AAAA records (IPv6)
ipa dnsrecord-add example.com web1 \
  --aaaa-rec=2001:db8::10

# CNAME records
ipa dnsrecord-add example.com www --cname-rec=lb.example.com.
ipa dnsrecord-add example.com blog --cname-rec=web1.example.com.
ipa dnsrecord-add example.com mail --cname-rec=smtp.example.com.

# MX records (mail servers)
ipa dnsrecord-add example.com @ \
  --mx-rec="10 mail1.example.com." \
  --mx-rec="20 mail2.example.com."

# TXT records (SPF, DKIM, DMARC)
ipa dnsrecord-add example.com @ \
  --txt-rec="v=spf1 mx ip4:192.168.1.0/24 -all"

ipa dnsrecord-add example.com _dmarc \
  --txt-rec="v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com"

ipa dnsrecord-add example.com default._domainkey \
  --txt-rec="v=DKIM1; k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GN..."

# SRV records (service discovery)
ipa dnsrecord-add example.com _ldap._tcp \
  --srv-rec="0 100 389 idm.example.com."

ipa dnsrecord-add example.com _kerberos._tcp \
  --srv-rec="0 100 88 idm.example.com."

ipa dnsrecord-add example.com _kerberos._udp \
  --srv-rec="0 100 88 idm.example.com."

# PTR records (reverse DNS)
ipa dnsrecord-add 1.168.192.in-addr.arpa 10 \
  --ptr-rec=web1.example.com.
ipa dnsrecord-add 1.168.192.in-addr.arpa 11 \
  --ptr-rec=web2.example.com.

# CAA records (certificate authority authorization)
ipa dnsrecord-add example.com @ \
  --caa-rec="0 issue \"letsencrypt.org\"" \
  --caa-rec="0 issuewild \"letsencrypt.org\"" \
  --caa-rec="0 iodef \"mailto:security@example.com\""

# NAPTR records (for SIP/VoIP)
ipa dnsrecord-add example.com @ \
  --naptr-rec="100 10 \"U\" \"E2U+sip\" \"!^.*$!sip:info@example.com!\" ."

# Add DNS forwarders
ipa dnsforwardzone-add cloudflare \
  --forwarder=1.1.1.1 \
  --forwarder=1.0.0.1 \
  --forward-policy=only

# Conditional forwarding
ipa dnsforwardzone-add partner.com \
  --forwarder=10.20.30.40 \
  --forward-policy=first
```

### Certificate Management and PKI

```bash
# Request host certificate with custom parameters
ipa-getcert request \
  -f /etc/pki/tls/certs/web1.crt \
  -k /etc/pki/tls/private/web1.key \
  -K host/web1.example.com \
  -D web1.example.com \
  -D www.example.com \
  -D api.example.com \
  -N "CN=web1.example.com,O=Example Inc,L=San Francisco,ST=CA,C=US" \
  -U id-kp-serverAuth \
  -U id-kp-clientAuth \
  -g 4096 \
  -v

# Request wildcard certificate
ipa-getcert request \
  -f /etc/pki/tls/certs/wildcard.crt \
  -k /etc/pki/tls/private/wildcard.key \
  -K host/idm.example.com \
  -D "*.example.com" \
  -D example.com \
  -N "CN=*.example.com" \
  -g 4096

# Service-specific certificates
# Apache/HTTPD
ipa service-add HTTP/web1.example.com
ipa-getcert request \
  -f /etc/pki/tls/certs/httpd.crt \
  -k /etc/pki/tls/private/httpd.key \
  -K HTTP/web1.example.com \
  -D web1.example.com \
  -C "systemctl reload httpd"

# PostgreSQL
ipa service-add postgres/db1.example.com
ipa-getcert request \
  -f /var/lib/pgsql/data/server.crt \
  -k /var/lib/pgsql/data/server.key \
  -K postgres/db1.example.com \
  -o postgres \
  -C "systemctl reload postgresql"

# LDAP
ipa service-add ldap/ldap1.example.com
ipa-getcert request \
  -f /etc/openldap/certs/ldap.crt \
  -k /etc/openldap/certs/ldap.key \
  -K ldap/ldap1.example.com

# Monitor certificates
ipa-getcert list

# Renew certificate manually
ipa-getcert resubmit -i <request-id>

# Auto-renewal with hooks
ipa-getcert request \
  -f /etc/pki/tls/certs/app.crt \
  -k /etc/pki/tls/private/app.key \
  -K HTTP/app.example.com \
  -C "systemctl reload nginx" \
  -B "/usr/local/bin/cert-backup.sh" \  # Pre-save command
  -a renew-grace-period=30              # Renew 30 days before expiry

# Export certificate and chain
ipa cert-show <serial> --certificate --chain > fullchain.pem

# Issue certificate from CSR
openssl req -new -newkey rsa:4096 \
  -keyout /tmp/app.key \
  -out /tmp/app.csr \
  -nodes \
  -subj "/CN=app.example.com/O=Example Inc/C=US"

ipa cert-request /tmp/app.csr \
  --principal host/app.example.com \
  --certificate-out /tmp/app.crt

# Revoke certificate
SERIAL=$(ipa-getcert list -i <request-id> | grep "serial:" | awk '{print $2}')
ipa cert-revoke $SERIAL --revocation-reason=4  # superseded

# Create sub-CA
ipa ca-add subca \
  --subject="CN=SubCA,O=Example Inc" \
  --desc="SubCA for internal services"
```

### Active Directory Trust Integration

```bash
# Establish AD trust
ipa trust-add ad.company.com \
  --type=ad \
  --admin Administrator \
  --password

# Verify trust
ipa trust-show ad.company.com

# Fetch AD domains
ipa trustdomain-find ad.company.com

# Enable AD user access to specific hosts
ipa idrange-add ad_range \
  --base-id=1000000 \
  --range-size=1000000 \
  --rid-base=0 \
  --dom-sid=S-1-5-21-...

# Create external group for AD users
ipa group-add --external ad_admins_external

# Add AD group to external group
ipa group-add-member ad_admins_external \
  --external='AD\Domain Admins'

# Create POSIX group
ipa group-add ad_admins_posix \
  --desc="AD Administrators"

# Link external and POSIX groups
ipa group-add-member ad_admins_posix \
  --groups=ad_admins_external

# Configure HBAC for AD users
ipa hbacrule-add ad_user_access
ipa hbacrule-add-user ad_user_access \
  --groups=ad_admins_posix
ipa hbacrule-add-host ad_user_access \
  --hostgroups=servers
ipa hbacrule-add-service ad_user_access \
  --hbacsvcs=sshd

# Test AD user authentication
echo "Password123" | kinit administrator@AD.COMPANY.COM
id administrator@ad.company.com
```

### Automember Rules

```bash
# Create automember rule for user groups
ipa automember-add --type=group engineers

# Add inclusive regex condition
ipa automember-add-condition --type=group engineers \
  --inclusive-regex='^.*@engineering\.example\.com$' \
  --key=mail

# Add based on user attribute
ipa automember-add-condition --type=group developers \
  --inclusive-regex='^developer$' \
  --key=title

# Hostgroup automember rules
ipa automember-add --type=hostgroup webservers

# Add condition for hostgroup
ipa automember-add-condition --type=hostgroup webservers \
  --inclusive-regex='^web[0-9]+\.example\.com$' \
  --key=fqdn

# Create database server automember
ipa automember-add --type=hostgroup dbservers
ipa automember-add-condition --type=hostgroup dbservers \
  --inclusive-regex='^db[0-9]+\.example\.com$' \
  --key=fqdn

# Apply automember rules to existing entries
ipa automember-rebuild --type=group
ipa automember-rebuild --type=hostgroup

# Test automember rules
ipa automember-find --type=group
```

### Backup and Recovery

```bash
# Full IPA backup
ipa-backup --data --logs --online

# Offline backup (recommended)
ipactl stop
ipa-backup --data --logs
ipactl start

# Backup to specific location
ipa-backup --data --logs --dir=/backup/ipa

# GPG encrypted backup
ipa-backup --data --logs --gpg --gpg-keyring=/root/.gnupg

# Incremental backup
ipa-backup --data --logs --online --instance=SECOND_BACKUP

# List backups
ipa-backup --list

# Restore from backup
ipactl stop
ipa-restore /var/lib/ipa/backup/ipa-full-2025-01-15-10-30-00
ipactl start

# Restore specific instance
ipa-restore --instance=ipa-full-2025-01-15 --data --online

# Database backup only
ipa-backup --data --online

# LDIF export (manual backup)
ldapsearch -x -H ldap://localhost \
  -D "cn=Directory Manager" \
  -W -b "dc=example,dc=com" > /backup/ldap-export.ldif

# Backup certificates
cp -r /var/lib/ipa/certs /backup/
cp /root/cacert.p12 /backup/

# Backup Kerberos database
kdb5_util dump /backup/krb5.dump
```

### Monitoring and Troubleshooting

```bash
# Check IPA service status
ipactl status

# Detailed status
systemctl status ipa
systemctl status dirsrv@EXAMPLE-COM.service
systemctl status krb5kdc.service
systemctl status kadmin.service
systemctl status httpd.service
systemctl status pki-tomcatd@pki-tomcat.service

# Check replication status
ipa-replica-manage list --verbose

# Check replication agreements
ldapsearch -x -D "cn=Directory Manager" -W \
  -b "cn=config" "(objectclass=nsds5replicationagreement)"

# Monitor LDAP operations
ldapsearch -x -H ldap://localhost -D "cn=Directory Manager" -W \
  -b "cn=monitor" "(objectClass=*)"

# Check DS logs
tail -f /var/log/dirsrv/slapd-EXAMPLE-COM/access
tail -f /var/log/dirsrv/slapd-EXAMPLE-COM/errors

# Kerberos logs
tail -f /var/log/krb5kdc.log
tail -f /var/log/kadmind.log

# Apache logs
tail -f /var/log/httpd/error_log
tail -f /var/log/httpd/access_log

# Check certificate tracking
ipa-getcert list

# Verify DNS
dig @localhost example.com ANY
dig @localhost _ldap._tcp.example.com SRV
dig @localhost _kerberos._tcp.example.com SRV

# Test Kerberos
kinit admin
klist
kvno host/idm.example.com

# LDAP connection test
ldapsearch -x -H ldap://idm.example.com \
  -b "dc=example,dc=com" "(uid=admin)"

# Test authentication
echo "password" | kinit testuser
ssh testuser@server.example.com

# Performance monitoring
# Install monitoring tools
dnf install -y sysstat iotop htop

# Monitor LDAP performance
ldapsearch -x -H ldap://localhost -D "cn=Directory Manager" -W \
  -b "cn=monitor,cn=ldbm database,cn=plugins,cn=config" \
  "(objectClass=*)" | grep -i cache

# Check database indexes
ipa-advise config-server-for-smart-card-auth

# Rebuild indexes
dsconf -D "cn=Directory Manager" ldap://localhost \
  backend index reindex userRoot

# Compact database
dsconf -D "cn=Directory Manager" ldap://localhost \
  backend compact userRoot
```

### Performance Tuning

```bash
# LDAP server tuning
# /etc/dirsrv/slapd-EXAMPLE-COM/dse.ldif

# Increase cache sizes
ldapmodify -x -D "cn=Directory Manager" -W << EOF
dn: cn=config,cn=ldbm database,cn=plugins,cn=config
changetype: modify
replace: nsslapd-cachememsize
nsslapd-cachememsize: 536870912
-
replace: nsslapd-dbcachesize
nsslapd-dbcachesize: 536870912
-
replace: nsslapd-dncachememsize
nsslapd-dncachememsize: 33554432
EOF

# Increase worker threads
ldapmodify -x -D "cn=Directory Manager" -W << EOF
dn: cn=config
changetype: modify
replace: nsslapd-threadnumber
nsslapd-threadnumber: 32
EOF

# Connection limits
ldapmodify -x -D "cn=Directory Manager" -W << EOF
dn: cn=config
changetype: modify
replace: nsslapd-maxdescriptors
nsslapd-maxdescriptors: 8192
-
replace: nsslapd-maxthreadsperconn
nsslapd-maxthreadsperconn: 10
EOF

# Kerberos tuning
# /var/kerberos/krb5kdc/kdc.conf
cat >> /var/kerberos/krb5kdc/kdc.conf << EOF
[kdcdefaults]
 kdc_tcp_ports = 88
 kdc_max_tcp_connections = 2048

[realms]
 EXAMPLE.COM = {
  max_life = 24h 0m 0s
  max_renewable_life = 7d 0h 0m 0s
  default_principal_flags = +preauth
 }
EOF

# Apache tuning for IPA
# /etc/httpd/conf.d/ipa.conf
cat >> /etc/httpd/conf.modules.d/00-mpm.conf << EOF
<IfModule mpm_prefork_module>
    StartServers         10
    MinSpareServers      10
    MaxSpareServers      20
    MaxRequestWorkers    256
    MaxConnectionsPerChild  4000
</IfModule>
EOF

systemctl reload httpd
```

### Security Hardening

```bash
# Disable anonymous LDAP binds
ldapmodify -x -D "cn=Directory Manager" -W << EOF
dn: cn=config
changetype: modify
replace: nsslapd-allow-anonymous-access
nsslapd-allow-anonymous-access: rootdse
EOF

# Enable password quality checking
ipa pwpolicy-mod --minlength=14 \
  --minclasses=4 \
  --maxfail=5 \
  --failinterval=1800 \
  --lockouttime=3600

# Require Kerberos pre-authentication
kadmin.local -q "modprinc +requires_preauth admin"

# Disable weak encryption types
ipa-advise config-server-for-smart-card-auth

# Edit /etc/krb5.conf
cat >> /etc/krb5.conf << EOF
[libdefaults]
 permitted_enctypes = aes256-cts-hmac-sha1-96 aes128-cts-hmac-sha1-96
 default_tgs_enctypes = aes256-cts-hmac-sha1-96 aes128-cts-hmac-sha1-96
 default_tkt_enctypes = aes256-cts-hmac-sha1-96 aes128-cts-hmac-sha1-96
EOF

# Configure SELinux
setsebool -P httpd_can_network_connect on
setsebool -P allow_httpd_mod_auth_pam on

# Firewall configuration
firewall-cmd --permanent --add-service=freeipa-ldap
firewall-cmd --permanent --add-service=freeipa-ldaps
firewall-cmd --permanent --add-service=dns
firewall-cmd --permanent --add-service=ntp
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-service=kerberos
firewall-cmd --permanent --add-service=kpasswd
firewall-cmd --reload

# Audit logging
ipa-advise enable-admins-sudo
auditctl -w /etc/ipa/ -p wa -k ipa_config_change
auditctl -w /var/log/httpd/ -p wa -k ipa_httpd_logs
```

## Additional Resources

- [Red Hat IDM Documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/planning_identity_management/)
- [FreeIPA (Upstream)](https://www.freeipa.org/page/Documentation)
- [RHEL Identity Management Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_identity_management/)

---

*Last updated: 2025-11-16*
