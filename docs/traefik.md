# Traefik - Cloud Native Edge Router

A comprehensive guide to Traefik - a modern HTTP reverse proxy and load balancer for microservices.

## Table of Contents

- [Introduction](#introduction)
- [Installation](#installation)
- [Core Concepts](#core-concepts)
- [Configuration](#configuration)
- [Kubernetes Ingress](#kubernetes-ingress)
- [Middlewares](#middlewares)
- [TLS & Certificates](#tls--certificates)
- [Load Balancing](#load-balancing)
- [Observability](#observability)
- [Best Practices](#best-practices)

## Introduction

### What is Traefik?

Traefik is a modern HTTP reverse proxy and load balancer designed for deploying microservices. It integrates seamlessly with existing infrastructure components and configures itself automatically and dynamically.

### Key Features

- **Automatic Service Discovery**: Auto-configuration from service registries
- **Let's Encrypt Integration**: Automatic SSL certificate provisioning
- **Multiple Protocols**: HTTP, HTTPS, TCP, UDP
- **Middleware System**: Modular request/response transformations
- **Load Balancing**: Multiple algorithms and health checks
- **Metrics & Tracing**: Prometheus, Datadog, Jaeger integration
- **High Availability**: Clustering support
- **WebSocket Support**: Native WebSocket proxying

## Installation

### Docker

```bash
# Run Traefik in Docker
docker run -d \
  -p 80:80 \
  -p 443:443 \
  -p 8080:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $PWD/traefik.yml:/etc/traefik/traefik.yml \
  --name traefik \
  traefik:v3.0

# Docker Compose
version: '3'

services:
  traefik:
    image: traefik:v3.0
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"  # Dashboard
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
```

### Kubernetes (Helm)

```bash
# Add Helm repository
helm repo add traefik https://traefik.github.io/charts
helm repo update

# Install Traefik
helm install traefik traefik/traefik \
  --namespace traefik \
  --create-namespace \
  --set dashboard.enabled=true \
  --set rbac.enabled=true

# Custom values
helm install traefik traefik/traefik \
  --namespace traefik \
  --create-namespace \
  --values traefik-values.yaml
```

### Traefik Helm Values

```yaml
# traefik-values.yaml
# Deployment
deployment:
  replicas: 3

# Enable dashboard
dashboard:
  enabled: true
  domain: traefik.example.com

# Entry points
ports:
  web:
    port: 80
    exposedPort: 80
    protocol: TCP
  websecure:
    port: 443
    exposedPort: 443
    protocol: TCP
    tls:
      enabled: true
  metrics:
    port: 9100
    expose: true
    protocol: TCP

# Providers
providers:
  kubernetesCRD:
    enabled: true
  kubernetesIngress:
    enabled: true
    publishedService:
      enabled: true

# Let's Encrypt
certificatesResolvers:
  letsencrypt:
    acme:
      email: admin@example.com
      storage: /data/acme.json
      httpChallenge:
        entryPoint: web

# Metrics
metrics:
  prometheus:
    enabled: true
    addEntryPointsLabels: true
    addServicesLabels: true

# Logs
logs:
  general:
    level: INFO
  access:
    enabled: true
```

### Binary Installation

```bash
# Download Traefik binary
wget https://github.com/traefik/traefik/releases/download/v3.0.0/traefik_v3.0.0_linux_amd64.tar.gz
tar -xzf traefik_v3.0.0_linux_amd64.tar.gz
sudo mv traefik /usr/local/bin/

# Make executable
sudo chmod +x /usr/local/bin/traefik

# Verify
traefik version
```

## Core Concepts

### Architecture

```
Client Request
     ↓
Entry Points (Ports: 80, 443, etc.)
     ↓
Routers (Match requests based on rules)
     ↓
Middlewares (Transform request/response)
     ↓
Services (Load balance to backends)
     ↓
Backend Servers
```

### Providers

Traefik discovers services from providers:

- **Docker**: Labels on containers
- **Kubernetes**: Ingress, IngressRoute CRDs
- **File**: Static configuration files
- **Consul**: Service discovery
- **Etcd**: Distributed configuration
- **HTTP**: API-based configuration

### Routers

Match requests and forward to services:

```yaml
# HTTP Router
http:
  routers:
    my-router:
      rule: "Host(`example.com`) && Path(`/api`)"
      service: my-service
      middlewares:
        - auth
        - compress
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt
```

### Services

Define how to reach backends:

```yaml
http:
  services:
    my-service:
      loadBalancer:
        servers:
          - url: "http://backend1:8080"
          - url: "http://backend2:8080"
        healthCheck:
          path: /health
          interval: 10s
          timeout: 3s
```

## Configuration

### Static Configuration

```yaml
# traefik.yml (Static Configuration)
global:
  checkNewVersion: true
  sendAnonymousUsage: false

# API and Dashboard
api:
  dashboard: true
  insecure: false

# Entry Points
entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https

  websecure:
    address: ":443"
    http:
      tls:
        certResolver: letsencrypt

# Providers
providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
    network: web

  file:
    directory: /etc/traefik/dynamic
    watch: true

# Certificate Resolvers
certificatesResolvers:
  letsencrypt:
    acme:
      email: admin@example.com
      storage: /letsencrypt/acme.json
      httpChallenge:
        entryPoint: web

# Logging
log:
  level: INFO
  filePath: /var/log/traefik/traefik.log

accessLog:
  filePath: /var/log/traefik/access.log
  format: json

# Metrics
metrics:
  prometheus:
    buckets:
      - 0.1
      - 0.3
      - 1.2
      - 5.0
```

### Dynamic Configuration

```yaml
# dynamic.yml
http:
  routers:
    api-router:
      rule: "Host(`api.example.com`)"
      service: api-service
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt
      middlewares:
        - rate-limit
        - auth

  services:
    api-service:
      loadBalancer:
        servers:
          - url: "http://192.168.1.100:8080"
          - url: "http://192.168.1.101:8080"
        healthCheck:
          path: /health
          interval: "10s"

  middlewares:
    rate-limit:
      rateLimit:
        average: 100
        burst: 50

    auth:
      basicAuth:
        users:
          - "user:$apr1$xxx"  # Use htpasswd
```

## Kubernetes Ingress

### Standard Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: websecure
    traefik.ingress.kubernetes.io/router.tls: "true"
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: traefik
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp
            port:
              number: 80
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
```

### IngressRoute CRD

```yaml
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: myapp-ingressroute
  namespace: default
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`myapp.example.com`) && PathPrefix(`/api`)
    kind: Rule
    services:
    - name: myapp-api
      port: 8080
    middlewares:
    - name: api-auth
    - name: rate-limit

  - match: Host(`myapp.example.com`)
    kind: Rule
    services:
    - name: myapp-web
      port: 80

  tls:
    certResolver: letsencrypt
    domains:
    - main: myapp.example.com
      sans:
      - www.myapp.example.com
```

### TCP IngressRoute

```yaml
apiVersion: traefik.io/v1alpha1
kind: IngressRouteTCP
metadata:
  name: postgres-ingressroute
spec:
  entryPoints:
    - postgres
  routes:
  - match: HostSNI(`*`)
    services:
    - name: postgres
      port: 5432
```

## Middlewares

### Authentication

```yaml
# Basic Auth
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: basic-auth
spec:
  basicAuth:
    secret: auth-secret  # Contains user:password (htpasswd format)

---
# Forward Auth (OAuth2, SSO)
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: oauth2-proxy
spec:
  forwardAuth:
    address: http://oauth2-proxy.auth.svc:4180
    authResponseHeaders:
      - X-Auth-User
      - X-Auth-Email
```

### Rate Limiting

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: rate-limit
spec:
  rateLimit:
    average: 100  # Requests per second
    burst: 50     # Burst size
    period: 1s
```

### Headers

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: security-headers
spec:
  headers:
    customResponseHeaders:
      X-Frame-Options: "SAMEORIGIN"
      X-Content-Type-Options: "nosniff"
      X-XSS-Protection: "1; mode=block"
      Referrer-Policy: "strict-origin-when-cross-origin"
    stsSeconds: 31536000
    stsIncludeSubdomains: true
    stsPreload: true
```

### Compression

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: compress
spec:
  compress: {}
```

### Redirect

```yaml
# HTTPS Redirect
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: redirect-https
spec:
  redirectScheme:
    scheme: https
    permanent: true

---
# WWW Redirect
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: redirect-www
spec:
  redirectRegex:
    regex: "^https://example.com/(.*)"
    replacement: "https://www.example.com/${1}"
    permanent: true
```

### Strip Prefix

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: strip-api-prefix
spec:
  stripPrefix:
    prefixes:
      - /api
```

### IP Whitelist

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: ip-whitelist
spec:
  ipWhiteList:
    sourceRange:
      - "192.168.1.0/24"
      - "172.16.0.1"
```

### Circuit Breaker

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: circuit-breaker
spec:
  circuitBreaker:
    expression: "NetworkErrorRatio() > 0.30"
```

### Retry

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: retry
spec:
  retry:
    attempts: 4
    initialInterval: 100ms
```

## TLS & Certificates

### Let's Encrypt

```yaml
# traefik.yml
certificatesResolvers:
  letsencrypt:
    acme:
      email: admin@example.com
      storage: /letsencrypt/acme.json
      # HTTP Challenge
      httpChallenge:
        entryPoint: web
      # DNS Challenge (for wildcards)
      # dnsChallenge:
      #   provider: cloudflare
      #   delayBeforeCheck: 0
```

### Custom TLS Certificate

```yaml
# Create TLS secret
kubectl create secret tls myapp-tls \
  --cert=cert.pem \
  --key=key.pem

# Use in IngressRoute
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: myapp
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`myapp.example.com`)
    kind: Rule
    services:
    - name: myapp
      port: 80
  tls:
    secretName: myapp-tls
```

### TLS Options

```yaml
apiVersion: traefik.io/v1alpha1
kind: TLSOption
metadata:
  name: modern-tls
spec:
  minVersion: VersionTLS12
  cipherSuites:
    - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
    - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
  curvePreferences:
    - CurveP521
    - CurveP384
  sniStrict: true
```

## Load Balancing

### Algorithms

```yaml
http:
  services:
    my-service:
      loadBalancer:
        # Round Robin (default)
        servers:
          - url: "http://backend1:8080"
          - url: "http://backend2:8080"

        # Weighted Round Robin
        # servers:
        #   - url: "http://backend1:8080"
        #     weight: 3
        #   - url: "http://backend2:8080"
        #     weight: 1

        # Sticky Sessions
        sticky:
          cookie:
            name: traefik_session
            secure: true
            httpOnly: true

        # Health Check
        healthCheck:
          path: /health
          interval: "10s"
          timeout: "3s"
          scheme: http
```

### Mirroring (Traffic Shadowing)

```yaml
apiVersion: traefik.io/v1alpha1
kind: TraefikService
metadata:
  name: mirror-service
spec:
  mirroring:
    name: production
    port: 80
    mirrors:
    - name: canary
      port: 80
      percent: 10
```

### Weighted Round Robin

```yaml
apiVersion: traefik.io/v1alpha1
kind: TraefikService
metadata:
  name: weighted-service
spec:
  weighted:
    services:
    - name: app-v1
      weight: 90
      port: 80
    - name: app-v2
      weight: 10
      port: 80
```

## Observability

### Prometheus Metrics

```yaml
# Enable metrics in traefik.yml
metrics:
  prometheus:
    buckets:
      - 0.1
      - 0.3
      - 1.2
      - 5.0
    addEntryPointsLabels: true
    addServicesLabels: true
    addRoutersLabels: true

# ServiceMonitor for Prometheus Operator
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: traefik
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: traefik
  endpoints:
  - port: metrics
    interval: 30s
```

### Access Logs

```yaml
accessLog:
  filePath: /var/log/traefik/access.log
  format: json
  filters:
    statusCodes:
      - "400-499"
      - "500-599"
    retryAttempts: true
    minDuration: "10ms"
  fields:
    defaultMode: keep
    headers:
      defaultMode: drop
      names:
        User-Agent: keep
        Authorization: drop
```

### Tracing

```yaml
# Jaeger tracing
tracing:
  jaeger:
    samplingServerURL: http://jaeger:5778/sampling
    localAgentHostPort: jaeger:6831

# Zipkin tracing
tracing:
  zipkin:
    httpEndpoint: http://zipkin:9411/api/v2/spans
    sameSpan: true
    id128Bit: true
```

## Advanced Configuration Examples

### Multi-Domain Routing with Path Matching

```yaml
# Complex routing with multiple domains and path conditions
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: complex-routing
spec:
  entryPoints:
    - websecure
  routes:
  # API v1 on api.example.com
  - match: Host(`api.example.com`) && PathPrefix(`/v1/users`)
    kind: Rule
    priority: 100
    services:
    - name: user-service-v1
      port: 8080
    middlewares:
    - name: api-v1-auth
    - name: rate-limit-api

  # API v2 on api.example.com
  - match: Host(`api.example.com`) && PathPrefix(`/v2/users`)
    kind: Rule
    priority: 100
    services:
    - name: user-service-v2
      port: 8080
    middlewares:
    - name: api-v2-auth
    - name: rate-limit-api

  # Admin panel with IP restriction
  - match: Host(`admin.example.com`)
    kind: Rule
    priority: 90
    services:
    - name: admin-panel
      port: 80
    middlewares:
    - name: admin-ip-whitelist
    - name: admin-basic-auth

  # Static assets with caching
  - match: Host(`static.example.com`) && PathPrefix(`/assets`)
    kind: Rule
    priority: 80
    services:
    - name: static-server
      port: 80
    middlewares:
    - name: cache-headers
    - name: compress

  # Default route
  - match: Host(`example.com`) || Host(`www.example.com`)
    kind: Rule
    priority: 50
    services:
    - name: main-website
      port: 80
    middlewares:
    - name: www-redirect

  tls:
    certResolver: letsencrypt
    domains:
    - main: example.com
      sans:
      - "*.example.com"
```

### Canary Deployment with Traffic Splitting

```yaml
# Weighted service for canary releases
apiVersion: traefik.io/v1alpha1
kind: TraefikService
metadata:
  name: canary-deployment
  namespace: production
spec:
  weighted:
    services:
    - name: app-stable
      weight: 95
      port: 8080
      kind: Service
    - name: app-canary
      weight: 5
      port: 8080
      kind: Service

---
# IngressRoute using the weighted service
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: app-canary-route
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`app.example.com`)
    kind: Rule
    services:
    - name: canary-deployment
      kind: TraefikService
    middlewares:
    - name: canary-headers
  tls:
    certResolver: letsencrypt

---
# Middleware to add canary tracking headers
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: canary-headers
spec:
  headers:
    customResponseHeaders:
      X-Canary-Version: "v2.0.0-canary"
```

### Blue-Green Deployment

```yaml
# Blue environment (current production)
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: app-blue
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`app.example.com`)
    kind: Rule
    services:
    - name: app-blue
      port: 8080
  tls:
    certResolver: letsencrypt

---
# Green environment (new version - testing via header)
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: app-green-testing
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`app.example.com`) && HeadersRegexp(`X-Environment`, `green`)
    kind: Rule
    priority: 100
    services:
    - name: app-green
      port: 8080
  tls:
    certResolver: letsencrypt

---
# Switch to green (update the main route)
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: app-production
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`app.example.com`)
    kind: Rule
    services:
    - name: app-green  # Changed from app-blue
      port: 8080
  tls:
    certResolver: letsencrypt
```

### Advanced Middleware Chaining

```yaml
# Complete middleware chain for production API
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: api-production
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`api.example.com`)
    kind: Rule
    services:
    - name: api-backend
      port: 8080
    middlewares:
    - name: security-headers        # 1. Add security headers
    - name: ip-whitelist           # 2. Check IP whitelist
    - name: rate-limit-global      # 3. Global rate limit
    - name: rate-limit-per-ip      # 4. Per-IP rate limit
    - name: oauth2-auth            # 5. OAuth2 authentication
    - name: strip-prefix           # 6. Strip /api prefix
    - name: add-correlation-id     # 7. Add correlation ID
    - name: compress               # 8. Compress response
    - name: circuit-breaker        # 9. Circuit breaker
    - name: retry                  # 10. Retry on failure
  tls:
    certResolver: letsencrypt

---
# Add correlation ID for tracing
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: add-correlation-id
spec:
  headers:
    customRequestHeaders:
      X-Correlation-ID: "{{uuid}}"
```

### Custom Error Pages

```yaml
# Custom error page middleware
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: custom-errors
spec:
  errors:
    status:
      - "400-599"
    service:
      name: error-pages
      port: 80
    query: "/{status}.html"

---
# Error pages service deployment
apiVersion: v1
kind: ConfigMap
metadata:
  name: error-pages-html
data:
  404.html: |
    <!DOCTYPE html>
    <html>
    <head><title>404 Not Found</title></head>
    <body>
      <h1>404 - Page Not Found</h1>
      <p>The resource you requested could not be found.</p>
    </body>
    </html>

  500.html: |
    <!DOCTYPE html>
    <html>
    <head><title>500 Internal Server Error</title></head>
    <body>
      <h1>500 - Internal Server Error</h1>
      <p>Something went wrong. Please try again later.</p>
    </body>
    </html>

  503.html: |
    <!DOCTYPE html>
    <html>
    <head><title>503 Service Unavailable</title></head>
    <body>
      <h1>503 - Service Unavailable</h1>
      <p>The service is temporarily unavailable. Please try again soon.</p>
    </body>
    </html>
```

### Request/Response Modification

```yaml
# Add custom headers based on conditions
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: custom-headers
spec:
  headers:
    # Custom request headers
    customRequestHeaders:
      X-Forwarded-Proto: "https"
      X-Custom-Header: "CustomValue"
      X-Real-IP: "{{.RemoteAddr}}"

    # Custom response headers
    customResponseHeaders:
      X-Custom-Response: "API-Gateway"
      X-Frame-Options: "SAMEORIGIN"
      X-Content-Type-Options: "nosniff"
      X-XSS-Protection: "1; mode=block"
      Content-Security-Policy: "default-src 'self'"
      Referrer-Policy: "strict-origin-when-cross-origin"
      Permissions-Policy: "geolocation=(), microphone=(), camera=()"

    # SSL Headers
    stsSeconds: 63072000
    stsIncludeSubdomains: true
    stsPreload: true
    forceSTSHeader: true

    # CORS headers
    accessControlAllowMethods:
      - "GET"
      - "POST"
      - "PUT"
      - "DELETE"
      - "OPTIONS"
    accessControlAllowOriginList:
      - "https://app.example.com"
      - "https://dashboard.example.com"
    accessControlAllowCredentials: true
    accessControlMaxAge: 100

    # Remove headers
    customRequestHeadersRemove:
      - "X-Powered-By"
      - "Server"

---
# URL rewriting middleware
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: url-rewrite
spec:
  replacePathRegex:
    regex: "^/old-api/(.*)"
    replacement: "/new-api/$1"

---
# Add prefix middleware
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: add-prefix
spec:
  addPrefix:
    prefix: "/api/v1"
```

### gRPC Routing

```yaml
# gRPC service routing
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: grpc-service
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`grpc.example.com`) && Headers(`Content-Type`, `application/grpc`)
    kind: Rule
    services:
    - name: grpc-backend
      port: 50051
      scheme: h2c  # HTTP/2 cleartext
    middlewares:
    - name: grpc-headers
  tls:
    certResolver: letsencrypt

---
# gRPC specific middleware
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: grpc-headers
spec:
  headers:
    customRequestHeaders:
      X-Request-ID: "{{uuid}}"
    customResponseHeaders:
      X-gRPC-Version: "1.0"

---
# gRPC with TLS (h2)
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: grpc-service-tls
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`grpc-secure.example.com`)
    kind: Rule
    services:
    - name: grpc-backend-tls
      port: 50051
      scheme: h2  # HTTP/2 with TLS
  tls:
    certResolver: letsencrypt
```

### WebSocket Configuration

```yaml
# WebSocket routing
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: websocket-app
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`ws.example.com`)
    kind: Rule
    services:
    - name: websocket-backend
      port: 8080
    middlewares:
    - name: websocket-headers
  tls:
    certResolver: letsencrypt

---
# WebSocket headers middleware
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: websocket-headers
spec:
  headers:
    customRequestHeaders:
      X-Forwarded-Proto: "wss"
      Connection: "upgrade"
      Upgrade: "websocket"

---
# File provider config for WebSocket
# dynamic.yml
http:
  routers:
    websocket:
      rule: "Host(`ws.example.com`) && PathPrefix(`/ws`)"
      service: websocket-service
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt

  services:
    websocket-service:
      loadBalancer:
        servers:
          - url: "http://websocket-backend:8080"
        # Important: Disable buffering for WebSocket
        passHostHeader: true
        responseForwarding:
          flushInterval: "1ms"
```

### TCP/UDP Routing

```yaml
# TCP routing for databases
apiVersion: traefik.io/v1alpha1
kind: IngressRouteTCP
metadata:
  name: postgres-tcp
spec:
  entryPoints:
    - postgres-entry
  routes:
  - match: HostSNI(`db.example.com`)
    services:
    - name: postgres
      port: 5432
  tls:
    passthrough: true

---
# TCP routing for Redis
apiVersion: traefik.io/v1alpha1
kind: IngressRouteTCP
metadata:
  name: redis-tcp
spec:
  entryPoints:
    - redis-entry
  routes:
  - match: HostSNI(`*`)
    services:
    - name: redis
      port: 6379

---
# UDP routing for DNS
apiVersion: traefik.io/v1alpha1
kind: IngressRouteUDP
metadata:
  name: dns-udp
spec:
  entryPoints:
    - dns-udp
  routes:
  - services:
    - name: coredns
      port: 53

---
# Static config for TCP/UDP entry points
# traefik.yml
entryPoints:
  postgres-entry:
    address: ":5432"

  redis-entry:
    address: ":6379"

  dns-udp:
    address: ":53/udp"

  mysql-entry:
    address: ":3306"
```

### Advanced Rate Limiting Strategies

```yaml
# Global rate limiting
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: rate-limit-global
spec:
  rateLimit:
    average: 1000
    burst: 2000
    period: 1s

---
# Per-IP rate limiting
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: rate-limit-per-ip
spec:
  rateLimit:
    average: 100
    burst: 200
    period: 1s
    sourceCriterion:
      ipStrategy:
        depth: 1  # Use first IP in X-Forwarded-For

---
# API endpoint specific rate limits
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: rate-limit-api-strict
spec:
  rateLimit:
    average: 10
    burst: 20
    period: 1m

---
# Rate limit with IP exclusions (use chain)
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: rate-limit-with-exclusions
spec:
  chain:
    middlewares:
    - name: trusted-ip-whitelist
    - name: rate-limit-general

---
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: trusted-ip-whitelist
spec:
  ipWhiteList:
    sourceRange:
      - "10.0.0.0/8"      # Internal network
      - "172.16.0.0/12"   # Private network
      - "192.168.0.0/16"  # Local network
```

### Advanced Authentication Patterns

```yaml
# OAuth2 Proxy for SSO
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: oauth2-proxy
spec:
  forwardAuth:
    address: "http://oauth2-proxy.auth.svc.cluster.local:4180"
    trustForwardHeader: true
    authResponseHeaders:
      - "X-Auth-Request-User"
      - "X-Auth-Request-Email"
      - "X-Auth-Request-Access-Token"
      - "Authorization"

---
# OIDC Authentication (Keycloak)
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: keycloak-auth
spec:
  forwardAuth:
    address: "http://keycloak-gatekeeper.auth.svc:3000"
    trustForwardHeader: true
    authResponseHeaders:
      - "X-Auth-Username"
      - "X-Auth-Userid"
      - "X-Auth-Email"
      - "X-Auth-Groups"

---
# Basic Auth with multiple users
apiVersion: v1
kind: Secret
metadata:
  name: auth-secret
type: Opaque
stringData:
  users: |
    admin:$apr1$H6uskkkW$IgXLP6ewTrSuBkTrqE8wj/
    user1:$apr1$ruca84Hq$mbjdMZBAG.KWn7vfN/SNK/
    user2:$apr1$H6uskkkW$IgXLP6ewTrSuBkTrqE8wj0

---
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: basic-auth-multi
spec:
  basicAuth:
    secret: auth-secret
    realm: "Protected Area"
    removeHeader: true

---
# Digest authentication
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: digest-auth
spec:
  digestAuth:
    secret: digest-secret
    realm: "Secure API"
    removeHeader: false

---
# IP-based authentication bypass
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: smart-auth
spec:
  chain:
    middlewares:
    - name: internal-ip-check
    - name: oauth2-proxy

---
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: internal-ip-check
spec:
  ipWhiteList:
    sourceRange:
      - "10.0.0.0/8"
    # If IP is in whitelist, skip subsequent auth
```

### Traffic Mirroring (Shadowing)

```yaml
# Mirror production traffic to test environment
apiVersion: traefik.io/v1alpha1
kind: TraefikService
metadata:
  name: mirror-to-test
spec:
  mirroring:
    name: production-api
    port: 8080
    mirrors:
    - name: test-api
      port: 8080
      percent: 100  # Mirror 100% of traffic
    - name: analytics-api
      port: 8080
      percent: 10   # Send 10% to analytics

---
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: api-with-mirroring
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`api.example.com`)
    kind: Rule
    services:
    - name: mirror-to-test
      kind: TraefikService
  tls:
    certResolver: letsencrypt
```

### Multi-Cluster Setup

```yaml
# Primary cluster routing
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: multi-cluster-route
  namespace: traefik
spec:
  entryPoints:
    - websecure
  routes:
  # Route to primary cluster by default
  - match: Host(`app.example.com`)
    kind: Rule
    priority: 50
    services:
    - name: app-primary
      namespace: production
      port: 8080

  # Route to secondary cluster via header
  - match: Host(`app.example.com`) && Header(`X-Cluster`, `secondary`)
    kind: Rule
    priority: 100
    services:
    - name: app-secondary
      namespace: production
      port: 8080

  # Route to DR cluster via subdomain
  - match: Host(`dr.app.example.com`)
    kind: Rule
    services:
    - name: app-dr
      namespace: production
      port: 8080

  tls:
    certResolver: letsencrypt

---
# Cross-cluster service (using external name)
apiVersion: v1
kind: Service
metadata:
  name: app-secondary
  namespace: production
spec:
  type: ExternalName
  externalName: app.cluster2.example.com
  ports:
  - port: 8080
```

### Advanced Health Checks

```yaml
# File provider: advanced health check configuration
# dynamic.yml
http:
  services:
    api-service-with-health:
      loadBalancer:
        servers:
          - url: "http://api1.example.com:8080"
          - url: "http://api2.example.com:8080"
          - url: "http://api3.example.com:8080"

        healthCheck:
          path: "/health"
          interval: "10s"
          timeout: "3s"
          scheme: "http"
          hostname: "api.example.com"
          port: 8080
          followRedirects: true
          headers:
            X-Health-Check: "true"

        # Sticky sessions
        sticky:
          cookie:
            name: "traefik_backend"
            secure: true
            httpOnly: true
            sameSite: "lax"

        # Server weights
        servers:
          - url: "http://api1.example.com:8080"
            weight: 3
          - url: "http://api2.example.com:8080"
            weight: 2
          - url: "http://api3.example.com:8080"
            weight: 1
```

### Circuit Breaker Patterns

```yaml
# Network error circuit breaker
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: circuit-breaker-network
spec:
  circuitBreaker:
    expression: "NetworkErrorRatio() > 0.30"
    checkPeriod: "10s"
    fallbackDuration: "30s"
    recoveryDuration: "10s"

---
# Response code circuit breaker
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: circuit-breaker-5xx
spec:
  circuitBreaker:
    expression: "ResponseCodeRatio(500, 600, 0, 600) > 0.25"
    checkPeriod: "10s"
    fallbackDuration: "60s"

---
# Combined circuit breaker
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: circuit-breaker-combined
spec:
  circuitBreaker:
    expression: "NetworkErrorRatio() > 0.20 || ResponseCodeRatio(500, 600, 0, 600) > 0.20"
    checkPeriod: "15s"
    fallbackDuration: "120s"
```

### Content-Based Routing

```yaml
# Route based on content type
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: content-based-routing
spec:
  entryPoints:
    - websecure
  routes:
  # JSON API requests
  - match: Host(`api.example.com`) && HeadersRegexp(`Content-Type`, `application/json`)
    kind: Rule
    priority: 100
    services:
    - name: json-api-service
      port: 8080

  # GraphQL requests
  - match: Host(`api.example.com`) && PathPrefix(`/graphql`)
    kind: Rule
    priority: 90
    services:
    - name: graphql-service
      port: 4000

  # gRPC requests
  - match: Host(`api.example.com`) && HeadersRegexp(`Content-Type`, `application/grpc`)
    kind: Rule
    priority: 85
    services:
    - name: grpc-service
      port: 50051
      scheme: h2c

  # XML/SOAP requests
  - match: Host(`api.example.com`) && HeadersRegexp(`Content-Type`, `text/xml`)
    kind: Rule
    priority: 80
    services:
    - name: soap-service
      port: 8080

  # Default REST API
  - match: Host(`api.example.com`)
    kind: Rule
    priority: 50
    services:
    - name: rest-api-service
      port: 8080

  tls:
    certResolver: letsencrypt
```

### Retry Strategies

```yaml
# Basic retry
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: retry-basic
spec:
  retry:
    attempts: 3
    initialInterval: "100ms"

---
# Aggressive retry for critical services
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: retry-aggressive
spec:
  retry:
    attempts: 5
    initialInterval: "50ms"

---
# Conservative retry for expensive operations
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: retry-conservative
spec:
  retry:
    attempts: 2
    initialInterval: "500ms"
```

### Plugin Configuration

```yaml
# Install plugin (via static config)
# traefik.yml
experimental:
  plugins:
    block-path:
      moduleName: github.com/traefik/plugin-blockpath
      version: v0.2.1

    rewrite-body:
      moduleName: github.com/traefik/plugin-rewritebody
      version: v0.3.1

    geoblock:
      moduleName: github.com/PascalMinder/geoblock
      version: v0.2.5

---
# Use block-path plugin
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: block-private-paths
spec:
  plugin:
    block-path:
      regex:
        - "^\\/\\..*$"        # Block hidden files
        - ".*\\.env$"         # Block .env files
        - ".*\\.git.*$"       # Block git files
        - ".*wp-admin.*$"     # Block WordPress admin

---
# Use rewrite-body plugin
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: rewrite-api-urls
spec:
  plugin:
    rewrite-body:
      rewrites:
        - regex: "http://old-api.example.com"
          replacement: "https://new-api.example.com"
        - regex: "v1/users"
          replacement: "v2/users"

---
# Use geoblock plugin
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: geoblock-countries
spec:
  plugin:
    geoblock:
      allowedCountries:
        - US
        - CA
        - GB
        - DE
      api: "https://get.geojs.io/v1/ip/country/{ip}"
      cacheSize: 25
      forceMonthlyUpdate: true
      allowLocalRequests: true
      logLocalRequests: false
```

### Monitoring and Observability

```yaml
# Prometheus metrics with custom labels
# traefik.yml
metrics:
  prometheus:
    buckets:
      - 0.005
      - 0.01
      - 0.025
      - 0.05
      - 0.1
      - 0.25
      - 0.5
      - 1.0
      - 2.5
      - 5.0
      - 10.0
    addEntryPointsLabels: true
    addRoutersLabels: true
    addServicesLabels: true
    entryPoint: metrics
    manualRouting: true

---
# Access log with custom format
accessLog:
  filePath: "/var/log/traefik/access.log"
  format: json
  bufferingSize: 100
  filters:
    statusCodes:
      - "200"
      - "300-302"
      - "400-499"
      - "500-599"
    retryAttempts: true
    minDuration: "10ms"
  fields:
    defaultMode: keep
    names:
      ClientUsername: drop
    headers:
      defaultMode: drop
      names:
        User-Agent: keep
        Authorization: redact
        Content-Type: keep
        X-Forwarded-For: keep
        X-Real-IP: keep
        X-Request-ID: keep

---
# Jaeger tracing configuration
tracing:
  jaeger:
    samplingServerURL: "http://jaeger-agent.monitoring:5778/sampling"
    samplingType: "const"
    samplingParam: 1.0
    localAgentHostPort: "jaeger-agent.monitoring:6831"
    gen128Bit: true
    propagation: "jaeger"
    traceContextHeaderName: "uber-trace-id"
    collector:
      endpoint: "http://jaeger-collector.monitoring:14268/api/traces?format=jaeger.thrift"
      user: "traefik"
      password: "secret"

---
# ServiceMonitor for Prometheus Operator
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: traefik
  namespace: traefik
  labels:
    app: traefik
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: traefik
  endpoints:
  - port: metrics
    interval: 15s
    path: /metrics
    scheme: http

---
# PrometheusRule for alerts
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: traefik-alerts
  namespace: traefik
spec:
  groups:
  - name: traefik
    interval: 30s
    rules:
    - alert: TraefikHighErrorRate
      expr: |
        sum(rate(traefik_service_requests_total{code=~"5.."}[5m])) by (service)
        /
        sum(rate(traefik_service_requests_total[5m])) by (service)
        > 0.05
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Traefik high error rate on {{ $labels.service }}"
        description: "Service {{ $labels.service }} has error rate above 5% (current: {{ $value | humanizePercentage }})"

    - alert: TraefikServiceDown
      expr: |
        up{job="traefik"} == 0
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: "Traefik instance is down"
        description: "Traefik instance {{ $labels.instance }} has been down for more than 2 minutes"

    - alert: TraefikHighResponseTime
      expr: |
        histogram_quantile(0.99,
          sum(rate(traefik_service_request_duration_seconds_bucket[5m])) by (service, le)
        ) > 1
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Traefik high response time on {{ $labels.service }}"
        description: "Service {{ $labels.service }} 99th percentile response time is above 1s"
```

### Docker Labels Configuration

```yaml
# Docker Compose with Traefik labels
version: '3.8'

services:
  # Traefik itself
  traefik:
    image: traefik:v3.0
    command:
      - "--api.dashboard=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.letsencrypt.acme.email=admin@example.com"
      - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "./letsencrypt:/letsencrypt"
    labels:
      # Dashboard
      - "traefik.enable=true"
      - "traefik.http.routers.dashboard.rule=Host(`traefik.example.com`)"
      - "traefik.http.routers.dashboard.service=api@internal"
      - "traefik.http.routers.dashboard.entrypoints=websecure"
      - "traefik.http.routers.dashboard.tls.certresolver=letsencrypt"
      - "traefik.http.routers.dashboard.middlewares=auth"
      - "traefik.http.middlewares.auth.basicauth.users=admin:$$apr1$$H6uskkkW$$IgXLP6ewTrSuBkTrqE8wj/"

  # Application with multiple routes
  webapp:
    image: myapp:latest
    labels:
      - "traefik.enable=true"

      # Main website
      - "traefik.http.routers.webapp.rule=Host(`example.com`)"
      - "traefik.http.routers.webapp.entrypoints=websecure"
      - "traefik.http.routers.webapp.tls.certresolver=letsencrypt"
      - "traefik.http.routers.webapp.service=webapp-svc"
      - "traefik.http.services.webapp-svc.loadbalancer.server.port=80"

      # API routes
      - "traefik.http.routers.api.rule=Host(`example.com`) && PathPrefix(`/api`)"
      - "traefik.http.routers.api.entrypoints=websecure"
      - "traefik.http.routers.api.tls.certresolver=letsencrypt"
      - "traefik.http.routers.api.service=api-svc"
      - "traefik.http.routers.api.middlewares=api-ratelimit,strip-api"
      - "traefik.http.services.api-svc.loadbalancer.server.port=8080"

      # Middlewares
      - "traefik.http.middlewares.api-ratelimit.ratelimit.average=100"
      - "traefik.http.middlewares.api-ratelimit.ratelimit.burst=50"
      - "traefik.http.middlewares.strip-api.stripprefix.prefixes=/api"

      # HTTP to HTTPS redirect
      - "traefik.http.routers.webapp-http.rule=Host(`example.com`)"
      - "traefik.http.routers.webapp-http.entrypoints=web"
      - "traefik.http.routers.webapp-http.middlewares=redirect-https"
      - "traefik.http.middlewares.redirect-https.redirectscheme.scheme=https"
      - "traefik.http.middlewares.redirect-https.redirectscheme.permanent=true"

  # Database with TCP routing
  postgres:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secretpass
    labels:
      - "traefik.enable=true"
      - "traefik.tcp.routers.postgres.rule=HostSNI(`*`)"
      - "traefik.tcp.routers.postgres.entrypoints=postgres"
      - "traefik.tcp.routers.postgres.service=postgres-svc"
      - "traefik.tcp.services.postgres-svc.loadbalancer.server.port=5432"
```

## Best Practices

### High Availability

```yaml
# Multiple replicas
deployment:
  replicas: 3

# Pod Disruption Budget
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: traefik-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app.kubernetes.io/name: traefik
```

### Security

```yaml
1. Use TLS everywhere:
   - Redirect HTTP to HTTPS
   - Use TLS 1.2+ only
   - Strong cipher suites

2. Enable security headers:
   - HSTS
   - X-Frame-Options
   - CSP headers

3. Rate limiting:
   - Protect against DDoS
   - Per-IP rate limits

4. IP whitelisting:
   - Restrict admin endpoints
   - Block known bad IPs

5. Keep Traefik updated:
   - Regular security patches
   - Monitor CVEs
```

### Performance

```yaml
1. Enable compression:
   - Reduce bandwidth
   - Faster response times

2. Use caching:
   - HTTP caching headers
   - CDN integration

3. Connection pooling:
   - Reuse backend connections
   - Reduce latency

4. Health checks:
   - Remove unhealthy backends
   - Automatic recovery

5. Optimize middleware order:
   - Most restrictive first
   - Caching before auth
```

### Monitoring

```yaml
1. Collect metrics:
   - Request rate
   - Error rate
   - Response time
   - Backend health

2. Set up alerts:
   - High error rate
   - Slow responses
   - Certificate expiration
   - Backend failures

3. Use distributed tracing:
   - Track requests across services
   - Identify bottlenecks

4. Log analysis:
   - Access patterns
   - Error trends
   - Security events
```

## Additional Resources

- [Official Traefik Documentation](https://doc.traefik.io/traefik/)
- [Traefik GitHub](https://github.com/traefik/traefik)
- [Traefik Community Forum](https://community.traefik.io/)
- [Traefik Pilot](https://pilot.traefik.io/) - SaaS Control Plane

---

*Last updated: 2025-11-16*
