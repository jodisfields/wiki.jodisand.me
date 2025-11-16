# Kubernetes Cheatsheet

A comprehensive guide to Kubernetes (K8s) - the industry-standard container orchestration platform.

## Table of Contents

- [Core Concepts](#core-concepts)
- [kubectl Commands](#kubectl-commands)
- [Workloads](#workloads)
- [Services & Networking](#services--networking)
- [Storage](#storage)
- [Configuration](#configuration)
- [Security](#security)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)
- [kubectl Plugins](#kubectl-plugins)
- [k9s - Kubernetes CLI](#k9s---kubernetes-cli)
- [CNI (Container Network Interface)](#cni-container-network-interface)

## Core Concepts

### Kubernetes Architecture

- **Control Plane**: Manages the cluster (API Server, Scheduler, Controller Manager, etcd)
- **Worker Nodes**: Run containerized applications
- **Pods**: Smallest deployable units containing one or more containers
- **Services**: Stable network endpoints for accessing pods
- **Namespaces**: Virtual clusters for resource isolation

### Object Types

```yaml
# Common Kubernetes objects
- Pod: Single instance of a running process
- Deployment: Manages ReplicaSets and Pods
- StatefulSet: Manages stateful applications
- DaemonSet: Runs a pod on every node
- Job: Creates one or more pods for batch processing
- CronJob: Scheduled jobs
- Service: Network access to pods
- Ingress: HTTP/HTTPS routing
- ConfigMap: Configuration data
- Secret: Sensitive data
- PersistentVolume: Storage resources
- PersistentVolumeClaim: Storage requests
```

## kubectl Commands

### Cluster Information

```bash
# Display cluster info
kubectl cluster-info

# View cluster nodes
kubectl get nodes
kubectl get nodes -o wide

# Describe a node
kubectl describe node <node-name>

# Check cluster version
kubectl version

# View cluster components status
kubectl get componentstatuses
kubectl get cs
```

### Context and Configuration

```bash
# View current context
kubectl config current-context

# List all contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <context-name>

# Set default namespace
kubectl config set-context --current --namespace=<namespace>

# View kubeconfig
kubectl config view
```

### Namespace Operations

```bash
# List all namespaces
kubectl get namespaces
kubectl get ns

# Create namespace
kubectl create namespace <namespace-name>

# Delete namespace
kubectl delete namespace <namespace-name>

# Set default namespace for current context
kubectl config set-context --current --namespace=<namespace>
```

## Workloads

### Pods

```bash
# List pods
kubectl get pods
kubectl get pods -n <namespace>
kubectl get pods --all-namespaces
kubectl get pods -o wide

# Create pod from YAML
kubectl apply -f pod.yaml

# Get pod details
kubectl describe pod <pod-name>

# View pod logs
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>
kubectl logs <pod-name> --previous
kubectl logs -f <pod-name>  # Follow logs

# Execute command in pod
kubectl exec <pod-name> -- <command>
kubectl exec -it <pod-name> -- /bin/bash

# Port forwarding
kubectl port-forward <pod-name> <local-port>:<pod-port>

# Delete pod
kubectl delete pod <pod-name>
kubectl delete pod <pod-name> --grace-period=0 --force

# Copy files to/from pod
kubectl cp <pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <pod-name>:/path/to/file
```

### Deployments

```bash
# List deployments
kubectl get deployments
kubectl get deploy

# Create deployment
kubectl create deployment <name> --image=<image>
kubectl apply -f deployment.yaml

# Scale deployment
kubectl scale deployment <name> --replicas=<count>

# Update deployment image
kubectl set image deployment/<name> <container>=<new-image>

# Rollout status
kubectl rollout status deployment/<name>

# Rollout history
kubectl rollout history deployment/<name>

# Rollback deployment
kubectl rollout undo deployment/<name>
kubectl rollout undo deployment/<name> --to-revision=<number>

# Restart deployment
kubectl rollout restart deployment/<name>

# Delete deployment
kubectl delete deployment <name>
```

### Example Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

### StatefulSets

```bash
# List statefulsets
kubectl get statefulsets
kubectl get sts

# Create statefulset
kubectl apply -f statefulset.yaml

# Scale statefulset
kubectl scale statefulset <name> --replicas=<count>

# Delete statefulset (keep pods)
kubectl delete statefulset <name> --cascade=orphan
```

### DaemonSets

```bash
# List daemonsets
kubectl get daemonsets
kubectl get ds

# Create daemonset
kubectl apply -f daemonset.yaml

# Delete daemonset
kubectl delete daemonset <name>
```

### Jobs and CronJobs

```bash
# List jobs
kubectl get jobs

# Create job
kubectl create job <name> --image=<image>
kubectl apply -f job.yaml

# List cronjobs
kubectl get cronjobs
kubectl get cj

# Create cronjob
kubectl create cronjob <name> --image=<image> --schedule="*/5 * * * *"

# Trigger cronjob manually
kubectl create job --from=cronjob/<cronjob-name> <job-name>
```

## Services & Networking

### Services

```bash
# List services
kubectl get services
kubectl get svc

# Create service
kubectl expose deployment <name> --port=<port> --target-port=<target-port>
kubectl apply -f service.yaml

# Get service details
kubectl describe service <name>

# Delete service
kubectl delete service <name>
```

### Service Types

```yaml
# ClusterIP (default) - Internal cluster access only
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080

---
# NodePort - Exposes service on each node's IP at a static port
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080  # 30000-32767

---
# LoadBalancer - Creates external load balancer (cloud provider)
apiVersion: v1
kind: Service
metadata:
  name: my-lb-service
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```

### Ingress

```bash
# List ingress resources
kubectl get ingress
kubectl get ing

# Create ingress
kubectl apply -f ingress.yaml

# Get ingress details
kubectl describe ingress <name>
```

### Example Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: example.com
    http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
  tls:
  - hosts:
    - example.com
    secretName: tls-secret
```

### Network Policies

```bash
# List network policies
kubectl get networkpolicies
kubectl get netpol

# Create network policy
kubectl apply -f networkpolicy.yaml
```

## Storage

### PersistentVolumes (PV) and PersistentVolumeClaims (PVC)

```bash
# List persistent volumes
kubectl get pv

# List persistent volume claims
kubectl get pvc

# Create PVC
kubectl apply -f pvc.yaml

# Delete PVC
kubectl delete pvc <name>
```

### Example PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
```

### StorageClasses

```bash
# List storage classes
kubectl get storageclass
kubectl get sc

# Describe storage class
kubectl describe sc <name>
```

## Configuration

### ConfigMaps

```bash
# List configmaps
kubectl get configmaps
kubectl get cm

# Create configmap from literal values
kubectl create configmap <name> --from-literal=key1=value1 --from-literal=key2=value2

# Create configmap from file
kubectl create configmap <name> --from-file=<path-to-file>

# Create configmap from directory
kubectl create configmap <name> --from-file=<directory>

# Get configmap
kubectl get configmap <name> -o yaml

# Delete configmap
kubectl delete configmap <name>
```

### Example ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgres://db:5432/mydb"
  log_level: "info"
  config.json: |
    {
      "feature_flags": {
        "new_ui": true
      }
    }
```

### Secrets

```bash
# List secrets
kubectl get secrets

# Create secret from literal
kubectl create secret generic <name> --from-literal=password=mypassword

# Create secret from file
kubectl create secret generic <name> --from-file=ssh-key=/path/to/key

# Create TLS secret
kubectl create secret tls <name> --cert=cert.pem --key=key.pem

# Create docker registry secret
kubectl create secret docker-registry <name> \
  --docker-server=<registry> \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email>

# Get secret (base64 encoded)
kubectl get secret <name> -o yaml

# Decode secret
kubectl get secret <name> -o jsonpath='{.data.password}' | base64 -d
```

### Using ConfigMaps and Secrets in Pods

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configured-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    # Environment variables from ConfigMap
    envFrom:
    - configMapRef:
        name: app-config
    env:
    # Single env var from ConfigMap
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    # Environment variable from Secret
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    # Mount ConfigMap as volume
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
    # Mount Secret as volume
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: config-volume
    configMap:
      name: app-config
  - name: secret-volume
    secret:
      secretName: db-secret
```

## Security

### ServiceAccounts

```bash
# List service accounts
kubectl get serviceaccounts
kubectl get sa

# Create service account
kubectl create serviceaccount <name>

# Get service account token
kubectl get serviceaccount <name> -o yaml
```

### RBAC (Role-Based Access Control)

```bash
# List roles
kubectl get roles

# List cluster roles
kubectl get clusterroles

# List role bindings
kubectl get rolebindings

# List cluster role bindings
kubectl get clusterrolebindings

# Create role
kubectl create role <name> --verb=get,list --resource=pods

# Create role binding
kubectl create rolebinding <name> --role=<role> --user=<user>

# Create cluster role binding
kubectl create clusterrolebinding <name> --clusterrole=<role> --user=<user>
```

### Example RBAC

```yaml
# Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### Security Contexts

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: sec-ctx-demo
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    securityContext:
      allowPrivilegeEscalation: false
      runAsNonRoot: true
      capabilities:
        drop:
        - ALL
      readOnlyRootFilesystem: true
```

## Troubleshooting

### Debugging Pods

```bash
# Get pod status
kubectl get pods
kubectl describe pod <pod-name>

# Check pod logs
kubectl logs <pod-name>
kubectl logs <pod-name> --previous
kubectl logs <pod-name> -c <container-name>

# Get pod events
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events --field-selector involvedObject.name=<pod-name>

# Interactive shell in pod
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -c <container-name> -- /bin/sh

# Debug with ephemeral container (K8s 1.23+)
kubectl debug <pod-name> -it --image=busybox

# Create debug pod
kubectl run debug --image=busybox -it --rm -- sh
```

### Resource Usage

```bash
# Node resource usage
kubectl top nodes

# Pod resource usage
kubectl top pods
kubectl top pods -n <namespace>
kubectl top pods --containers

# Resource quotas
kubectl get resourcequota
kubectl describe resourcequota <name>

# Limit ranges
kubectl get limitrange
kubectl describe limitrange <name>
```

### Common Issues and Solutions

```bash
# ImagePullBackOff / ErrImagePull
kubectl describe pod <pod-name>
# Check: Image name, registry credentials, network connectivity

# CrashLoopBackOff
kubectl logs <pod-name> --previous
# Check: Application errors, missing dependencies, configuration issues

# Pending pods
kubectl describe pod <pod-name>
# Check: Resource constraints, node selectors, taints/tolerations

# Service not accessible
kubectl get endpoints <service-name>
# Check: Selector labels match pod labels, port configuration

# DNS issues
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup kubernetes.default
# Check: CoreDNS pods running, service discovery working
```

### Cluster Troubleshooting

```bash
# Check cluster component health
kubectl get componentstatuses

# Check node conditions
kubectl describe node <node-name>

# Check cluster events
kubectl get events --all-namespaces --sort-by=.metadata.creationTimestamp

# API server logs (depends on cluster setup)
kubectl logs -n kube-system <kube-apiserver-pod>

# Check CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

## Best Practices

### Resource Management

```yaml
# Always set resource requests and limits
apiVersion: v1
kind: Pod
metadata:
  name: best-practice-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    resources:
      requests:
        memory: "128Mi"
        cpu: "250m"
      limits:
        memory: "256Mi"
        cpu: "500m"
```

### Health Checks

```yaml
# Implement liveness and readiness probes
apiVersion: v1
kind: Pod
metadata:
  name: healthy-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 3
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      timeoutSeconds: 3
      failureThreshold: 3
```

### Labels and Annotations

```yaml
# Use meaningful labels and annotations
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
    version: "1.0"
    environment: production
    team: platform
  annotations:
    description: "Main application deployment"
    owner: "platform-team@example.com"
spec:
  selector:
    matchLabels:
      app: myapp
      version: "1.0"
  template:
    metadata:
      labels:
        app: myapp
        version: "1.0"
    spec:
      containers:
      - name: myapp
        image: myapp:1.0
```

### Security Best Practices

1. **Use non-root users**: Run containers as non-root users
2. **Read-only root filesystem**: Make root filesystem read-only when possible
3. **Drop capabilities**: Remove unnecessary Linux capabilities
4. **Use network policies**: Restrict network traffic between pods
5. **Scan images**: Regularly scan container images for vulnerabilities
6. **Use RBAC**: Implement proper role-based access control
7. **Rotate secrets**: Regularly rotate credentials and secrets
8. **Enable audit logging**: Track API access and changes

### Deployment Strategies

```yaml
# Rolling Update (default)
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0

---
# Recreate (downtime)
spec:
  strategy:
    type: Recreate
```

### Pod Disruption Budgets

```yaml
# Ensure high availability during maintenance
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp
```

## kubectl Plugins

### Installing kubectl Plugins

```bash
# Install krew (kubectl plugin manager)
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)

# Add to PATH (add to ~/.bashrc or ~/.zshrc)
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"

# Update krew
kubectl krew update

# List available plugins
kubectl krew search

# Install plugin
kubectl krew install <plugin-name>

# List installed plugins
kubectl krew list

# Upgrade plugins
kubectl krew upgrade
```

### Popular kubectl Plugins

```bash
# ctx - Switch between contexts
kubectl krew install ctx
kubectl ctx                    # List contexts
kubectl ctx <context-name>     # Switch context

# ns - Switch between namespaces
kubectl krew install ns
kubectl ns                     # List namespaces
kubectl ns <namespace>         # Switch namespace

# tree - Show hierarchy of resources
kubectl krew install tree
kubectl tree deployment myapp

# neat - Clean up YAML output
kubectl krew install neat
kubectl get pod mypod -o yaml | kubectl neat

# stern - Multi-pod log tailing
kubectl krew install stern
kubectl stern myapp            # Tail logs from all pods matching 'myapp'
kubectl stern --namespace=prod myapp

# access-matrix - Show RBAC permissions
kubectl krew install access-matrix
kubectl access-matrix

# node-shell - Shell into nodes
kubectl krew install node-shell
kubectl node-shell <node-name>

# view-secret - Decode secrets
kubectl krew install view-secret
kubectl view-secret <secret-name>

# images - Show images used in cluster
kubectl krew install images
kubectl images

# whoami - Show current user info
kubectl krew install whoami
kubectl whoami
```

## k9s - Kubernetes CLI

### Installation

```bash
# macOS
brew install k9s

# Linux (binary)
wget https://github.com/derailed/k9s/releases/latest/download/k9s_Linux_amd64.tar.gz
tar -xzf k9s_Linux_amd64.tar.gz
sudo mv k9s /usr/local/bin/

# Using Go
go install github.com/derailed/k9s@latest

# Verify installation
k9s version
```

### Basic Usage

```bash
# Start k9s
k9s

# Start in specific namespace
k9s -n <namespace>

# Start with specific context
k9s --context <context-name>

# Read-only mode
k9s --readonly

# Start at specific resource
k9s -c pods
k9s -c deployments
```

### k9s Navigation

```bash
# General Navigation
:pods         # View pods
:svc          # View services
:deploy       # View deployments
:ns           # View namespaces
:no           # View nodes
:ctx          # Switch context
:quit         # Exit k9s

# Key Bindings
<enter>       # View resource details
<d>           # Describe resource
<l>           # View logs
<e>           # Edit resource
<y>           # View YAML
<shift-f>     # Port forward
<s>           # Shell into container
<ctrl-d>      # Delete resource
<ctrl-k>      # Kill pod
</>           # Filter
<esc>         # Back/Clear
<?> or <h>    # Help

# Pod Operations
<l>           # View logs
<p>           # Previous logs
<f>           # Follow logs
<s>           # Shell into container
<shift-f>     # Port forward
<ctrl-k>      # Kill pod

# Filtering
/nginx        # Filter resources containing 'nginx'
/-l app=web   # Filter by label
/!Running     # Inverse filter (not Running)
```

### k9s Configuration

```yaml
# ~/.config/k9s/config.yml
k9s:
  # Refresh rate (ms)
  refreshRate: 2000

  # Max log lines
  maxConnRetry: 5

  # Enable mouse support
  enableMouse: true

  # Default namespace
  namespace:
    active: default
    favorites:
      - default
      - kube-system
      - production

  # View settings
  view:
    active: pods

  # Log settings
  logger:
    tail: 100
    buffer: 5000

  # Theme
  thresholds:
    cpu:
      critical: 90
      warn: 70
    memory:
      critical: 90
      warn: 70
```

### k9s Skins/Themes

```yaml
# ~/.config/k9s/skin.yml
k9s:
  # Dracula theme example
  body:
    fgColor: "#f8f8f2"
    bgColor: "#282a36"
    logoColor: "#ff79c6"

  prompt:
    fgColor: "#f8f8f2"
    bgColor: "#282a36"
    suggestColor: "#6272a4"

  info:
    fgColor: "#8be9fd"
    sectionColor: "#f8f8f2"

  help:
    fgColor: "#f8f8f2"
    bgColor: "#282a36"
    keyColor: "#50fa7b"

  # Or use built-in skins
  # Options: default, dracula, monokai, nord, solarized-dark, etc.
```

### k9s Plugins

```yaml
# ~/.config/k9s/plugin.yml
plugins:
  # Dive into image layers
  dive:
    shortCut: Shift-D
    description: Dive image
    scopes:
      - containers
    command: dive
    background: false
    args:
      - $COL-IMAGE

  # Debug with ephemeral container
  debug:
    shortCut: Shift-E
    description: Add debug container
    scopes:
      - pods
    command: kubectl
    background: false
    args:
      - debug
      - -it
      - -n
      - $NAMESPACE
      - $NAME
      - --image=nicolaka/netshoot
      - --target=$NAME
```

## CNI (Container Network Interface)

### Overview

CNI plugins handle networking for Kubernetes pods:

- **Flannel**: Simple overlay network
- **Calico**: Network policy and BGP routing
- **Cilium**: eBPF-based networking and security
- **Weave**: Simple, resilient network
- **Canal**: Flannel + Calico policies

### Cilium CNI Integration

```bash
# Install Cilium CLI
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-amd64.tar.gz
sudo tar xzvfC cilium-linux-amd64.tar.gz /usr/local/bin
rm cilium-linux-amd64.tar.gz

# Install Cilium on Kubernetes
cilium install --version 1.14.5

# Check status
cilium status
cilium status --wait

# Enable Hubble (observability)
cilium hubble enable --ui

# Verify installation
cilium connectivity test

# View Cilium pods
kubectl -n kube-system get pods -l k8s-app=cilium
```

### Cilium Features

```bash
# Network Policies
# Cilium supports both Kubernetes NetworkPolicies
# and CiliumNetworkPolicies with advanced features

# Enable IP transparency
cilium config set enable-ipv4=true
cilium config set enable-ipv6=false

# Enable bandwidth manager
cilium config set enable-bandwidth-manager=true

# Enable host firewall
cilium config set enable-host-firewall=true

# View Cilium endpoints
kubectl -n kube-system exec -ti ds/cilium -- cilium endpoint list

# View identity
kubectl -n kube-system exec -ti ds/cilium -- cilium identity list

# Monitor traffic
kubectl -n kube-system exec -ti ds/cilium -- cilium monitor

# Check policy enforcement
kubectl -n kube-system exec -ti ds/cilium -- cilium policy get
```

### Hubble Observability

```bash
# Install Hubble CLI
export HUBBLE_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/hubble/master/stable.txt)
curl -L --fail --remote-name-all https://github.com/cilium/hubble/releases/download/$HUBBLE_VERSION/hubble-linux-amd64.tar.gz
sudo tar xzvfC hubble-linux-amd64.tar.gz /usr/local/bin

# Port-forward Hubble
cilium hubble port-forward &

# Observe flows
hubble observe
hubble observe --namespace default
hubble observe --pod myapp

# Filter by verdict
hubble observe --verdict DROPPED
hubble observe --verdict FORWARDED

# Service dependencies
hubble observe --protocol http

# Access Hubble UI
kubectl port-forward -n kube-system svc/hubble-ui 12000:80
# Open browser to http://localhost:12000
```

### Example Cilium Network Policy

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: allow-frontend-to-backend
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
          rules:
            http:
              - method: "GET"
                path: "/api/.*"
```

For more detailed Cilium documentation, see [Cilium CNI](cilium.md).

## Quick Reference

### Common kubectl Shortcuts

```bash
# Short names
po     = pods
svc    = services
deploy = deployments
rs     = replicasets
ds     = daemonsets
sts    = statefulsets
cm     = configmaps
ns     = namespaces
pv     = persistentvolumes
pvc    = persistentvolumeclaims
ing    = ingress
no     = nodes

# Usage examples
kubectl get po
kubectl get svc
kubectl describe deploy myapp
```

### Useful Aliases

```bash
# Add to ~/.bashrc or ~/.zshrc
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias kgd='kubectl get deployments'
alias kdp='kubectl describe pod'
alias kl='kubectl logs'
alias kex='kubectl exec -it'
alias kaf='kubectl apply -f'
alias kdel='kubectl delete'
```

### Output Formats

```bash
# Get output in different formats
kubectl get pods -o wide
kubectl get pods -o yaml
kubectl get pods -o json
kubectl get pods -o name
kubectl get pods -o jsonpath='{.items[*].metadata.name}'

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName

# Sort by creation time
kubectl get pods --sort-by=.metadata.creationTimestamp
```

## Additional Resources

- [Official Kubernetes Documentation](https://kubernetes.io/docs/)
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Kubernetes Patterns](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/)

---

*Last updated: 2025-11-16*
