
# Kubernetes DaemonSets



## What is a DaemonSet?

- A **DaemonSet** ensures that **one copy of a specific Pod runs on each node** in the cluster.
    
- When a **new node** joins the cluster → the DaemonSet automatically schedules the Pod there.
    
- When a **node is removed** → the Pod is also removed.
    
- Guarantees **uniform pod distribution** across nodes.


## Difference from ReplicaSet / Deployment

- **ReplicaSet / Deployment**: Ensures a defined number of replicas across nodes (could be multiple per node, or none).
    
- **DaemonSet**: Ensures **exactly one Pod per node**.


## Use Cases of DaemonSets

1. **Monitoring & Logging Agents**
    
    - Deploy monitoring agents (e.g., Prometheus Node Exporter, Fluentd, Filebeat) on every node.
        
    - Simplifies cluster-wide observability.
        
2. **Networking Components**
    
    - Network plugins (e.g., CNI plugins like Calico, Flannel, Weave Net).
        
    - Require a **node-local agent** on each node.
        
3. **Kubernetes Components**
    
    - `kube-proxy` is a key example (runs on every node, often managed as a DaemonSet).


## DaemonSet YAML Structure

Similar to a ReplicaSet but with `kind: DaemonSet`.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
      - name: agent
        image: my-monitoring-agent:latest
```

**Key Point**:
- `spec.selector` labels **must match** pod template labels.

## How Does a DaemonSet Work?

- **Before v1.12**: DaemonSets bypassed the scheduler using `nodeName` to directly bind pods.
    
- **From v1.12 onwards**:
    
    - Uses the **default scheduler**.
        
    - Relies on **NodeAffinity rules** to ensure pods are scheduled on all nodes.


## Commands


```yaml
# Create a DaemonSet from YAML
kubectl apply -f daemonset.yaml

# View DaemonSets
kubectl get daemonset

# Describe details
kubectl describe daemonset my-daemonset

# Check pods scheduled per node
kubectl get pods -o wide
```


## Summary

- **DaemonSet = one pod per node** (automatically added/removed as nodes change).
    
- **Best for**: Monitoring, logging, networking agents, kube-proxy.
    
- **Similar to ReplicaSet**, but ensures node-wide presence instead of fixed replica count.

