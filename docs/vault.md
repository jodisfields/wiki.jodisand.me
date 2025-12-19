# HashiCorp Vault

A comprehensive guide to HashiCorp Vault - a tool for managing secrets and protecting sensitive data.

## Table of Contents

- [Introduction](#introduction)
- [Installation](#installation)
- [Server Configuration](#server-configuration)
- [Authentication Methods](#authentication-methods)
- [Secrets Engines](#secrets-engines)
- [Policies](#policies)
- [Dynamic Secrets](#dynamic-secrets)
- [Encryption as a Service](#encryption-as-a-service)
- [High Availability](#high-availability)
- [Best Practices](#best-practices)

## Introduction

### What is Vault?

HashiCorp Vault is a secrets management tool that provides:
- **Secret Storage**: Encrypted storage for credentials, API keys, passwords
- **Dynamic Secrets**: Generate secrets on-demand
- **Data Encryption**: Encryption as a service
- **Leasing & Renewal**: Time-limited access with automatic renewal
- **Revocation**: Instant secret revocation
- **Audit Logging**: Detailed audit trails

### Key Concepts

- **Secrets**: Sensitive data (passwords, API keys, certificates)
- **Paths**: Location where secrets are stored
- **Policies**: Define access permissions
- **Tokens**: Authentication credentials
- **Leases**: TTL for secrets
- **Seal/Unseal**: Vault encryption state

## Installation

### Binary Installation

```bash
# Download Vault
wget https://releases.hashicorp.com/vault/1.15.0/vault_1.15.0_linux_amd64.zip

# Install
unzip vault_1.15.0_linux_amd64.zip
sudo mv vault /usr/local/bin/

# Verify
vault version

# Enable autocomplete
vault -autocomplete-install
```

### Docker

```bash
# Run Vault in dev mode
docker run --cap-add=IPC_LOCK \
  -e 'VAULT_DEV_ROOT_TOKEN_ID=myroot' \
  -p 8200:8200 \
  --name vault \
  vault:latest

# Production mode with config
docker run --cap-add=IPC_LOCK \
  -v $(pwd)/vault/config:/vault/config \
  -v $(pwd)/vault/data:/vault/data \
  -p 8200:8200 \
  --name vault \
  vault:latest server
```

### Kubernetes (Helm)

```bash
# Add Helm repo
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update

# Install Vault
helm install vault hashicorp/vault \
  --set "server.ha.enabled=true" \
  --set "server.ha.replicas=3"
```

## Server Configuration

### Dev Mode (Testing Only)

```bash
# Start dev server
vault server -dev

# Dev server details:
# - Root token: provided in output
# - No TLS
# - In-memory storage
# - Auto-unsealed
```

### Production Configuration

```hcl
# config.hcl
storage "raft" {
  path    = "/vault/data"
  node_id = "node1"
}

listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_disable = 0
  tls_cert_file = "/vault/tls/cert.pem"
  tls_key_file  = "/vault/tls/key.pem"
}

api_addr = "https://vault.example.com:8200"
cluster_addr = "https://192.168.1.10:8201"

ui = true

seal "awskms" {
  region     = "us-east-1"
  kms_key_id = "alias/vault-seal"
}
```

### Start Vault Server

```bash
# Start Vault
vault server -config=config.hcl

# As systemd service
sudo systemctl enable vault
sudo systemctl start vault
```

### Initialize Vault

```bash
# Set Vault address
export VAULT_ADDR='https://vault.example.com:8200'

# Initialize (generates unseal keys and root token)
vault operator init

# Save unseal keys and root token securely!

# Output example:
# Unseal Key 1: ...
# Unseal Key 2: ...
# Unseal Key 3: ...
# Unseal Key 4: ...
# Unseal Key 5: ...
# Initial Root Token: s.xxxxx
```

### Unseal Vault

```bash
# Unseal with keys (requires threshold, typically 3 of 5)
vault operator unseal <key1>
vault operator unseal <key2>
vault operator unseal <key3>

# Check seal status
vault status

# Auto-unseal with AWS KMS, Azure Key Vault, or GCP KMS
# (configured in seal stanza)
```

### Login

```bash
# Login with root token
vault login <root-token>

# Login with other auth method
vault login -method=userpass username=admin
```

## Authentication Methods

### Enable Auth Methods

```bash
# List enabled auth methods
vault auth list

# Enable userpass
vault auth enable userpass

# Enable AppRole
vault auth enable approle

# Enable Kubernetes
vault auth enable kubernetes

# Enable LDAP
vault auth enable ldap

# Enable GitHub
vault auth enable github
```

### Userpass Authentication

```bash
# Create user
vault write auth/userpass/users/admin \
  password=securepass \
  policies=admin

# Login
vault login -method=userpass username=admin

# Change password
vault write auth/userpass/users/admin/password \
  password=newpassword
```

### AppRole Authentication

```bash
# Enable AppRole
vault auth enable approle

# Create role
vault write auth/approle/role/my-app \
  token_policies="my-app-policy" \
  token_ttl=1h \
  token_max_ttl=4h

# Get Role ID
vault read auth/approle/role/my-app/role-id

# Generate Secret ID
vault write -f auth/approle/role/my-app/secret-id

# Login with AppRole
vault write auth/approle/login \
  role_id="<role-id>" \
  secret_id="<secret-id>"
```

### Kubernetes Authentication

```bash
# Configure Kubernetes auth
vault write auth/kubernetes/config \
  kubernetes_host="https://kubernetes.default.svc:443" \
  kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
  token_reviewer_jwt=@/var/run/secrets/kubernetes.io/serviceaccount/token

# Create role
vault write auth/kubernetes/role/myapp \
  bound_service_account_names=myapp \
  bound_service_account_namespaces=default \
  policies=myapp-policy \
  ttl=1h
```

## Secrets Engines

### Enable Secrets Engines

```bash
# List enabled engines
vault secrets list

# Enable KV v2
vault secrets enable -path=secret kv-v2

# Enable database
vault secrets enable database

# Enable PKI
vault secrets enable pki

# Enable AWS
vault secrets enable aws

# Disable engine
vault secrets disable secret/
```

### KV Secrets Engine

```bash
# Write secret (KV v2)
vault kv put secret/myapp/config \
  username=admin \
  password=secretpass \
  api_key=abc123

# Read secret
vault kv get secret/myapp/config

# Get specific field
vault kv get -field=password secret/myapp/config

# List secrets
vault kv list secret/myapp/

# Delete secret (soft delete)
vault kv delete secret/myapp/config

# Undelete
vault kv undelete -versions=2 secret/myapp/config

# Permanently delete
vault kv destroy -versions=2 secret/myapp/config

# Read specific version
vault kv get -version=1 secret/myapp/config

# View metadata
vault kv metadata get secret/myapp/config
```

### Database Secrets Engine

```bash
# Configure PostgreSQL
vault write database/config/postgresql \
  plugin_name=postgresql-database-plugin \
  allowed_roles="readonly" \
  connection_url="postgresql://{{username}}:{{password}}@postgres:5432/mydb?sslmode=disable" \
  username="vault" \
  password="vaultpass"

# Create role
vault write database/roles/readonly \
  db_name=postgresql \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; \
    GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
  default_ttl="1h" \
  max_ttl="24h"

# Generate credentials
vault read database/creds/readonly

# Rotate root credentials
vault write -f database/rotate-root/postgresql
```

### AWS Secrets Engine

```bash
# Configure AWS
vault write aws/config/root \
  access_key=AKIAIOSFODNN7EXAMPLE \
  secret_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY \
  region=us-east-1

# Create role
vault write aws/roles/my-role \
  credential_type=iam_user \
  policy_document=-<<EOF
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "s3:*",
    "Resource": "*"
  }]
}
EOF

# Generate AWS credentials
vault read aws/creds/my-role
```

### PKI Secrets Engine

```bash
# Enable PKI
vault secrets enable pki

# Configure max lease TTL
vault secrets tune -max-lease-ttl=87600h pki

# Generate root CA
vault write -field=certificate pki/root/generate/internal \
  common_name="example.com" \
  ttl=87600h > CA_cert.crt

# Configure CA and CRL URLs
vault write pki/config/urls \
  issuing_certificates="http://vault.example.com:8200/v1/pki/ca" \
  crl_distribution_points="http://vault.example.com:8200/v1/pki/crl"

# Create role
vault write pki/roles/example-dot-com \
  allowed_domains="example.com" \
  allow_subdomains=true \
  max_ttl="720h"

# Issue certificate
vault write pki/issue/example-dot-com \
  common_name="test.example.com" \
  ttl="24h"
```

## Policies

### Create Policy

```hcl
# policy.hcl
# Allow reading secrets
path "secret/data/myapp/*" {
  capabilities = ["read", "list"]
}

# Allow writing to specific path
path "secret/data/myapp/config" {
  capabilities = ["create", "update"]
}

# Deny deletion
path "secret/data/myapp/*" {
  capabilities = ["delete"]
  deny = true
}

# Admin access
path "*" {
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}

# Database creds
path "database/creds/readonly" {
  capabilities = ["read"]
}
```

### Manage Policies

```bash
# Write policy
vault policy write myapp-policy policy.hcl

# List policies
vault policy list

# Read policy
vault policy read myapp-policy

# Delete policy
vault policy delete myapp-policy

# Assign policy to token
vault token create -policy=myapp-policy
```

## Dynamic Secrets

### Database Example

```bash
# Read credentials (automatically generated)
vault read database/creds/readonly

# Output:
# Key                Value
# ---                -----
# lease_id           database/creds/readonly/abc123
# lease_duration     1h
# username           v-root-readonly-xyz
# password           A1a-generated-password

# Credentials are automatically cleaned up after lease expires
```

### AWS Example

```bash
# Generate AWS credentials
vault read aws/creds/my-role

# Credentials expire based on role TTL
# IAM user is automatically deleted after expiration
```

## Encryption as a Service

### Transit Secrets Engine

```bash
# Enable transit
vault secrets enable transit

# Create encryption key
vault write -f transit/keys/my-key

# Encrypt data
vault write transit/encrypt/my-key \
  plaintext=$(echo "my secret data" | base64)

# Decrypt data
vault write transit/decrypt/my-key \
  ciphertext=vault:v1:xyz...

# Rotate encryption key
vault write -f transit/keys/my-key/rotate

# Re-encrypt with new key
vault write transit/rewrap/my-key \
  ciphertext=vault:v1:xyz...
```

## High Availability

### Raft Storage Backend

```hcl
# Node 1 config
storage "raft" {
  path = "/vault/data"
  node_id = "node1"

  retry_join {
    leader_api_addr = "https://node2:8200"
  }

  retry_join {
    leader_api_addr = "https://node3:8200"
  }
}
```

### Join Raft Cluster

```bash
# On additional nodes
vault operator raft join https://node1:8200
```

### Raft Operations

```bash
# List peers
vault operator raft list-peers

# Remove peer
vault operator raft remove-peer node2

# Take snapshot
vault operator raft snapshot save backup.snap

# Restore snapshot
vault operator raft snapshot restore backup.snap
```

## Best Practices

### Security

1. **Never use dev mode in production**
2. **Enable TLS** for all connections
3. **Use auto-unseal** (AWS KMS, Azure Key Vault, etc.)
4. **Implement least privilege** policies
5. **Enable audit logging**
6. **Rotate tokens regularly**
7. **Use short TTLs** for secrets
8. **Protect unseal keys** (store separately, use Shamir's Secret Sharing)

### Operational

1. **Use HA deployment** (minimum 3 nodes)
2. **Regular backups** of Vault data
3. **Monitor metrics** (Prometheus, CloudWatch)
4. **Implement disaster recovery** procedures
5. **Use namespaces** for multi-tenancy
6. **Version control** policies
7. **Test recovery procedures**
8. **Automate secret rotation**

### Audit Logging

```bash
# Enable audit logging
vault audit enable file file_path=/var/log/vault/audit.log

# Enable syslog audit
vault audit enable syslog

# List audit devices
vault audit list

# Disable audit device
vault audit disable file/
```

## Additional Resources

- [Vault Documentation](https://www.vaultproject.io/docs)
- [Vault Tutorials](https://learn.hashicorp.com/vault)
- [Vault GitHub](https://github.com/hashicorp/vault)
- [Vault API Documentation](https://www.vaultproject.io/api-docs)

---

*Last updated: 2025-11-16*
