

# What are Static Pods?

- **Static Pods** are pods **directly managed by the kubelet** on a node, not by the Kubernetes Control Plane (API server, scheduler, etc.).
    
- They:
    
    - Always run on a specific node.
        
    - Do not use higher-level controllers (no Deployment, ReplicaSet).
        
    - Are defined by placing a **Pod manifest YAML file** in a specific directory on the node.
        
- Kubelet continuously monitors this directory and automatically creates/deletes pods based on the files.
    

Example: If kube-apiserver or controller-manager crashes, the kubelet can still keep running critical components as **static pods**.

---

# Why Static Pods? (Use Cases)

1. **Bootstrapping a cluster** – When setting up Kubernetes manually (kubeadm or from scratch), critical components like `kube-apiserver`, `kube-scheduler`, `etcd` often run as static pods.
    
2. **Critical node-level workloads** – Monitoring agents, log collectors, or any pod that must always run on a specific node regardless of cluster state.
    
3. **Testing / Troubleshooting** – Run a pod directly via kubelet when the control plane is not working.

---


# How Static Pods Work

- Kubelet has two ways to manage static pods:
    
    1. **Static Pod Path (`--pod-manifest-path`)**
        
        - You configure kubelet with a directory (e.g., `/etc/kubernetes/manifests`).
            
        - Any YAML file placed here is automatically created as a Pod.
            
    2. **HTTP File (`--manifest-url`)**
        
        - Kubelet can fetch pod definitions from a remote URL.
            
- Kubelet creates a **Mirror Pod** in the API server:
    
    - Mirror Pod = Representation of the static pod inside the Kubernetes API (read-only).
        
    - This allows you to see static pods with `kubectl get pods -n kube-system`.
        
    - But you cannot delete/update them via kubectl (you must modify the YAML file on the node).
---

# Creating a Static Pod (Step by Step)

### Step 1: Check the kubelet config

```yaml
cat /var/lib/kubelet/config.yaml | grep staticPodPath
```

You’ll see something like:

```yaml
staticPodPath: /etc/kubernetes/manifests
```

### Step 2: Create a Pod manifest

Example: `nginx-static.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-static
  namespace: default
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

### Step 3: Place it in the directory

```yaml
sudo mv nginx-static.yaml /etc/kubernetes/manifests/
```

### Step 4: Verify

```yaml
kubectl get pods
```

**You’ll see:**

```yaml
nginx-static-<node-name>   Running   ...
```

**The pod name includes the node name because static pods are tied to nodes.**


---

# Properties of Static Pods

1. **Node-bound**: Always tied to one specific node.
    
2. **Lifecycle managed by kubelet**:
    
    - If file is deleted → pod stops.
        
    - If kubelet restarts → pod is recreated.
        
3. **No controllers**:
    
    - If pod crashes, kubelet restarts it, but no rescheduling to another node.
        
4. **Mirror pods** appear in API server:
    
    - Useful for monitoring/logging.
        
    - Read-only representation.

---

# Mirror Pods in Detail

- Created automatically by kubelet in API server.
    
- If you `kubectl describe pod`, you’ll see annotation:

```yaml
annotations:
  kubernetes.io/config.source: file
```

- Deleting a mirror pod with `kubectl delete pod` won’t delete the actual static pod → kubelet just recreates it.

---

# **Summary**

- Static Pods are node-local pods managed by kubelet, not the control plane.
    
- Used mainly for critical cluster components and bootstrapping.
    
- Created by placing YAML files in a directory watched by kubelet.
    
- Visible in API server as mirror pods but not directly controllable.
    
- Good for infra; not ideal for applications.