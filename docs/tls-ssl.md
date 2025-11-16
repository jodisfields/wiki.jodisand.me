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

## Advanced OpenSSL Examples

### Advanced CSR Generation

```bash
# CSR with all fields and multiple SANs
openssl req -new -key private.key -out request.csr \
  -subj "/C=US/ST=California/L=San Francisco/O=Example Inc/OU=IT Department/CN=www.example.com/emailAddress=admin@example.com" \
  -addext "subjectAltName=DNS:www.example.com,DNS:example.com,DNS:api.example.com,DNS:*.example.com,IP:192.168.1.100,IP:10.0.0.1"

# CSR with extended key usage
openssl req -new -key private.key -out request.csr \
  -config <(cat <<EOF
[req]
default_bits = 4096
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[dn]
C=US
ST=California
L=San Francisco
O=Example Inc
OU=Engineering
CN=www.example.com

[req_ext]
subjectAltName = @alt_names
keyUsage = digitalSignature, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth, clientAuth
basicConstraints = CA:FALSE

[alt_names]
DNS.1 = www.example.com
DNS.2 = example.com
DNS.3 = *.example.com
DNS.4 = api.example.com
DNS.5 = cdn.example.com
IP.1 = 192.168.1.100
IP.2 = 10.0.0.1
email.1 = admin@example.com
EOF
)

# CSR for code signing
openssl req -new -key private.key -out codesign.csr \
  -subj "/C=US/ST=CA/O=Example Inc/CN=Example Code Signing" \
  -addext "extendedKeyUsage=codeSigning"

# CSR for email/S/MIME
openssl req -new -key private.key -out email.csr \
  -subj "/C=US/ST=CA/O=Example Inc/CN=user@example.com/emailAddress=user@example.com" \
  -addext "extendedKeyUsage=emailProtection" \
  -addext "subjectAltName=email:user@example.com,email:alternate@example.com"

# Verify CSR
openssl req -in request.csr -noout -text
openssl req -in request.csr -noout -verify
openssl req -in request.csr -noout -subject
openssl req -in request.csr -noout -pubkey
```

### Advanced Certificate Signing

```bash
# Sign certificate with custom v3 extensions
openssl x509 -req -days 365 -in request.csr \
  -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial \
  -out certificate.crt \
  -extfile <(cat <<EOF
basicConstraints = CA:FALSE
keyUsage = critical, digitalSignature, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = DNS:www.example.com, DNS:example.com, DNS:*.example.com
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
certificatePolicies = 1.2.3.4
crlDistributionPoints = URI:http://crl.example.com/ca.crl
authorityInfoAccess = OCSP;URI:http://ocsp.example.com, caIssuers;URI:http://ca.example.com/ca.crt
EOF
)

# Sign with SHA-256
openssl x509 -req -sha256 -days 365 -in request.csr \
  -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial \
  -out certificate.crt

# Sign with SHA-512
openssl x509 -req -sha512 -days 730 -in request.csr \
  -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial \
  -out certificate.crt

# Create multi-year certificate
openssl x509 -req -days 1095 -in request.csr \
  -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial \
  -out certificate.crt \
  -extfile extensions.cnf

# Sign intermediate CA certificate
openssl x509 -req -days 1825 -in intermediate.csr \
  -CA root-ca.pem -CAkey root-key.pem -CAcreateserial \
  -out intermediate-ca.pem \
  -extfile <(cat <<EOF
basicConstraints = critical, CA:TRUE, pathlen:0
keyUsage = critical, keyCertSign, cRLSign
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
EOF
)
```

### Certificate Validation and Verification

```bash
# Verify certificate against CA bundle
openssl verify -CAfile ca-bundle.crt certificate.crt
openssl verify -CAfile root-ca.pem -untrusted intermediate-ca.pem server-cert.pem

# Verify certificate chain with CRL
openssl verify -CAfile root-ca.pem -CRLfile ca.crl -crl_check certificate.crt

# Verify with CRL check for entire chain
openssl verify -CAfile root-ca.pem -untrusted intermediate-ca.pem \
  -CRLfile ca.crl -crl_check_all server-cert.pem

# Check certificate purpose
openssl verify -purpose sslserver -CAfile ca-bundle.crt server-cert.pem
openssl verify -purpose sslclient -CAfile ca-bundle.crt client-cert.pem
openssl verify -purpose smimesign -CAfile ca-bundle.crt email-cert.pem
openssl verify -purpose smimeencrypt -CAfile ca-bundle.crt email-cert.pem

# Verify certificate signature
openssl x509 -in certificate.crt -noout -text | grep "Signature Algorithm"

# Verify private key matches certificate
CERT_MODULUS=$(openssl x509 -noout -modulus -in certificate.crt | openssl md5)
KEY_MODULUS=$(openssl rsa -noout -modulus -in private.key | openssl md5)
echo "Certificate: $CERT_MODULUS"
echo "Private Key: $KEY_MODULUS"
[ "$CERT_MODULUS" = "$KEY_MODULUS" ] && echo "Match!" || echo "Mismatch!"

# Verify CSR matches private key
CSR_MODULUS=$(openssl req -noout -modulus -in request.csr | openssl md5)
KEY_MODULUS=$(openssl rsa -noout -modulus -in private.key | openssl md5)
[ "$CSR_MODULUS" = "$KEY_MODULUS" ] && echo "Match!" || echo "Mismatch!"
```

### OCSP Operations

```bash
# OCSP request
openssl ocsp -issuer ca-cert.pem -cert certificate.crt \
  -url http://ocsp.example.com -resp_text

# OCSP with nonce (prevents replay attacks)
openssl ocsp -issuer ca-cert.pem -cert certificate.crt \
  -url http://ocsp.example.com -nonce -resp_text

# Verify OCSP response
openssl ocsp -issuer ca-cert.pem -cert certificate.crt \
  -url http://ocsp.example.com -CAfile ca-bundle.crt \
  -resp_text -verify_other ca-cert.pem

# Save OCSP response
openssl ocsp -issuer ca-cert.pem -cert certificate.crt \
  -url http://ocsp.example.com -respout ocsp-response.der

# Read OCSP response
openssl ocsp -respin ocsp-response.der -text

# Create OCSP responder (requires OCSP signing cert)
openssl ocsp -port 8080 -index index.txt \
  -CA ca-cert.pem -rkey ocsp-key.pem -rsigner ocsp-cert.pem \
  -text
```

### CRL Operations

```bash
# Generate CRL
openssl ca -config openssl.cnf -gencrl -out ca.crl

# View CRL
openssl crl -in ca.crl -text -noout

# Convert CRL PEM to DER
openssl crl -in ca.crl -outform DER -out ca.crl.der

# Convert CRL DER to PEM
openssl crl -in ca.crl.der -inform DER -out ca.crl.pem

# Revoke certificate and update CRL
openssl ca -config openssl.cnf -revoke certificate.crt \
  -crl_reason keyCompromise

# Generate updated CRL after revocation
openssl ca -config openssl.cnf -gencrl -out ca.crl

# Check if certificate is revoked
openssl verify -CAfile ca-cert.pem -CRLfile ca.crl -crl_check certificate.crt

# Extract revoked serial numbers
openssl crl -in ca.crl -text -noout | grep "Serial Number"
```

### Advanced TLS Connection Testing

```bash
# Test with SNI (Server Name Indication)
openssl s_client -connect example.com:443 -servername www.example.com

# Test without SNI (may get default cert)
openssl s_client -connect example.com:443 -noservername

# Test specific TLS versions
openssl s_client -connect example.com:443 -tls1
openssl s_client -connect example.com:443 -tls1_1
openssl s_client -connect example.com:443 -tls1_2
openssl s_client -connect example.com:443 -tls1_3

# Test specific cipher suite
openssl s_client -connect example.com:443 \
  -cipher 'ECDHE-RSA-AES256-GCM-SHA384'

# List all supported ciphers
openssl ciphers -v 'ALL:COMPLEMENTOFALL'
openssl ciphers -v 'HIGH:!aNULL:!MD5'
openssl ciphers -v 'TLSv1.3'

# Test cipher suites supported by server
for cipher in $(openssl ciphers 'ALL:eNULL' | tr ':' ' '); do
  echo -n "Testing $cipher... "
  result=$(echo -n | openssl s_client -cipher "$cipher" -connect example.com:443 2>&1)
  if echo "$result" | grep -q "Cipher is"; then
    echo "Supported"
  else
    echo "Not supported"
  fi
done

# Show full certificate chain
openssl s_client -connect example.com:443 -showcerts 2>&1 | \
  awk '/BEGIN CERT/,/END CERT/ {print}'

# Test with client certificate
openssl s_client -connect example.com:443 \
  -cert client-cert.pem -key client-key.pem -CAfile ca-bundle.crt

# Test STARTTLS for email (SMTP)
openssl s_client -connect mail.example.com:587 -starttls smtp

# Test STARTTLS for IMAP
openssl s_client -connect mail.example.com:143 -starttls imap

# Test STARTTLS for POP3
openssl s_client -connect mail.example.com:110 -starttls pop3

# Test STARTTLS for FTP
openssl s_client -connect ftp.example.com:21 -starttls ftp

# Test with proxy
openssl s_client -connect example.com:443 -proxy proxy.example.com:3128

# Show session details
openssl s_client -connect example.com:443 -tlsextdebug -status -msg

# Save session for reuse
openssl s_client -connect example.com:443 -sess_out session.pem

# Reuse saved session
openssl s_client -connect example.com:443 -sess_in session.pem

# Benchmark TLS handshake
openssl s_time -connect example.com:443 -time 10 -www /
```

### Certificate Fingerprints and Hashes

```bash
# SHA-256 fingerprint
openssl x509 -in certificate.crt -noout -fingerprint -sha256

# SHA-1 fingerprint (legacy)
openssl x509 -in certificate.crt -noout -fingerprint -sha1

# MD5 fingerprint (legacy)
openssl x509 -in certificate.crt -noout -fingerprint -md5

# Subject key identifier
openssl x509 -in certificate.crt -noout -text | grep -A1 "Subject Key Identifier"

# Authority key identifier
openssl x509 -in certificate.crt -noout -text | grep -A1 "Authority Key Identifier"

# Certificate serial number
openssl x509 -in certificate.crt -noout -serial

# Get serial number in decimal
openssl x509 -in certificate.crt -noout -serial | cut -d= -f2 | \
  python3 -c "import sys; print(int(sys.stdin.read().strip(), 16))"

# Public key fingerprint
openssl x509 -in certificate.crt -noout -pubkey | \
  openssl pkey -pubin -outform DER | \
  openssl dgst -sha256 -binary | \
  base64

# SPKI (Subject Public Key Info) fingerprint for pinning
openssl x509 -in certificate.crt -noout -pubkey | \
  openssl pkey -pubin -outform DER | \
  openssl dgst -sha256 -binary | \
  base64

# Extract public key and get fingerprint
openssl x509 -in certificate.crt -pubkey -noout | \
  openssl rsa -pubin -outform DER 2>/dev/null | \
  openssl dgst -sha256
```

### Batch Certificate Operations

```bash
# Process multiple certificates
for cert in *.crt; do
  echo "=== $cert ==="
  openssl x509 -in "$cert" -noout -subject -dates -fingerprint -sha256
  echo ""
done

# Find expiring certificates (within 30 days)
for cert in *.crt; do
  if openssl x509 -in "$cert" -noout -checkend 2592000; then
    :
  else
    echo "Certificate $cert expires within 30 days:"
    openssl x509 -in "$cert" -noout -subject -dates
  fi
done

# Extract all certificates from a bundle
csplit -s -f cert- ca-bundle.crt '/-----BEGIN CERTIFICATE-----/' '{*}'

# Merge multiple certificates into bundle
cat cert1.pem cert2.pem cert3.pem > ca-bundle.pem

# Sort certificates by expiration date
for cert in *.crt; do
  expiry=$(openssl x509 -in "$cert" -noout -enddate | cut -d= -f2)
  echo "$(date -d "$expiry" +%Y%m%d) $cert"
done | sort

# Batch convert PEM to DER
for pem in *.pem; do
  der="${pem%.pem}.der"
  openssl x509 -in "$pem" -outform DER -out "$der"
done
```

### Advanced PKCS Operations

```bash
# Create PKCS#12 with custom name and password
openssl pkcs12 -export -out certificate.p12 \
  -inkey private.key \
  -in certificate.crt \
  -certfile ca-chain.crt \
  -name "My Certificate" \
  -passout pass:mypassword

# Extract certificate chain from PKCS#12
openssl pkcs12 -in certificate.p12 -nokeys -out cert-chain.pem

# Extract private key from PKCS#12 (encrypted)
openssl pkcs12 -in certificate.p12 -nocerts -out private-encrypted.key

# Extract private key from PKCS#12 (unencrypted)
openssl pkcs12 -in certificate.p12 -nocerts -nodes -out private.key

# Extract CA certificates from PKCS#12
openssl pkcs12 -in certificate.p12 -cacerts -nokeys -out ca-certs.pem

# Extract only client certificate (not CA certs)
openssl pkcs12 -in certificate.p12 -clcerts -nokeys -out client-cert.pem

# View PKCS#12 contents
openssl pkcs12 -in certificate.p12 -info -noout

# Create PKCS#7 bundle
openssl crl2pkcs7 -nocrl \
  -certfile certificate.crt \
  -certfile ca-chain.crt \
  -out bundle.p7b

# Extract certificates from PKCS#7
openssl pkcs7 -in bundle.p7b -print_certs -out certificates.pem

# Verify PKCS#7 signature
openssl smime -verify -in signed.p7s -inform PEM \
  -certfile ca-cert.pem -CAfile ca-bundle.crt

# Create signed PKCS#7
openssl smime -sign -in file.txt -out signed.p7s \
  -signer certificate.crt -inkey private.key
```

### Certificate Inspection and Parsing

```bash
# Extract specific fields
openssl x509 -in certificate.crt -noout -subject
openssl x509 -in certificate.crt -noout -issuer
openssl x509 -in certificate.crt -noout -dates
openssl x509 -in certificate.crt -noout -serial
openssl x509 -in certificate.crt -noout -email

# Extract SANs
openssl x509 -in certificate.crt -noout -ext subjectAltName

# Extract all extensions
openssl x509 -in certificate.crt -noout -text | grep -A 50 "X509v3 extensions:"

# Extract specific extension
openssl x509 -in certificate.crt -noout -ext keyUsage
openssl x509 -in certificate.crt -noout -ext extendedKeyUsage
openssl x509 -in certificate.crt -noout -ext basicConstraints

# Get certificate in different text formats
openssl x509 -in certificate.crt -text
openssl x509 -in certificate.crt -text -noout  # Without PEM
openssl x509 -in certificate.crt -subject -noout
openssl x509 -in certificate.crt -issuer -noout
openssl x509 -in certificate.crt -dates -noout
openssl x509 -in certificate.crt -purpose -noout

# Extract public key
openssl x509 -in certificate.crt -noout -pubkey
openssl x509 -in certificate.crt -noout -pubkey -out pubkey.pem

# Get certificate signature algorithm
openssl x509 -in certificate.crt -noout -text | grep "Signature Algorithm"

# Check if certificate is CA
openssl x509 -in certificate.crt -noout -text | grep "CA:TRUE"

# Get key usage
openssl x509 -in certificate.crt -noout -text | grep -A1 "Key Usage"

# Parse certificate as JSON (using custom script)
openssl x509 -in certificate.crt -text -noout | \
  awk '/Subject:/ {print "subject:", $0} /Issuer:/ {print "issuer:", $0}'
```

### SSL/TLS Protocol Testing

```bash
# Test SSLv3 (should fail - insecure)
openssl s_client -connect example.com:443 -ssl3 2>&1 | grep -E "Protocol|Cipher"

# Test all TLS versions
for version in tls1 tls1_1 tls1_2 tls1_3; do
  echo "Testing $version..."
  openssl s_client -connect example.com:443 -$version < /dev/null 2>&1 | \
    grep -E "Protocol|Cipher"
done

# Check for vulnerabilities
# BEAST (CBC ciphers in TLS 1.0)
openssl s_client -connect example.com:443 -tls1 -cipher 'AES' 2>&1 | grep Cipher

# POODLE (SSLv3)
openssl s_client -connect example.com:443 -ssl3 2>&1 | grep "Protocol"

# Heartbleed test (requires specific OpenSSL build)
# Use external tools like nmap or dedicated scanners

# Test for weak ciphers
for cipher in RC4 3DES MD5 EXPORT NULL aNULL eNULL; do
  echo "Testing $cipher..."
  openssl s_client -connect example.com:443 -cipher "$cipher" < /dev/null 2>&1 | \
    grep "Cipher is"
done

# Test cipher order preference
openssl s_client -connect example.com:443 -cipher 'ALL' 2>&1 | \
  grep "Cipher is"
```

### Performance and Benchmarking

```bash
# Benchmark RSA key generation
openssl speed rsa2048
openssl speed rsa4096

# Benchmark ECDSA
openssl speed ecdsap256
openssl speed ecdsap384

# Benchmark ciphers
openssl speed aes-256-gcm
openssl speed aes-128-gcm
openssl speed chacha20-poly1305

# Benchmark TLS handshakes
openssl s_time -connect example.com:443 -time 30 -www /

# Benchmark with different ciphers
openssl s_time -connect example.com:443 -time 10 \
  -cipher ECDHE-RSA-AES256-GCM-SHA384 -www /

# Test connection reuse
openssl s_time -connect example.com:443 -time 10 -reuse -www /
```

### Certificate Chain Validation

```bash
# Validate complete chain
openssl verify -CAfile root-ca.pem -untrusted intermediate-ca.pem server-cert.pem

# Build and validate chain from server
openssl s_client -connect example.com:443 -showcerts 2>&1 | \
  awk '/BEGIN CERT/,/END CERT/ {print}' > fullchain.pem
openssl verify -CAfile /etc/ssl/certs/ca-certificates.crt fullchain.pem

# Extract individual certificates from chain
csplit -s -f cert- fullchain.pem '/-----BEGIN CERTIFICATE-----/' '{*}'

# Verify chain order
i=0
for cert in cert-*[0-9]; do
  echo "=== Certificate $i ==="
  openssl x509 -in "$cert" -noout -subject -issuer
  echo ""
  ((i++))
done

# Check for missing intermediate
openssl s_client -connect example.com:443 2>&1 | \
  grep "verify error:num=20"  # "unable to get local issuer certificate"

# Download intermediate certificate
curl -o intermediate.crt http://ca.example.com/intermediate.crt

# Rebuild chain with downloaded intermediate
cat server.crt intermediate.crt > chain.crt
openssl verify -CAfile root-ca.pem chain.crt
```

### Wildcard and Multi-Domain Certificates

```bash
# Create wildcard certificate CSR
openssl req -new -key private.key -out wildcard.csr \
  -subj "/C=US/ST=CA/O=Example Inc/CN=*.example.com" \
  -addext "subjectAltName=DNS:*.example.com,DNS:example.com"

# Create multi-domain (SAN) certificate CSR
openssl req -new -key private.key -out multidomain.csr \
  -config <(cat <<EOF
[req]
default_bits = 2048
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[dn]
C=US
ST=California
O=Example Inc
CN=example.com

[req_ext]
subjectAltName = @alt_names

[alt_names]
DNS.1 = example.com
DNS.2 = www.example.com
DNS.3 = mail.example.com
DNS.4 = ftp.example.com
DNS.5 = example.net
DNS.6 = www.example.net
DNS.7 = *.api.example.com
EOF
)

# Verify wildcard certificate matches domain
openssl s_client -connect sub.example.com:443 -servername sub.example.com 2>&1 | \
  openssl x509 -noout -text | grep -A1 "Subject Alternative Name"

# Test various subdomains with wildcard
for subdomain in www api mail cdn; do
  echo "Testing $subdomain.example.com..."
  openssl s_client -connect example.com:443 \
    -servername "$subdomain.example.com" < /dev/null 2>&1 | \
    grep "Verify return code"
done
```

### Automated Certificate Monitoring

```bash
#!/bin/bash
# Certificate expiration monitoring script

CERT_FILE="/path/to/certificate.crt"
WARN_DAYS=30
ALERT_DAYS=7

# Get expiration date
EXPIRY_DATE=$(openssl x509 -in "$CERT_FILE" -noout -enddate | cut -d= -f2)
EXPIRY_EPOCH=$(date -d "$EXPIRY_DATE" +%s)
CURRENT_EPOCH=$(date +%s)
DAYS_LEFT=$(( ($EXPIRY_EPOCH - $CURRENT_EPOCH) / 86400 ))

echo "Certificate: $CERT_FILE"
echo "Expires: $EXPIRY_DATE"
echo "Days remaining: $DAYS_LEFT"

if [ $DAYS_LEFT -lt $ALERT_DAYS ]; then
  echo "CRITICAL: Certificate expires in $DAYS_LEFT days!"
  # Send alert
elif [ $DAYS_LEFT -lt $WARN_DAYS ]; then
  echo "WARNING: Certificate expires in $DAYS_LEFT days"
  # Send warning
else
  echo "OK: Certificate valid for $DAYS_LEFT days"
fi

# Check certificate for common issues
echo ""
echo "=== Certificate Validation ==="

# Check if self-signed
if openssl verify -CAfile "$CERT_FILE" "$CERT_FILE" 2>&1 | grep -q "self signed"; then
  echo "WARNING: Certificate is self-signed"
fi

# Check key size
KEY_SIZE=$(openssl x509 -in "$CERT_FILE" -noout -text | \
  grep "Public-Key:" | grep -oE '[0-9]+')
if [ "$KEY_SIZE" -lt 2048 ]; then
  echo "WARNING: Key size ($KEY_SIZE bit) is less than 2048 bits"
fi

# Check signature algorithm
SIG_ALG=$(openssl x509 -in "$CERT_FILE" -noout -text | \
  grep "Signature Algorithm" | head -1 | awk '{print $3}')
if [[ "$SIG_ALG" == *"sha1"* ]]; then
  echo "WARNING: Using weak signature algorithm (SHA-1)"
fi

# Online certificate monitoring for domain
DOMAIN="example.com"
echo ""
echo "=== Checking ${DOMAIN} ==="

# Get certificate from server
SERVER_CERT=$(echo | openssl s_client -connect ${DOMAIN}:443 -servername ${DOMAIN} 2>/dev/null)

# Extract expiry
echo "$SERVER_CERT" | openssl x509 -noout -dates

# Check chain
echo "$SERVER_CERT" | openssl x509 -noout -text | grep -A1 "CA Issuers"

# Get OCSP status
OCSP_URI=$(echo "$SERVER_CERT" | openssl x509 -noout -ocsp_uri)
if [ -n "$OCSP_URI" ]; then
  echo "OCSP URI: $OCSP_URI"
  # Perform OCSP check
  echo "$SERVER_CERT" | \
    openssl ocsp -issuer <(echo "$SERVER_CERT" | openssl x509) \
    -cert <(echo "$SERVER_CERT" | openssl x509) \
    -url "$OCSP_URI" -noverify 2>/dev/null | grep "Response Status"
fi
```

### Security Scanning with OpenSSL

```bash
# Comprehensive TLS security scan
#!/bin/bash

TARGET="example.com:443"
echo "=== TLS Security Scan for $TARGET ==="

# Protocol versions
echo ""
echo "Protocol Versions:"
for proto in ssl3 tls1 tls1_1 tls1_2 tls1_3; do
  result=$(openssl s_client -connect $TARGET -$proto < /dev/null 2>&1)
  if echo "$result" | grep -q "Cipher is"; then
    echo "  $proto: ENABLED"
  else
    echo "  $proto: disabled"
  fi
done

# Weak ciphers
echo ""
echo "Weak Cipher Tests:"
for cipher in 'RC4' '3DES' 'EXPORT' 'NULL' 'aNULL' 'eNULL' 'MD5'; do
  result=$(openssl s_client -connect $TARGET -cipher "$cipher" < /dev/null 2>&1)
  if echo "$result" | grep -q "Cipher is ${cipher}"; then
    echo "  WARNING: $cipher is supported"
  else
    echo "  $cipher: not supported (good)"
  fi
done

# Certificate validation
echo ""
echo "Certificate Validation:"
CERT=$(openssl s_client -connect $TARGET < /dev/null 2>&1)

# Extract and check certificate
echo "$CERT" | openssl x509 -noout -dates
echo "$CERT" | openssl x509 -noout -subject
echo "$CERT" | openssl x509 -noout -issuer

# Check key size
KEY_SIZE=$(echo "$CERT" | openssl x509 -noout -text | \
  grep "Public-Key:" | grep -oE '[0-9]+')
echo "Key Size: $KEY_SIZE bits"
if [ "$KEY_SIZE" -lt 2048 ]; then
  echo "  WARNING: Key size is less than 2048 bits"
fi

# Check for common vulnerabilities
echo ""
echo "Vulnerability Checks:"

# Heartbleed indicator
if echo "$CERT" | grep -q "TLS server extension \"heartbeat\""; then
  echo "  Heartbeat extension present (potential Heartbleed)"
fi

# Check for compression
if echo "$CERT" | grep -q "Compression: NONE"; then
  echo "  Compression: disabled (good - CRIME mitigation)"
else
  echo "  WARNING: Compression may be enabled (CRIME vulnerability)"
fi

# Check for session resumption
echo ""
echo "Session Resumption:"
openssl s_client -connect $TARGET -reconnect 2>&1 | grep "Reused"

# Check OCSP stapling
echo ""
echo "OCSP Stapling:"
openssl s_client -connect $TARGET -status < /dev/null 2>&1 | \
  grep "OCSP Response Status"
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
