# TLS/SSL Certificates & PKI

A comprehensive guide to TLS/SSL certificates, Certificate Authorities, and Public Key Infrastructure.

## Table of Contents

- [Introduction](#introduction)
- [Certificate Basics](#certificate-basics)
- [OpenSSL Commands](#openssl-commands)
- [Certificate Authorities](#certificate-authorities)
- [Certificate Chains](#certificate-chains)
- [System Trust Stores](#system-trust-stores)
- [Common Formats](#common-formats)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## Introduction

### What is TLS/SSL?

- **TLS (Transport Layer Security)**: Cryptographic protocol for secure communication
- **SSL (Secure Sockets Layer)**: Predecessor to TLS (deprecated)
- **HTTPS**: HTTP over TLS/SSL

### PKI Components

```
Certificate Authority (CA)
    ↓
Root Certificate
    ↓
Intermediate Certificate(s)
    ↓
End-Entity/Leaf Certificate
```

### Certificate Types

- **DV (Domain Validation)**: Validates domain ownership
- **OV (Organization Validation)**: Validates organization identity
- **EV (Extended Validation)**: Highest level of validation
- **Wildcard**: Covers *.domain.com
- **Multi-Domain (SAN)**: Multiple domains in one cert
- **Self-Signed**: Not trusted by browsers (testing only)

## Certificate Basics

### X.509 Certificate Structure

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 1234567890
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=US, O=Example CA, CN=Example Root CA
        Validity
            Not Before: Jan  1 00:00:00 2024 GMT
            Not After : Jan  1 00:00:00 2025 GMT
        Subject: C=US, O=Example Inc, CN=www.example.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
        X509v3 extensions:
            X509v3 Subject Alternative Name:
                DNS:www.example.com, DNS:example.com
            X509v3 Key Usage:
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage:
                TLS Web Server Authentication
    Signature Algorithm: sha256WithRSAEncryption
```

### Key Concepts

```yaml
Private Key:
  - Secret key
  - Never shared
  - Used for signing/decryption

Public Key:
  - Publicly shared
  - In certificate
  - Used for verification/encryption

CSR (Certificate Signing Request):
  - Request for CA to sign
  - Contains public key
  - Contains subject info

Subject:
  - Entity the cert represents
  - Domain name, organization

Issuer:
  - CA that signed the cert

Validity Period:
  - Not Before/Not After dates

Subject Alternative Name (SAN):
  - Additional domains/IPs
```

## OpenSSL Commands

### Generate Private Key

```bash
# RSA Key (2048-bit)
openssl genrsa -out private.key 2048

# RSA Key (4096-bit)
openssl genrsa -out private.key 4096

# RSA Key with password protection
openssl genrsa -aes256 -out private.key 2048

# ECDSA Key (Elliptic Curve)
openssl ecparam -genkey -name prime256v1 -out private.key

# Remove password from key
openssl rsa -in encrypted.key -out decrypted.key
```

### Generate CSR (Certificate Signing Request)

```bash
# Generate CSR from private key
openssl req -new -key private.key -out request.csr

# Generate key and CSR together
openssl req -new -newkey rsa:2048 -nodes -keyout private.key -out request.csr

# CSR with SANs (create openssl.cnf)
openssl req -new -key private.key -out request.csr -config openssl.cnf

# View CSR
openssl req -text -noout -in request.csr
```

### OpenSSL Configuration for SAN

```ini
# openssl.cnf
[req]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
req_extensions = req_ext

[dn]
C = US
ST = California
L = San Francisco
O = Example Inc
CN = www.example.com

[req_ext]
subjectAltName = @alt_names

[alt_names]
DNS.1 = www.example.com
DNS.2 = example.com
DNS.3 = api.example.com
IP.1 = 192.168.1.100
```

### Self-Signed Certificates

```bash
# Generate self-signed certificate
openssl req -x509 -newkey rsa:2048 -nodes \
  -keyout private.key \
  -out certificate.crt \
  -days 365 \
  -subj "/C=US/ST=CA/L=SF/O=Example/CN=www.example.com"

# Self-signed with SAN
openssl req -x509 -newkey rsa:2048 -nodes \
  -keyout private.key \
  -out certificate.crt \
  -days 365 \
  -config openssl.cnf \
  -extensions req_ext
```

### View Certificates

```bash
# View certificate details
openssl x509 -in certificate.crt -text -noout

# View certificate dates
openssl x509 -in certificate.crt -noout -dates

# View certificate subject
openssl x509 -in certificate.crt -noout -subject

# View certificate issuer
openssl x509 -in certificate.crt -noout -issuer

# View SANs
openssl x509 -in certificate.crt -noout -ext subjectAltName

# Verify certificate
openssl x509 -in certificate.crt -noout -checkend 86400  # Check if expires in 24h
```

### Convert Certificates

```bash
# PEM to DER
openssl x509 -in cert.pem -outform DER -out cert.der

# DER to PEM
openssl x509 -in cert.der -inform DER -out cert.pem

# PEM to PKCS12 (PFX)
openssl pkcs12 -export -out cert.pfx \
  -inkey private.key \
  -in certificate.crt \
  -certfile ca-chain.crt

# PKCS12 to PEM
openssl pkcs12 -in cert.pfx -out cert.pem -nodes

# Extract private key from PKCS12
openssl pkcs12 -in cert.pfx -nocerts -out private.key -nodes

# Extract certificate from PKCS12
openssl pkcs12 -in cert.pfx -nokeys -out certificate.crt
```

### Test SSL/TLS Connections

```bash
# Test HTTPS connection
openssl s_client -connect example.com:443 -servername example.com

# Show certificate chain
openssl s_client -connect example.com:443 -showcerts

# Test specific TLS version
openssl s_client -connect example.com:443 -tls1_2
openssl s_client -connect example.com:443 -tls1_3

# Test cipher suite
openssl s_client -connect example.com:443 -cipher 'ECDHE-RSA-AES256-GCM-SHA384'

# Verify certificate
openssl s_client -connect example.com:443 -CAfile ca-bundle.crt
```

## Certificate Authorities

### Create Root CA

```bash
# Generate Root CA private key
openssl genrsa -aes256 -out ca-key.pem 4096

# Generate Root CA certificate
openssl req -new -x509 -days 3650 -key ca-key.pem -out ca-cert.pem \
  -subj "/C=US/ST=CA/O=Example CA/CN=Example Root CA"

# View Root CA certificate
openssl x509 -in ca-cert.pem -text -noout
```

### Create Intermediate CA

```bash
# Generate Intermediate CA key
openssl genrsa -aes256 -out intermediate-key.pem 4096

# Create Intermediate CA CSR
openssl req -new -key intermediate-key.pem -out intermediate.csr \
  -subj "/C=US/ST=CA/O=Example CA/CN=Example Intermediate CA"

# Sign Intermediate CA with Root CA
openssl x509 -req -days 1825 -in intermediate.csr \
  -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial \
  -out intermediate-cert.pem \
  -extfile <(echo "basicConstraints=CA:TRUE,pathlen:0")
```

### Sign Certificate with CA

```bash
# Sign CSR with CA
openssl x509 -req -days 365 -in request.csr \
  -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial \
  -out certificate.crt

# Sign with extensions
openssl x509 -req -days 365 -in request.csr \
  -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial \
  -out certificate.crt \
  -extfile <(printf "subjectAltName=DNS:example.com,DNS:www.example.com\nkeyUsage=digitalSignature,keyEncipherment\nextendedKeyUsage=serverAuth")
```

## Certificate Chains

### Building Certificate Chain

```bash
# Certificate chain order:
# 1. Server/End-entity certificate
# 2. Intermediate CA certificate(s)
# 3. Root CA certificate (optional)

# Create chain file
cat certificate.crt intermediate.crt > chain.crt

# Or include root
cat certificate.crt intermediate.crt root.crt > fullchain.crt
```

### Verify Certificate Chain

```bash
# Verify against CA bundle
openssl verify -CAfile ca-bundle.crt certificate.crt

# Verify complete chain
openssl verify -CAfile root.crt -untrusted intermediate.crt certificate.crt

# Show chain
openssl s_client -connect example.com:443 -showcerts | \
  openssl x509 -text -noout
```

### Extract Certificates from Chain

```bash
# Split chain into separate files
csplit -f cert- fullchain.pem '/-----BEGIN CERTIFICATE-----/' '{*}'

# Or using awk
awk 'BEGIN {c=0} /-----BEGIN CERT/ {c++} {print > "cert" c ".pem"}' fullchain.pem
```

## System Trust Stores

### Linux (Debian/Ubuntu)

```bash
# Trust store location
/etc/ssl/certs/ca-certificates.crt
/usr/share/ca-certificates/

# Add custom CA
sudo cp my-ca.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates

# Remove CA
sudo rm /usr/local/share/ca-certificates/my-ca.crt
sudo update-ca-certificates --fresh

# List trusted CAs
awk -v cmd='openssl x509 -noout -subject' '
    /BEGIN/{close(cmd)};{print | cmd}' < /etc/ssl/certs/ca-certificates.crt
```

### Linux (RHEL/CentOS)

```bash
# Trust store
/etc/pki/tls/certs/ca-bundle.crt
/etc/pki/ca-trust/source/anchors/

# Add custom CA
sudo cp my-ca.crt /etc/pki/ca-trust/source/anchors/
sudo update-ca-trust

# Extract all CAs
sudo update-ca-trust extract
```

### macOS

```bash
# Add to system keychain
sudo security add-trusted-cert -d -r trustRoot \
  -k /Library/Keychains/System.keychain my-ca.crt

# Add to login keychain
security add-trusted-cert -d -r trustRoot \
  -k ~/Library/Keychains/login.keychain-db my-ca.crt

# Remove from keychain
sudo security delete-certificate -c "CA Name" /Library/Keychains/System.keychain

# List certificates
security find-certificate -a /Library/Keychains/System.keychain
```

### Windows

```powershell
# Import certificate (PowerShell)
Import-Certificate -FilePath C:\path\to\ca.crt `
  -CertStoreLocation Cert:\LocalMachine\Root

# Remove certificate
Get-ChildItem Cert:\LocalMachine\Root | Where-Object {$_.Subject -like "*CA Name*"} | Remove-Item

# Using certmgr
certmgr.msc  # GUI certificate manager
```

### Application-Specific

```bash
# Java keystore
keytool -import -trustcacerts -alias myca \
  -file ca-cert.pem \
  -keystore $JAVA_HOME/lib/security/cacerts \
  -storepass changeit

# Python certifi
# Add to: $(python -m certifi)/cacert.pem

# Node.js
export NODE_EXTRA_CA_CERTS=/path/to/ca-cert.pem

# cURL
curl --cacert ca-cert.pem https://example.com

# wget
wget --ca-certificate=ca-cert.pem https://example.com
```

## Common Formats

### Certificate Formats

```bash
# PEM (Privacy Enhanced Mail)
- Base64 encoded
- Extension: .pem, .crt, .cer, .key
- BEGIN/END markers
- Most common format

# DER (Distinguished Encoding Rules)
- Binary format
- Extension: .der, .cer
- Used by Java

# PKCS#7/P7B
- Base64 or binary
- Extension: .p7b, .p7c
- Contains certificates only (no private key)
- Used by Windows, Java

# PKCS#12/PFX
- Binary format
- Extension: .pfx, .p12
- Contains certificate + private key
- Password protected
- Used by Windows
```

### Convert Between Formats

```bash
# PEM to DER
openssl x509 -in cert.pem -outform DER -out cert.der

# DER to PEM
openssl x509 -in cert.der -inform DER -out cert.pem

# PEM to PKCS7
openssl crl2pkcs7 -nocrl -certfile cert.pem -out cert.p7b

# PKCS7 to PEM
openssl pkcs7 -print_certs -in cert.p7b -out cert.pem

# PEM to PKCS12
openssl pkcs12 -export -out cert.pfx \
  -inkey private.key -in cert.pem -certfile ca-chain.pem

# PKCS12 to PEM
openssl pkcs12 -in cert.pfx -out cert.pem -nodes
```

## Best Practices

### Key Management

```yaml
Private Keys:
  - Never share private keys
  - Store securely (encrypted)
  - Use strong passwords
  - Rotate regularly
  - Backup safely

Key Sizes:
  - RSA: Minimum 2048-bit (4096-bit recommended)
  - ECDSA: 256-bit or 384-bit
  - Avoid RSA 1024-bit (insecure)
```

### Certificate Lifecycle

```yaml
Generation:
  - Use strong keys (RSA 2048+ or ECDSA 256+)
  - Include all necessary SANs
  - Set appropriate validity period
  - Use reputable CA

Deployment:
  - Protect private keys
  - Use complete chain
  - Configure proper cipher suites
  - Enable HSTS, OCSP stapling

Monitoring:
  - Track expiration dates
  - Set renewal reminders (30-60 days before)
  - Monitor for revocations
  - Test regularly

Renewal:
  - Automate when possible (cert-manager, ACME)
  - Generate new keys (don't reuse)
  - Update all systems
  - Verify after deployment

Revocation:
  - Revoke compromised certificates immediately
  - Update CRL/OCSP
  - Deploy new certificates
```

### Security

```yaml
TLS Configuration:
  - Disable SSLv2, SSLv3, TLS 1.0, TLS 1.1
  - Enable TLS 1.2 and TLS 1.3
  - Use strong cipher suites
  - Enable Perfect Forward Secrecy
  - Implement HSTS

Recommended Ciphers (Mozilla Modern):
  - TLS 1.3: TLS_AES_128_GCM_SHA256, TLS_AES_256_GCM_SHA384, TLS_CHACHA20_POLY1305_SHA256
  - TLS 1.2: ECDHE-ECDSA-AES128-GCM-SHA256, ECDHE-RSA-AES128-GCM-SHA256

Headers:
  - Strict-Transport-Security
  - X-Frame-Options
  - X-Content-Type-Options
```

## Troubleshooting

### Common Issues

```bash
# Certificate not trusted
# - Incomplete chain
# - CA not in trust store
# - Expired certificate
openssl verify -CAfile ca-bundle.crt certificate.crt
openssl s_client -connect example.com:443 -CAfile ca-bundle.crt

# Name mismatch
# - CN/SAN doesn't match domain
openssl x509 -in cert.crt -noout -text | grep -A1 "Subject Alternative Name"

# Expired certificate
openssl x509 -in cert.crt -noout -dates
openssl s_client -connect example.com:443 | openssl x509 -noout -dates

# Wrong certificate order in chain
# - Should be: leaf -> intermediate -> root
cat cert.crt intermediate.crt > chain.crt

# Private key mismatch
# - Certificate and key don't match
openssl x509 -noout -modulus -in cert.crt | openssl md5
openssl rsa -noout -modulus -in key.pem | openssl md5
# Modulus should match

# Testing tools
# SSL Labs: https://www.ssllabs.com/ssltest/
curl -I https://example.com
testssl.sh https://example.com
```

## Additional Resources

- [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/)
- [Let's Encrypt](https://letsencrypt.org/)
- [OpenSSL Documentation](https://www.openssl.org/docs/)
- [SSL Labs](https://www.ssllabs.com/)
- [Certificate Transparency](https://certificate.transparency.dev/)

---

*Last updated: 2025-11-16*
