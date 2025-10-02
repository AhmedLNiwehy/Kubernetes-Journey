
# **Kubernetes  Node Affinity**

## What is Node Affinity?

Node Affinity is a Kubernetes scheduling feature that allows you to **control which nodes a Pod can run on** based on labels assigned to the nodes.

- It’s more powerful and flexible than **Node Selectors**.
    
- It supports advanced logic like `In`, `NotIn`, `Exists`, and more.
    
- It ensures Pods are scheduled only on nodes that match defined rules.
    

 **Use Case**: Run **resource-hungry data processing pods** only on large nodes.

---

## Example Setup

- Cluster has 3 nodes:
    
    - **Node 1** → Large (higher resources)
        
    - **Node 2 & Node 3** → Small (lower resources)
        
- Goal: Ensure the **data-processing pod** runs only on the large node.
    

---

## 🔑 Node Labels

Before using affinity, label your nodes:

```yaml
kubectl label nodes node1 size=large
kubectl label nodes node2 size=medium
# Leave node3 unlabeled (small)
```

## Node Affinity Structure

Node Affinity rules live under the `spec.affinity.nodeAffinity` section of a Pod manifest.

There are **two types (current)**:

1. **requiredDuringSchedulingIgnoredDuringExecution**
    
    - Mandatory rule → Pod won’t be scheduled unless conditions match.
        
    - "Hard rule."
        
2. **preferredDuringSchedulingIgnoredDuringExecution**
    
    - Best-effort rule → Scheduler tries to honor it but will fall back if no matches.
        
    - "Soft rule."
        

(Future: `requiredDuringExecution` will evict Pods if labels change.)

---

## Example 1 — Simple Required Rule (like Node Selector)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: data-processing
spec:
  containers:
  - name: processor
    image: my-data-processor:latest
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
            - large
```

Pod will only be scheduled on nodes with label `size=large`.

## Example 2 — Multiple Options (Large OR Medium)

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: size
          operator: In
          values:
          - large
          - medium
```

Pod can be scheduled on **large or medium nodes**.

## Example 3 — Excluding Nodes

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: size
          operator: NotIn
          values:
          - small
```

Pod will run on **any node except nodes labeled `small`**.

## Example 4 — Exists Operator

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: size
          operator: Exists
```

Pod will run on any node that has the label `size` (regardless of value).

---

## How Node Affinity Works

### During Scheduling

- Rules are checked **before the Pod is created**.
    
- If `required...` is used and no node matches → Pod stays in `Pending`.
    
- If `preferred...` is used and no node matches → Pod is placed **anywhere available**.
    

### During Execution

- Current types ignore changes after Pod is scheduled.
    
- If a label is removed → Pod **continues running**.
    
- Future type `requiredDuringExecution` will **evict Pods** if rules are broken.
    

---

## Key Takeaways

- Node Affinity = advanced Node Selector.
    
- Lets you use operators like `In`, `NotIn`, `Exists`.
    
- Use **required** for critical workloads.
    
- Use **preferred** for soft preferences.
    
- Always remember: **Affinity needs node labels to work**.
    

---

## Practice Ideas

1. Label your Minikube nodes as `size=large`, `size=medium`, and leave one unlabeled.
    
2. Deploy Pods with different affinity rules.
    
3. Test what happens when:
    
    - You remove a label from a node.
        
    - You use `required...` with no matching node.
        
    - You use `preferred...` with no matching node.
