
# Admission Controllers in Kubernetes

### **Request Flow in Kubernetes**

When you issue a command using `kubectl`, such as creating a Pod:

1. The request goes to the **API Server**.
    
2. It passes through:
    
    - **Authentication** → verifies _who_ you are (e.g., using certificates in the kubeconfig).
        
    - **Authorization** → checks _what_ you’re allowed to do (via RBAC: roles and permissions).
        
3. Finally, before being persisted to **etcd**, it goes through **Admission Controllers**.
    

---

### What Are Admission Controllers?**

Admission Controllers are **plugins** in the API Server that can:

- **Validate** requests (e.g., reject Pods that run as root).
    
- **Mutate** requests (e.g., automatically add labels or inject defaults).
    
- **Enforce security and policy rules** beyond what RBAC can handle.
    

They sit between the authorization phase and the actual persistence of the object.

---

### **Why We Need Admission Controllers**

RBAC controls _who_ can do _what_.  
Admission Controllers control _how_ resources are created or modified.

**Examples:**

- Reject Pods using images from Docker Hub — enforce internal registries only.
    
- Disallow `latest` image tags.
    
- Enforce required metadata labels.
    
- Ensure security settings like “no root containers”.
    

---

### Types of Admission Controllers**

There are many built-in controllers. Some examples:

|Controller|Function|
|---|---|
|`AlwaysPullImages`|Ensures every time a Pod starts, the image is pulled again.|
|`DefaultStorageClass`|Automatically assigns a default StorageClass to PVCs if not specified.|
|`EventRateLimit`|Limits API request rates to prevent overload.|
|`NamespaceExists`|Rejects requests to non-existent namespaces.|
|`NamespaceLifecycle`|Prevents deleting system namespaces like `default` or `kube-system`.|

---

### **Viewing Enabled Admission Controllers**

Run this command inside the API server pod:

```yaml
k exec -it <api-server-pod-name> -n kube-system -- kube-apiserver -h | grep enable-admission-plugins
```

This shows all **enabled** admission plugins.

---

### **Enabling or Disabling Admission Controllers**

In a **kubeadm** setup:

1. Edit `/etc/kubernetes/manifests/kube-apiserver.yaml`.
    
2. Add or modify these flags under the command section:

```yaml
--enable-admission-plugins=NamespaceLifecycle,DefaultStorageClass
--disable-admission-plugins=DefaultStorageClass
```

3. Save the file — the kubelet will automatically restart the API server pod.

---

### **Example: Enabling NamespaceAutoProvision**

If enabled, when you create a Pod in a non-existent namespace:

- The controller automatically creates the namespace.
    
- Then proceeds to create the Pod.
    

If disabled, the request fails with:  
`Error from server (NotFound): namespaces "blue" not found`

---

### **Final Note**

The current best practice is to rely on:

- `NamespaceLifecycle` for namespace validation.
    
- Policy engines like **OPA Gatekeeper** or **Kyverno** for more complex admission rules.