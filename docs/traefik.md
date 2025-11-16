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
