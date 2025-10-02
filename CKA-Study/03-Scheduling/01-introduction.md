## 1. **What is Scheduling in Kubernetes?**

- **Definition**: Scheduling is the process of assigning Pods to Nodes in a Kubernetes cluster.
    
- **Why it matters**: Not every Pod can run on every Node. Nodes differ in CPU, memory, labels, taints, and affinities. Scheduling ensures Pods are placed where they can run successfully and efficiently.
    

  The default component responsible for this is the **kube-scheduler**.

## 2. **Scheduling Workflow (High Level)**

When you create a Pod:

1. **API Server receives the Pod manifest** (without a Node name).
    
2. **kube-scheduler watches** for unscheduled Pods (Pods without `spec.nodeName`).
    
3. Scheduler does two things:
    
    - **Filtering** → Eliminate nodes that don’t fit the Pod’s requirements.
        
    - **Scoring** → Rank the remaining nodes based on preferences.
        
4. **Binding** → kube-scheduler assigns the Pod to the best Node by writing the `spec.nodeName`.


## 3. What If There Is No Scheduler ?

- Pods that don’t specify a **`nodeName`** or a **custom scheduler** will stay in **`Pending`** state forever.
    
    - They’re waiting for someone (the scheduler) to assign them a node.
        
- The control plane doesn’t “auto-fix” this — scheduling is a separate responsibility.

## 4. How to Run Pods Without kube-scheduler ?

Even if the scheduler is missing, you have **manual options**:

### 1. **Set `nodeName`**

If you specify `nodeName` in your Pod spec, the API server directly assigns the Pod to that node (scheduler is skipped).

```yaml
spec:
  nodeName: node01
```

### 2. **Manual Binding Object**

You can bind the Pod yourself with a `Binding` object:

```yaml
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node01
```
You’d apply this to the API server, and it will attach the Pod to that node.

### 3. **Write Your Own Scheduler**

You can run your own process that:

1. Watches for unscheduled Pods.
    
2. Picks a node (based on your logic).
    
3. Creates a Binding object.
    

That’s how custom schedulers are built.

## **Summary**  
If there’s no scheduler:

- Pods without `nodeName` → stuck in **Pending**.
    
- Pods with `nodeName` or manual Binding → still run.
    
- You can replace kube-scheduler with your own if you want.