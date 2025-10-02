## 1. **What Are Labels?**

- **Definition**: Labels are **key/value pairs** attached to Kubernetes objects (Pods, Nodes, Services, etc.).
    
- **Purpose**: They don’t affect object behavior directly. Instead, they’re used for **identification, grouping, and selection**.
    

**Example**:

```yaml
metadata:
  labels:
    app: nginx
    env: dev
    tier: frontend
```

## 2. **How to Add Labels**

### (a) While Creating Objects

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
    env: dev
spec:
  containers:
  - name: nginx
    image: nginx
```

### (b) Add/Change Labels Later

```bash
kubectl label pod nginx version=1.0
kubectl label pod nginx env=prod --overwrite
```

### (c) View Labels

```bash
kubectl get pods --show-labels
kubectl get pods -l app=nginx
```

## 3. **What Are Selectors?**

Selectors let you **filter resources** based on labels.

- **Equality-based**:
```bash
kubectl get pods -l env=dev
kubectl get pods -l 'env!=prod'
```

- **Set-based**:
```bash
kubectl get pods -l 'tier in (frontend, backend)'
kubectl get pods -l 'env notin (qa)'
```


## 4. **Labels + Selectors in Action**

### Example: **Service selecting Pods**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx   # Matches Pods with label app=nginx
  ports:
  - port: 80
    targetPort: 80
```

### Example: **Deployment managing Pods**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        env: dev
    spec:
      containers:
      - name: nginx
        image: nginx
```
