## 2. Core Kubernetes Objects

### 2.1 Pods (Basic Unit of Deployment)

#### What is a Pod?

A **Pod** is the smallest and simplest unit in Kubernetes and represents a single instance of a running process in the cluster. A pod encapsulates one or more containers, storage, and networking resources.

#### Key Characteristics of Pods:

- Pods run on **worker nodes**.
- Pods share the same **network namespace**, meaning they can communicate with each other over `localhost`.
- Pods have **shared storage volumes** that allow containers within the pod to communicate and persist data.
- Pods are **ephemeral**, meaning they don’t exist permanently, and can be created, destroyed, or re-created based on the workload.

#### Multi-container Pods

A **multi-container pod** runs more than one container in a single pod. These containers share the pod's network and storage. They are used when containers need to tightly work together and share resources, such as logs or configuration files.

For example, you may have a pod containing a main application container and a sidecar container that handles logging or monitoring.

**Example: Pod with Multi-containers YAML**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
    - name: app-container
      image: nginx
    - name: sidecar-container
      image: busybox
      command: ["sh", "-c", "while true; do echo 'Logging'; sleep 30; done"]
```

#### Pod Lifecycle & States

Pods have various lifecycle states that describe their current status:

1. **Pending**: The pod has been accepted by the Kubernetes cluster but is not yet running on any node.
2. **Running**: The pod is running and all of its containers are actively executing.
3. **Succeeded**: All containers in the pod have completed successfully.
4. **Failed**: At least one container in the pod has terminated in failure.
5. **Unknown**: The state of the pod cannot be determined.

**Command to Check Pod Status:**

```bash
kubectl get pods
```

#### Init Containers & Ephemeral Containers

- **Init Containers**: These are special containers that run before the main containers in a pod. They are typically used for setup tasks, like initializing configuration or waiting for resources.
- **Ephemeral Containers**: These are temporary containers that can be added to running pods for troubleshooting or debugging. They are not part of the pod's lifecycle and are removed once the troubleshooting session ends.

**Example of Pod with Init Containers:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-init-container
spec:
  initContainers:
    - name: init-container
      image: busybox
      command: ["sh", "-c", "echo Initializing; sleep 5"]
  containers:
    - name: main-container
      image: nginx
```

---

### 2.2 Controllers & Workloads

#### ReplicaSet

A **ReplicaSet** ensures that a specified number of pod replicas are running at any given time. If a pod crashes or is deleted, the ReplicaSet creates a new one to maintain the desired number of replicas.

**Example ReplicaSet YAML:**

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: nginx
          image: nginx
```

**Command to Scale ReplicaSet:**

```bash
kubectl scale replicaset my-replicaset --replicas=5
```

#### Deployment

A **Deployment** is a higher-level controller that manages ReplicaSets and ensures that the desired number of replicas are running. It supports rolling updates and rollbacks, allowing for smooth application updates.

**Example Deployment YAML:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
          image: nginx
```

**Command to Perform a Rolling Update:**

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.19.0
```

#### DaemonSet

A **DaemonSet** ensures that a specific pod runs on every node in the cluster. This is useful for monitoring, logging, or networking services that need to run on every node.

**Example DaemonSet YAML:**

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: my-daemonset
spec:
  selector:
    matchLabels:
      name: my-daemonset
  template:
    metadata:
      labels:
        name: my-daemonset
    spec:
      containers:
        - name: my-container
          image: my-image
```

**Command to Check DaemonSet Pods:**

```bash
kubectl get pods -l name=my-daemonset
```

#### StatefulSet

A **StatefulSet** is used for managing stateful applications, such as databases. It ensures that the pods are created and terminated in a specific order and provides guarantees around the ordering and uniqueness of the pod names.

**Example StatefulSet YAML:**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "web"
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: nginx
```

**Command to Scale StatefulSet:**

```bash
kubectl scale statefulset web --replicas=5
```

#### Job & CronJob

- **Job**: A **Job** creates one or more pods and ensures that they successfully terminate. It is typically used for batch processing tasks.

**Example Job YAML:**

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  template:
    spec:
      containers:
        - name: job-container
          image: busybox
          command: ["sh", "-c", "echo Hello World"]
      restartPolicy: OnFailure
```

- **CronJob**: A **CronJob** runs jobs on a scheduled basis, similar to a cron job in Linux systems.

**Example CronJob YAML:**

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cron-job
spec:
  schedule: "0 12 * * *" # Runs every day at 12:00 PM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: cron-container
              image: busybox
              command: ["sh", "-c", "echo Cron Job"]
          restartPolicy: OnFailure
```

---

### 2.3 Kubernetes Networking

#### Cluster Networking Model

In Kubernetes, networking allows pods to communicate with each other across nodes. Kubernetes abstracts networking in a way that each pod gets its own IP address. The network model ensures that all pods can reach each other across nodes, subject to network policies and services.

#### CNI (Container Network Interface)

The **Container Network Interface (CNI)** is a standard for managing networking configurations in Linux containers. Kubernetes supports various CNI plugins (like Calico, Flannel, and Weave) to provide pod networking.

#### Service Types

Kubernetes provides four types of services to expose applications to the network:

1. **ClusterIP**:  
   Exposes the service only inside the cluster. This is the default service type.

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: my-service
   spec:
     selector:
       app: my-app
     ports:
       - protocol: TCP
         port: 80
         targetPort: 8080
     type: ClusterIP
   ```

2. **NodePort**:  
   Exposes the service on each node's IP at a static port. This allows external access to the service via `NodeIP:NodePort`.

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: my-service
   spec:
     selector:
       app: my-app
     ports:
       - protocol: TCP
         port: 80
         targetPort: 8080
         nodePort: 30000
     type: NodePort
   ```

3. **LoadBalancer**:  
   Exposes the service via an external load balancer, often used in cloud environments (AWS, GCP, Azure).

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: my-service
   spec:
     selector:
       app: my-app
     ports:
       - protocol: TCP
         port: 80
         targetPort: 8080
     type: LoadBalancer
   ```

4. **ExternalName**:  
   Maps the service to an external DNS name (useful for integrating external services).
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: my-service
   spec:
     type: ExternalName
     externalName: example.com
   ```

#### Ingress Controllers & Ingress Rules

An **Ingress** is an API object that manages external HTTP and HTTPS access to services within a cluster. It requires an **Ingress Controller** like NGINX or Traefik to manage the routing of requests.

**Example Ingress Rule YAML:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
```

#### Network Policies

**Network Policies** are used to control the communication between pods. They define which pods can communicate with each other and which cannot.

**Example Network Policy YAML:**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-same-namespace
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector: {}
```

This policy allows pods in the same namespace to communicate with each other while preventing communication from outside the namespace.
