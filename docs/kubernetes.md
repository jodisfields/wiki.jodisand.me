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
