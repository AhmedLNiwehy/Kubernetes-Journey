
# Taints and Tolerations in Kubernetes

## What are Taints and Tolerations?

- **Taint (on Node):** A _repellent_ applied on a node that tells Kubernetes scheduler: _“Do not place pods here unless they can tolerate this taint.”_
    
- **Toleration (on Pod):** A property on a pod that says: _“I can live with this taint, schedule me if needed.”_
    

  **Key idea:**

- Taints **restrict** pods from being placed on nodes.
    
- Tolerations **allow exceptions** so some pods can still run there.
    

---

## Analogy: Bug and Repellent

- **Person = Node**
    
- **Bug = Pod**
    
- **Repellent Spray = Taint**
    
- By default, bugs (pods) cannot land on a sprayed person (tainted node).
    
- But some bugs may be resistant (pods with toleration) → they can still land.
    

  Two things decide if a bug can land on a person:

1. The taint on the person (node).
    
2. The bug’s toleration to that taint.
    

---

##  Example Cluster

- 3 Nodes: `node1`, `node2`, `node3`
    
- 4 Pods: `A`, `B`, `C`, `D`
    

### Without Taints

- Scheduler balances pods across nodes evenly.
    

### With Taints

- Apply taint on `node1`: `app=blue:NoSchedule`
    
- Pods **without toleration** → cannot be scheduled on `node1`.
    
- Only pods with matching toleration can run there.
    

  Example flow:

- Pod A → tries `node1`, rejected, goes to `node2`
    
- Pod B → rejected from `node1`, goes to `node3`
    
- Pod C → rejected from `node1`, goes to `node2`
    
- Pod D → has toleration, accepted on `node1`
    

---

## How to Apply Taints

```bash
kubectl taint nodes node1 app=blue:NoSchedule
```

- **Key =** `app`
    
- **Value =** `blue`
    
- **Effect =** `NoSchedule`
    

### Taint Effects

1. **NoSchedule** → pods without toleration cannot be scheduled.
    
2. **PreferNoSchedule** → avoid scheduling pods if possible, but not guaranteed.
    
3. **NoExecute** →
    
    - New pods without toleration are not scheduled.
        
    - Existing pods without toleration are **evicted (killed)**.
        

---

## 🔹 Adding Tolerations to Pods

In Pod YAML definition:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-d
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```

   Notes:

- Taints are on **nodes**.
    
- Tolerations are on **pods**.
    
- Double quotes required around values.
    

---

## NoExecute Example

```bash
kubectl taint nodes node1 app=blue:NoExecute
```

- Any running pods on `node1` without toleration → **evicted**.
    
- Pods with toleration → stay.
    

---

##  Master Node Taints

- By default, Kubernetes **masters** are tainted to prevent pods from running there.
    
- Example:
    
    ```bash
    kubectl describe node master | grep Taints
    ```
    
- You’ll see something like:
    
    ```
    Taints: node-role.kubernetes.io/control-plane:NoSchedule
    ```
    
- Best practice: **Do not run workloads on master.**
    

---

##  Important Takeaways

1. **Taints restrict nodes**, not pods.
    
2. **Tolerations allow pods** to bypass taints.
    
3. Taints do **not force** pods onto a node → they just prevent others from entering.
    
4. Use **node affinity** if you want to force pods to specific nodes.
    

---

##  Practice Commands

- Add a taint:
    
    ```bash
    kubectl taint nodes node1 app=blue:NoSchedule
    ```
    
- Remove a taint:
    
    ```bash
    kubectl taint nodes node1 app=blue:NoSchedule-
    ```
    
- Check node taints:
    
    ```bash
    kubectl describe node node1 | grep Taint
    ```
    
- Add toleration in pod YAML:
    
    ```yaml
    tolerations:
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
    ```