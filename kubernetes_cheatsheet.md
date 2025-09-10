# Kubernetes Comprehensive Cheat Sheet

This cheat sheet provides a detailed overview of common Kubernetes commands and their declarative YAML equivalents.

## 1. `kubectl` Basic & Context Commands

These are essential commands for interacting with your cluster.

| Command | Description | Example |
|---|---|---|
| `kubectl version` | Display client and server versions. | `kubectl version` |
| `kubectl cluster-info` | Display information about the master and services. | `kubectl cluster-info` |
| `kubectl config get-contexts` | List all available cluster contexts. | `kubectl config get-contexts` |
| `kubectl config current-context`| Get the current context. | `kubectl config current-context` |
| `kubectl config use-context <name>`| Switch to a different context. | `kubectl config use-context docker-desktop` |
| `kubectl get all --all-namespaces`| List all resources in all namespaces. | `kubectl get all -A` |

---

## 2. Labels and Selectors

Labels are key/value pairs attached to objects like Pods. They are used to organize and to select subsets of objects. Selectors are the core grouping mechanism in Kubernetes, used by services and controllers to identify the Pods they operate on.

- **Labels:** Defined in the `metadata` section of an object.
  ```yaml
  metadata:
    labels:
      app: my-app
      tier: frontend
  ```
- **Selectors:** Used in the `spec` of a controller or service to select Pods with matching labels.
  ```yaml
  # Example in a Service
  spec:
    selector:
      app: my-app # Selects any pod with the label 'app: my-app'
  ```

---

## 3. Namespaces

Namespaces are used to create virtual clusters within your physical cluster, providing a scope for names.

### Imperative Command

```bash
# Create a namespace
kubectl create namespace <namespace-name>

# Example
kubectl create namespace dev
```

### Declarative (YAML)

**`namespace.yaml`**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

---

## 4. Pods

A Pod is the smallest and simplest unit in the Kubernetes object model that you create or deploy.

### Imperative Command

```bash
# Run a pod with a specific image
kubectl run <pod-name> --image=<image-name>

# Example
kubectl run my-nginx-pod --image=nginx:latest
```

### Declarative (YAML)

**`pod.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx-pod
  labels:
    app: web
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

---

## 5. Deployments

Deployments provide declarative updates for Pods and ReplicaSets, managing stateless applications.

### Imperative Command

```bash
# Create a deployment
kubectl create deployment <deployment-name> --image=<image-name>

# Scale a deployment
kubectl scale deployment <deployment-name> --replicas=<replica-count>
```

### Declarative (YAML)

**`deployment.yaml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx-deployment
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
        image: nginx:latest
        ports:
        - containerPort: 80
```

---

## 6. Services

A Service is an abstraction which defines a logical set of Pods and a policy by which to access them.

### Imperative Command

```bash
# Expose a deployment with a service
kubectl expose deployment <deployment-name> --type=<service-type> --port=<port> --target-port=<target-port>
```

### Declarative (YAML) - NodePort

**`service-nodeport.yaml`**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

---

## 7. ConfigMaps and Secrets

- **ConfigMap:** Stores non-confidential data in key-value pairs.
- **Secret:** Stores sensitive information, such as passwords or API keys.

### Imperative Command

```bash
# Create a ConfigMap from a literal value
kubectl create configmap app-config --from-literal=app.color=blue

# Create a Secret from a literal value
kubectl create secret generic db-credentials --from-literal=username=admin --from-literal=password='S3cr3t'
```

### Declarative (YAML) - Secret

**`secret.yaml`**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  # Values must be base64 encoded
  username: YWRtaW4=
  password: UzNjcmV0
```

---

## 8. Volumes, PersistentVolumes (PV), and PersistentVolumeClaims (PVC)

- **Volume:** A directory accessible to containers in a Pod. Its lifecycle is tied to the Pod. Common types include `emptyDir`, `hostPath`, `configMap`, `secret`.
- **PersistentVolume (PV):** A piece of storage in the cluster provisioned by an administrator. It is a resource in the cluster, independent of any individual Pod.
- **PersistentVolumeClaim (PVC):** A request for storage by a user. It consumes PV resources. Pods request storage by claiming a PVC.

### Declarative (YAML) - PVC and a Pod that uses it

**`pvc.yaml`**
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
      storage: 1Gi
```

**`pod-with-pvc.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-with-storage
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
      - name: my-storage
        mountPath: /data
  volumes:
    - name: my-storage
      persistentVolumeClaim:
        claimName: my-pvc
```

---

## 9. ReplicaSets

A ReplicaSet ensures that a specified number of replica Pods are running at any given time. **Note:** It's highly recommended to use Deployments, which manage ReplicaSets for you.

### Declarative (YAML)

**`replicaset.yaml`**
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-rs
  template:
    metadata:
      labels:
        app: nginx-rs
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```

---

## 10. StatefulSets

Manages the deployment and scaling of a set of Pods for stateful applications, providing guarantees about the ordering and uniqueness of these Pods.

### Declarative (YAML)

**Requires a Headless Service:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  clusterIP: None
  selector:
    app: nginx-sts
```
**`statefulset.yaml`**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx-headless"
  replicas: 2
  selector:
    matchLabels:
      app: nginx-sts
  template:
    metadata:
      labels:
        app: nginx-sts
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

---

## 11. DaemonSets

A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. Useful for cluster-wide agents like log collectors or monitoring agents.

### Declarative (YAML)

**`daemonset.yaml`**
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-agent
spec:
  selector:
    matchLabels:
      name: fluentd-agent
  template:
    metadata:
      labels:
        name: fluentd-agent
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1
```

---

## 12. Jobs and CronJobs

- **Job:** Creates one or more Pods and ensures that a specified number of them successfully terminate. For run-to-completion tasks.
- **CronJob:** Creates Jobs on a repeating schedule.

### Declarative (YAML) - CronJob

**`cronjob.yaml`**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cron
spec:
  schedule: "*/1 * * * *" # Runs every minute
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            command:
            - /bin/sh
            - -c
            - date; echo "Hello from the CronJob"
          restartPolicy: OnFailure
```

---

## 13. Resource Management (Requests & Limits)

Specify CPU and memory needs for each container to ensure reliable performance and cluster stability.

- **requests:** Guaranteed resources for a container.
- **limits:** Maximum resources a container can use.

**`pod-with-resources.yaml`**
```yaml
spec:
  containers:
  - name: my-container
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

---

## 14. Probes (Health Checks)

Probes are used by the kubelet to check the health of a container.

- **livenessProbe:** Checks if the container is running. If it fails, the container is restarted.
- **readinessProbe:** Checks if the container is ready to serve traffic. If it fails, it's removed from service endpoints.
- **startupProbe:** Checks if the application within the container has started.

**`pod-with-probes.yaml`**
```yaml
spec:
  containers:
  - name: my-app
    image: my-app-image
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

---

## 15. Horizontal Pod Autoscaler (HPA)

Automatically scales the number of pods in a deployment based on observed CPU utilization or other metrics.

### Imperative Command
```bash
kubectl autoscale deployment <name> --cpu-percent=50 --min=1 --max=10
```

### Declarative (YAML)
**`hpa.yaml`**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-nginx-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

---

## 16. Ingress

Manages external access to services in a cluster, typically HTTP/HTTPS. **Requires an Ingress Controller.**

### Declarative (YAML)
**`ingress.yaml`**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: "myapp.example.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-nginx-service
            port:
              number: 80
```

---

## 17. Network Policies

Provide firewall-like capabilities to control traffic flow between Pods.

### Declarative (YAML) - Allow ingress from specific pods
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

---

## 18. RBAC (Role-Based Access Control)

Manages who can do what within the cluster.

- **Role/ClusterRole:** A set of permissions.
- **RoleBinding/ClusterRoleBinding:** Grants the permissions in a Role to a set of users.

### Declarative (YAML) - Role and RoleBinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
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

---

## 19. Taints and Tolerations

- **Taints:** Applied to Nodes to repel Pods.
- **Tolerations:** Applied to Pods to allow them to be scheduled on Nodes with matching taints.

### Imperative Command
```bash
kubectl taint nodes node1 app=blue:NoSchedule
```
### Declarative (YAML) - Pod with a Toleration
```yaml
spec:
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```

---

## 20. Affinity and Anti-Affinity

A property of Pods that attracts them to or repels them from a set of other objects.

- **Node Affinity:** Attracts a Pod to a set of nodes based on node labels.
- **Pod Affinity / Anti-Affinity:** Attracts/repels a Pod from other Pods based on pod labels.

### Declarative (YAML) - Pod with Node Affinity
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
```

---

## 21. Debugging and Monitoring

| Command | Description |
|---|---|
| `kubectl get events --sort-by='.lastTimestamp'` | List all events, sorted by time. Crucial for finding problems. |
| `kubectl describe <resource> <name>` | The best all-purpose command for inspecting a resource and finding issues. |
| `kubectl logs <pod-name>` | View the logs from a pod (`-f` to follow). |
| `kubectl exec -it <pod-name> -- /bin/sh` | Get an interactive shell inside a running pod. |
| `kubectl port-forward <pod-name> <local>:<remote>` | Forward a local port to a port on a pod. |
| `kubectl top pod` / `kubectl top node` | View CPU and Memory usage. |

---

## 22. Package Management with Helm

Helm is the package manager for Kubernetes. It uses "charts" to define, install, and upgrade applications.

| Command | Description |
|---|---|
| `helm repo add <name> <url>` | Add a chart repository. |
| `helm search repo <keyword>` | Search for charts. |
| `helm install <release> <chart>` | Install a chart. |
| `helm list` | List all releases. |
| `helm upgrade <release> <chart>` | Upgrade a release. |
| `helm uninstall <release>` | Uninstall a release. |