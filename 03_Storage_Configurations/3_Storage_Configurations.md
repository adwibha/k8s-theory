## 3. Storage & Configurations

### 3.1 Persistent Data Storage

#### Volumes & Volume Types

In Kubernetes, **volumes** provide persistent storage to containers in pods. While containers are ephemeral and lose data when they are terminated, volumes persist across container restarts.

Some common **volume types** include:

- **emptyDir**: A temporary storage that exists as long as the pod is running. It is erased when the pod is deleted. Ideal for caching or scratch data.
- **hostPath**: Mounts a file or directory from the node’s filesystem into the pod. It’s commonly used for local storage or node-specific configurations, but it’s not suitable for production environments since it doesn’t work well in distributed environments.
- **PersistentVolume (PV)**: A persistent, cluster-wide storage resource that is provisioned by an administrator or dynamically via a StorageClass. It is independent of pods and can be reused across different pods and nodes.

- **PersistentVolumeClaim (PVC)**: A request for storage by a pod, specifying the amount of storage and access modes required. PVCs are bound to available PVs that meet the request.

**Example: emptyDir Volume**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-emptydir
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - mountPath: /data
          name: emptydir-volume
  volumes:
    - name: emptydir-volume
      emptyDir: {}
```

#### Persistent Volumes (PV) & Persistent Volume Claims (PVC)

- **PersistentVolume (PV)**: Represents actual storage resources, like cloud storage (EBS, GCE persistent disks), network file systems, or local disk storage.
- **PersistentVolumeClaim (PVC)**: Is a request for storage that is matched to a suitable PV, based on size, access modes, and storage class.

**Example: Persistent Volume (PV) YAML**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-example
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data
```

**Example: Persistent Volume Claim (PVC) YAML**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Once the PVC is created, it will be bound to the available PV that meets its requirements.

#### Storage Classes

A **StorageClass** defines different types of storage available in the cluster, and enables **dynamic provisioning** of PersistentVolumes (PVs). When you create a PVC, you can specify which StorageClass to use. If no class is specified, Kubernetes uses the default storage class.

**Example: StorageClass YAML**

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
```

In this example, the `provisioner` is set to `kubernetes.io/aws-ebs`, meaning AWS EBS volumes will be provisioned dynamically.

#### CSI (Container Storage Interface)

The **Container Storage Interface (CSI)** is a standardized interface that allows Kubernetes to interact with different storage providers. It enables Kubernetes to integrate with a wide variety of external storage systems, such as block storage, file storage, or object storage.

Kubernetes uses a CSI driver to manage external storage services (e.g., AWS EBS, Google Cloud Persistent Disks, Ceph, etc.).

---

### 3.2 Configuration Management

#### ConfigMaps

A **ConfigMap** is used to store configuration data in key-value pairs. It allows you to separate configuration from your application code, so that it can be updated independently. ConfigMaps can be injected into pods as environment variables, command-line arguments, or mounted as files.

**Example: Creating a ConfigMap**

```bash
kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2
```

**Example: ConfigMap YAML**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  config1: value1
  config2: value2
```

**Injecting ConfigMap Data into a Pod:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-configmap
spec:
  containers:
    - name: app
      image: nginx
      envFrom:
        - configMapRef:
            name: app-config
```

In the above example, `configMapRef` is used to inject all key-value pairs in the ConfigMap as environment variables into the pod.

#### Secrets

**Secrets** are used to store sensitive data, such as passwords, OAuth tokens, SSH keys, etc., in Kubernetes. They are stored securely and encoded in base64. Secrets can be injected into pods as environment variables or mounted as volumes.

**Example: Creating a Secret**

```bash
kubectl create secret generic my-secret --from-literal=password=mysecretpassword
```

**Example: Secret YAML**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  password: c2VjcmV0cGFzc3dvcmQ= # Base64 encoded password
```

**Injecting Secrets into a Pod:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-secret
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: SECRET_PASSWORD
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: password
```

In this example, the `SECRET_PASSWORD` environment variable will be injected into the pod with the decoded value from the secret.

#### Environment Variables & Config Injection

Environment variables are commonly used to inject configuration data into containers. This can be done using **ConfigMaps**, **Secrets**, or by directly specifying environment variables in the pod specification.

You can inject environment variables directly into your pod's container like so:

**Example: Injecting Environment Variables Directly into a Pod**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-env-vars
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: ENV_VAR
          value: "my_value"
```

Alternatively, you can use **ConfigMaps** or **Secrets** to manage and inject configuration data, which allows easier management of configuration changes without modifying your application code.
