# Cilium - eBPF-based Networking and Security

A comprehensive guide to Cilium - a modern CNI plugin leveraging eBPF for networking, observability, and security in Kubernetes.

## Table of Contents

- [Introduction](#introduction)
- [Installation](#installation)
- [Architecture](#architecture)
- [Networking](#networking)
- [Security Policies](#security-policies)
- [Service Mesh](#service-mesh)
- [Observability with Hubble](#observability-with-hubble)
- [Load Balancing](#load-balancing)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

## Introduction

### What is Cilium?

Cilium is an open-source project providing eBPF-based networking, security, and observability for cloud-native environments. It operates at the Linux kernel level to provide high-performance networking and advanced security features.

### Key Features

- **eBPF-based**: Runs in the Linux kernel for maximum performance
- **Identity-based Security**: Security policies based on service identity, not IP addresses
- **API-aware Network Security**: L7 protocol visibility (HTTP, gRPC, Kafka, etc.)
- **Multi-cluster Connectivity**: Connect multiple Kubernetes clusters
- **Service Mesh**: Sidecar-less service mesh capabilities
- **Network Observability**: Real-time network and security monitoring with Hubble

### Why Cilium?

```yaml
Traditional CNI:
  - IP-based policies (brittle in dynamic environments)
  - Limited protocol awareness
  - Performance overhead
  - No built-in observability

Cilium Advantages:
  - Identity-based policies (survive pod rescheduling)
  - Deep protocol visibility (L3-L7)
  - Kernel-level performance
  - Built-in observability with Hubble
  - Advanced load balancing
  - Service mesh without sidecars
```

## Installation

### Prerequisites

```bash
# Kernel requirements
# Linux kernel >= 4.9.17 (5.10+ recommended for all features)

# Check kernel version
uname -r

# Verify eBPF support
mount | grep bpf
# Should show: bpffs on /sys/fs/bpf type bpf

# Check for required kernel configs
grep -E 'CONFIG_BPF|CONFIG_BPF_SYSCALL' /boot/config-$(uname -r)
```

### Install Cilium CLI

```bash
# Download Cilium CLI
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi

curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}

sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}

# Verify installation
cilium version --client
```

### Install Cilium on Kubernetes

```bash
# Basic installation
cilium install --version 1.14.5

# Install with specific datapath mode
cilium install --version 1.14.5 --datapath-mode=native

# Install with encryption
cilium install --version 1.14.5 --encryption=wireguard

# Install with Hubble enabled
cilium install --version 1.14.5 \
  --set hubble.relay.enabled=true \
  --set hubble.ui.enabled=true

# Verify installation
cilium status --wait

# Check Cilium pods
kubectl -n kube-system get pods -l k8s-app=cilium
```

### Helm Installation (Advanced)

```bash
# Add Cilium Helm repo
helm repo add cilium https://helm.cilium.io/
helm repo update

# Install with custom values
helm install cilium cilium/cilium \
  --version 1.14.5 \
  --namespace kube-system \
  --set operator.replicas=1 \
  --set ipam.mode=kubernetes \
  --set kubeProxyReplacement=strict \
  --set hubble.relay.enabled=true \
  --set hubble.ui.enabled=true

# Upgrade Cilium
helm upgrade cilium cilium/cilium \
  --version 1.14.6 \
  --namespace kube-system \
  --reuse-values
```

### Installation with values.yaml

```yaml
# cilium-values.yaml
# IPv4/IPv6
ipv4:
  enabled: true
ipv6:
  enabled: false

# IPAM mode (kubernetes, cluster-pool, multi-pool)
ipam:
  mode: kubernetes

# Replace kube-proxy
kubeProxyReplacement: "strict"

# Enable Hubble
hubble:
  enabled: true
  relay:
    enabled: true
  ui:
    enabled: true

# Enable bandwidth manager
bandwidthManager:
  enabled: true

# Enable local redirect policy
localRedirectPolicy: true

# Encryption (wireguard, ipsec)
encryption:
  enabled: true
  type: wireguard

# L7 proxy
proxy:
  prometheus:
    enabled: true

# BGP Control Plane
bgpControlPlane:
  enabled: false

# Cluster mesh
clustermesh:
  useAPIServer: false
```

```bash
helm install cilium cilium/cilium \
  --namespace kube-system \
  --values cilium-values.yaml
```

## Architecture

### Components

```
┌─────────────────────────────────────────────┐
│           Kubernetes Cluster                │
│                                             │
│  ┌──────────────┐      ┌──────────────┐    │
│  │   Node 1     │      │   Node 2     │    │
│  │              │      │              │    │
│  │ ┌──────────┐ │      │ ┌──────────┐ │    │
│  │ │  Pod A   │ │      │ │  Pod B   │ │    │
│  │ └──────────┘ │      │ └──────────┘ │    │
│  │      ↕       │      │      ↕       │    │
│  │ ┌──────────┐ │      │ ┌──────────┐ │    │
│  │ │ Cilium   │←┼──────┼→│ Cilium   │ │    │
│  │ │ Agent    │ │      │ │ Agent    │ │    │
│  │ └──────────┘ │      │ └──────────┘ │    │
│  └──────────────┘      └──────────────┘    │
│         ↕                      ↕            │
│  ┌──────────────────────────────────────┐  │
│  │    Cilium Operator (manages CRDs)    │  │
│  └──────────────────────────────────────┘  │
│                    ↕                        │
│  ┌──────────────────────────────────────┐  │
│  │    Hubble Relay (observability)      │  │
│  └──────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

### Cilium Agent

The Cilium agent runs on each node as a DaemonSet and manages:
- eBPF program compilation and loading
- Network connectivity for pods
- Policy enforcement
- Load balancing
- Service discovery

### Cilium Operator

Manages cluster-wide resources:
- CRD lifecycle
- IP address management (IPAM)
- Node and endpoint cleanup
- Identity management

## Networking

### Datapath Modes

```bash
# Tunneling mode (VXLAN/Geneve) - Default
# Works on any network, overlay network
cilium install --datapath-mode=tunnel

# Native routing mode
# Direct routing, requires L3 network connectivity
cilium install --datapath-mode=native --ipv4-native-routing-cidr=10.0.0.0/8

# AWS ENI mode
# Uses AWS Elastic Network Interfaces
cilium install --datapath-mode=aws-eni --enable-endpoint-routes=true
```

### IP Address Management (IPAM)

```yaml
# Kubernetes host-scope IPAM
ipam:
  mode: kubernetes

# Cluster-scope IPAM (Cilium manages pool)
ipam:
  mode: cluster-pool
  operator:
    clusterPoolIPv4PodCIDRList: "10.0.0.0/8"
    clusterPoolIPv4MaskSize: 24

# Multi-pool IPAM
ipam:
  mode: multi-pool
```

### Kube-proxy Replacement

```bash
# Enable kube-proxy replacement
cilium install --set kubeProxyReplacement=strict

# Verify kube-proxy is replaced
cilium status | grep KubeProxyReplacement

# Check eBPF maps
kubectl -n kube-system exec ds/cilium -- cilium service list
```

### Direct Server Return (DSR)

```yaml
# Enable DSR for improved performance
loadBalancer:
  mode: dsr

# Maglev consistent hashing (for DSR)
loadBalancer:
  algorithm: maglev
```

## Security Policies

### Identity-based Security

Cilium uses identity, not IP addresses, for policy enforcement:

```bash
# View identities
kubectl -n kube-system exec ds/cilium -- cilium identity list

# Example output:
# ID      Labels
# 1       reserved:host
# 2       reserved:world
# 4       reserved:health
# 12345   k8s:app=frontend
# 12346   k8s:app=backend
```

### Kubernetes Network Policies

Cilium fully supports standard Kubernetes NetworkPolicies:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

### Cilium Network Policies (L3-L7)

```yaml
# L3/L4 Policy
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: backend-policy
  namespace: default
spec:
  endpointSelector:
    matchLabels:
      app: backend
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: frontend
    toPorts:
    - ports:
      - port: "8080"
        protocol: TCP

---
# L7 HTTP Policy
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: api-policy
spec:
  endpointSelector:
    matchLabels:
      app: api-server
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: web
    toPorts:
    - ports:
      - port: "80"
        protocol: TCP
      rules:
        http:
        - method: "GET"
          path: "/api/v1/.*"
        - method: "POST"
          path: "/api/v1/users"

---
# Egress Policy (DNS-aware)
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: external-access
spec:
  endpointSelector:
    matchLabels:
      app: myapp
  egress:
  - toFQDNs:
    - matchName: "api.external.com"
  - toPorts:
    - ports:
      - port: "443"
        protocol: TCP

---
# Kafka Policy
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: kafka-policy
spec:
  endpointSelector:
    matchLabels:
      app: kafka-consumer
  egress:
  - toEndpoints:
    - matchLabels:
        app: kafka
    toPorts:
    - ports:
      - port: "9092"
        protocol: TCP
      rules:
        kafka:
        - role: "consume"
          topic: "events"
```

### Cluster-wide Policies

```yaml
# Apply policy to entire cluster
apiVersion: cilium.io/v2
kind: CiliumClusterwideNetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  endpointSelector: {}
  ingress:
  - fromEntities:
    - cluster
```

### Host Policies

```yaml
# Policy for node/host
apiVersion: cilium.io/v2
kind: CiliumClusterwideNetworkPolicy
metadata:
  name: host-fw-control-plane
spec:
  nodeSelector:
    matchLabels:
      node-role.kubernetes.io/control-plane: ""
  ingress:
  - fromEntities:
    - cluster
    toPorts:
    - ports:
      - port: "6443"
        protocol: TCP
```

## Service Mesh

### Enable Service Mesh

```bash
# Install with service mesh
cilium install \
  --set kubeProxyReplacement=strict \
  --set loadBalancer.l7.backend=envoy \
  --set tls.secretsBackend=k8s
```

### Ingress Controller

```yaml
# Cilium Ingress example
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    ingress.cilium.io/loadbalancer-mode: shared
spec:
  ingressClassName: cilium
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

### Gateway API

```yaml
# Gateway API support
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: my-gateway
spec:
  gatewayClassName: cilium
  listeners:
  - name: http
    protocol: HTTP
    port: 80
  - name: https
    protocol: HTTPS
    port: 443
    tls:
      certificateRefs:
      - name: my-cert

---
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: my-route
spec:
  parentRefs:
  - name: my-gateway
  hostnames:
  - "myapp.example.com"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: myapp
      port: 80
```

## Observability with Hubble

### Install Hubble CLI

```bash
# Install Hubble CLI
export HUBBLE_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/hubble/master/stable.txt)
HUBBLE_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then HUBBLE_ARCH=arm64; fi

curl -L --fail --remote-name-all https://github.com/cilium/hubble/releases/download/$HUBBLE_VERSION/hubble-linux-${HUBBLE_ARCH}.tar.gz{,.sha256sum}
sha256sum --check hubble-linux-${HUBBLE_ARCH}.tar.gz.sha256sum
sudo tar xzvfC hubble-linux-${HUBBLE_ARCH}.tar.gz /usr/local/bin
rm hubble-linux-${HUBBLE_ARCH}.tar.gz{,.sha256sum}
```

### Enable Hubble

```bash
# Enable Hubble
cilium hubble enable --ui

# Port-forward Hubble Relay
cilium hubble port-forward &

# Verify Hubble status
hubble status
```

### Hubble Observability Commands

```bash
# Observe all flows
hubble observe

# Follow flows in real-time
hubble observe --follow

# Filter by namespace
hubble observe --namespace default

# Filter by pod
hubble observe --pod myapp

# Filter by label
hubble observe --from-label app=frontend

# Filter by verdict
hubble observe --verdict DROPPED
hubble observe --verdict FORWARDED
hubble observe --verdict AUDIT

# Filter by protocol
hubble observe --protocol TCP
hubble observe --protocol HTTP
hubble observe --protocol DNS

# HTTP-specific filtering
hubble observe --http-status 404
hubble observe --http-method GET
hubble observe --http-path "/api/.*"

# DNS queries
hubble observe --type trace:to-endpoint --protocol dns

# Dropped packets
hubble observe --verdict DROPPED --type drop

# Security events
hubble observe --type policy-verdict

# Service dependencies
hubble observe --from-pod frontend --to-pod backend
```

### Hubble UI

```bash
# Access Hubble UI
kubectl port-forward -n kube-system svc/hubble-ui 12000:80

# Open browser
# http://localhost:12000
```

### Hubble Metrics

```bash
# Enable Hubble metrics
helm upgrade cilium cilium/cilium \
  --namespace kube-system \
  --reuse-values \
  --set hubble.metrics.enabled="{dns,drop,tcp,flow,icmp,http}"

# Scrape metrics with Prometheus
# Metrics available at: http://<cilium-agent>:9091/metrics
```

## Load Balancing

### Maglev Consistent Hashing

```yaml
# Enable Maglev for consistent hashing
loadBalancer:
  algorithm: maglev
  mode: dsr  # Direct Server Return
```

### XDP Acceleration

```yaml
# Enable XDP for extreme performance
loadBalancer:
  acceleration: native  # or best-effort
```

### Load Balancer IP Pool

```yaml
# Define LoadBalancer IP pools
apiVersion: cilium.io/v2alpha1
kind:CiliumLoadBalancerIPPool
metadata:
  name: blue-pool
spec:
  cidrs:
  - cidr: "192.168.1.0/24"
  serviceSelector:
    matchLabels:
      color: blue
```

## Troubleshooting

### Connectivity Test

```bash
# Run connectivity test
cilium connectivity test

# Test specific scenario
cilium connectivity test --test pod-to-pod
cilium connectivity test --test pod-to-service
```

### Debug Commands

```bash
# Cilium status
cilium status

# Check endpoints
kubectl -n kube-system exec ds/cilium -- cilium endpoint list

# Check policy
kubectl -n kube-system exec ds/cilium -- cilium policy get

# Monitor events
kubectl -n kube-system exec ds/cilium -- cilium monitor

# BPF maps
kubectl -n kube-system exec ds/cilium -- cilium bpf lb list
kubectl -n kube-system exec ds/cilium -- cilium bpf ct list global

# Debug a specific pod
kubectl -n kube-system exec ds/cilium -- cilium endpoint get <pod-ip>
```

### Log Collection

```bash
# Collect logs
cilium sysdump

# Get logs from specific component
kubectl -n kube-system logs ds/cilium
kubectl -n kube-system logs deployment/cilium-operator

# Increase log verbosity
kubectl -n kube-system exec ds/cilium -- cilium config set debug true
```

### Common Issues

```bash
# Issue: Pod not getting IP
# Check IPAM
kubectl -n kube-system exec ds/cilium -- cilium status --verbose | grep -A 10 IPAM

# Issue: Policy not working
# Check policy enforcement
kubectl -n kube-system exec ds/cilium -- cilium policy get
# Check identity
kubectl -n kube-system exec ds/cilium -- cilium identity list

# Issue: Service not reachable
# Check service list
kubectl -n kube-system exec ds/cilium -- cilium service list
# Check load balancer maps
kubectl -n kube-system exec ds/cilium -- cilium bpf lb list
```

## Best Practices

### Performance Optimization

```yaml
1. Enable Native Routing:
   datapath:
     mode: native

2. Use kube-proxy Replacement:
   kubeProxyReplacement: strict

3. Enable XDP Acceleration:
   loadBalancer:
     acceleration: native

4. Use Maglev for Load Balancing:
   loadBalancer:
     algorithm: maglev

5. Enable Bandwidth Manager:
   bandwidthManager:
     enabled: true
```

### Security Hardening

```yaml
1. Enable Encryption:
   encryption:
     enabled: true
     type: wireguard

2. Enable Host Firewall:
   hostFirewall:
     enabled: true

3. Use L7 Policies:
   # Implement application-aware policies

4. Enable Policy Audit Mode First:
   # Test policies before enforcement
   policyAuditMode: true

5. Restrict External Access:
   # Use FQDN-based egress policies
```

### Monitoring & Observability

```yaml
1. Enable Hubble Metrics:
   hubble:
     metrics:
       enabled: "{dns,drop,tcp,flow,http}"

2. Integrate with Prometheus:
   prometheus:
     enabled: true
     serviceMonitor:
       enabled: true

3. Use Hubble UI:
   hubble:
     ui:
       enabled: true

4. Enable Flow Logs:
   # Monitor network flows regularly

5. Set Up Alerts:
   # Alert on policy violations, drops, etc.
```

### Upgrade Strategy

```bash
# 1. Check release notes
# Read: https://github.com/cilium/cilium/releases

# 2. Test in non-production first
# 3. Backup current configuration
kubectl get -n kube-system cm cilium-config -o yaml > cilium-config-backup.yaml

# 4. Perform upgrade
helm upgrade cilium cilium/cilium \
  --version <new-version> \
  --namespace kube-system \
  --reuse-values

# 5. Verify health
cilium status
cilium connectivity test

# 6. Monitor for issues
kubectl -n kube-system logs -f ds/cilium
```

## Additional Resources

- [Official Cilium Documentation](https://docs.cilium.io/)
- [Cilium GitHub](https://github.com/cilium/cilium)
- [eBPF Documentation](https://ebpf.io/)
- [Cilium Slack Community](https://cilium.herokuapp.com/)
- [Cilium Labs](https://cilium.io/labs/) - Interactive tutorials

---

*Last updated: 2025-11-16*
