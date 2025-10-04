
## How the Kubernetes Scheduler Works

When a **Pod** is created, it goes through these stages:

1. **Queueing Phase:**
    
    - All pending pods wait in a **scheduling queue**.
        
    - The `PrioritySort` plugin decides which pods should be scheduled first (based on PriorityClass).
        
2. **Filtering Phase:**
    
    - The scheduler removes nodes that **can’t host the pod**, using plugins such as:
        
        - `NodeResourcesFit` (checks CPU/memory requirements)
            
        - `NodeUnschedulable` (skips drained or cordoned nodes)
            
        - `NodeName` (if a pod is tied to a specific node)
            
3. **Scoring Phase:**
    
    - The remaining nodes are **scored** using plugins like:
        
        - `NodeResourcesFit` (prefers nodes with more free resources)
            
        - `ImageLocality` (prefers nodes with the image already pulled)
            
4. **Binding Phase:**
    
    - The `DefaultBinder` plugin **binds** the pod to the chosen node.

---

## Scheduler Framework & Plugins

The **Kubernetes Scheduler Framework** allows developers to **customize or extend** scheduling behavior using **plugins**.

Each plugin can hook into different “**extension points**,” such as:

- `PreFilter`
    
- `Filter`
    
- `PostFilter`
    
- `Score`
    
- `Reserve`
    
- `Permit`
    
- `Bind`
    
- `PostBind`
    

This modular approach allows fine control over scheduling logic.

---

## Problem with Multiple Schedulers (Before v1.18)

Before Kubernetes v1.18:

- If you needed different scheduling behaviors, you had to **run multiple scheduler processes** (e.g., `default-scheduler`, `my-scheduler`, `my-scheduler-2`).
    
- Each process had its own config file and binary.
    
- This caused **extra overhead** and **race conditions** (two schedulers scheduling on the same node simultaneously).
    

---

## Scheduler Profiles (Since Kubernetes v1.18)

**Introduced in KEP-1451**, Scheduler Profiles let you define **multiple scheduling configurations** inside **one scheduler process**.

Each profile:

- Has a **unique `schedulerName`** (used in the pod’s spec).
    
- Can enable/disable **different plugins** at each phase.
    

So now, instead of running multiple schedulers, you can define multiple **profiles** in the scheduler’s configuration file.

---

## Example: Scheduler Configuration with Multiple Profiles

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: default-scheduler
    plugins:
      score:
        disabled:
          - name: NodeResourcesBalancedAllocation
        enabled:
          - name: ImageLocality

  - schedulerName: high-priority-scheduler
    plugins:
      preFilter:
        disabled:
          - name: TaintToleration
      filter:
        enabled:
          - name: NodeAffinity
```

Then, when you define a Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  schedulerName: high-priority-scheduler
  containers:
    - name: nginx
      image: nginx
```

The pod will be scheduled using the `high-priority-scheduler` profile — even though both schedulers are part of the same scheduler binary!

---

## Summary

| Concept                 | Description                                                                                        |
| ----------------------- | -------------------------------------------------------------------------------------------------- |
| **Scheduler Framework** | Modular system that uses plugins for each scheduling phase                                         |
| **Plugin**              | A piece of code that defines how a phase behaves (filter, score, bind, etc.)                       |
| **Extension Point**     | A hook where you can attach a plugin                                                               |
| **Scheduler Profile**   | A configuration of plugins defining a unique scheduling behavior inside the same scheduler process |
| **KEP-1451**            | Kubernetes Enhancement Proposal introducing multi-scheduler profiles                               |
