## 4. Security & Access Control

### 4.1 Authentication & Authorization

#### Service Accounts

A **Service Account** in Kubernetes provides an identity for processes running in pods. This identity can be used to interact with the Kubernetes API server and access cluster resources. By default, Kubernetes automatically creates a default service account in every namespace. However, custom service accounts can be created to grant specific permissions.

Service accounts are typically associated with **Role-Based Access Control (RBAC)** roles to define what resources the service account can access.

**Example: Creating a Service Account**

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
```

To associate this service account with a pod:

**Example: Pod with Service Account**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-service-account
spec:
  serviceAccountName: my-service-account
  containers:
    - name: app
      image: nginx
```

#### Role-Based Access Control (RBAC)

**RBAC** in Kubernetes allows fine-grained access control to resources based on roles and the actions (verbs) users can perform on them. RBAC uses **Roles** (within namespaces) or **ClusterRoles** (cluster-wide) that define the allowed operations on resources. A **RoleBinding** or **ClusterRoleBinding** links the role to a user, group, or service account.

- **Role** defines access within a specific namespace.
- **ClusterRole** defines access cluster-wide.
- **RoleBinding** binds a Role to a user or service account.
- **ClusterRoleBinding** binds a ClusterRole to a user or service account cluster-wide.

**Example: Creating a Role and RoleBinding**

**Role (namespace-specific)**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: read-pods
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]
```

**RoleBinding**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: default
subjects:
  - kind: ServiceAccount
    name: my-service-account
    namespace: default
roleRef:
  kind: Role
  name: read-pods
  apiGroup: rbac.authorization.k8s.io
```

**ClusterRole (cluster-wide)**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin
rules:
  - apiGroups: [""]
    resources: ["pods", "services"]
    verbs: ["get", "list", "create", "delete"]
```

**ClusterRoleBinding**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-binding
subjects:
  - kind: ServiceAccount
    name: my-service-account
    namespace: default
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

#### Kubernetes API Access Control

Kubernetes API access control determines **who** can access the API server and **what** actions they can perform. It is managed through authentication (validating identity) and authorization (validating what actions can be performed) mechanisms.

Kubernetes supports several authentication methods, including:

- **Service Account Tokens**: Automatically generated for pods.
- **Bearer Tokens**: Tokens that allow users or services to authenticate to the API server.
- **Client Certificates**: Allows users to authenticate via TLS client certificates.

For authorization, RBAC is used to define which users, groups, or service accounts have access to which resources and actions.

### 4.2 Pod & Cluster Security

#### Pod Security Standards (PSS) & Admission Controllers

**Pod Security Standards (PSS)** define baseline security standards for Kubernetes pods. These standards enforce policies like:

- Preventing privileged containers
- Enforcing user and group IDs
- Ensuring containers run as non-root

**Admission Controllers** are software components that intercept incoming API requests before they are persisted in the cluster. They can enforce policies such as PSS.

The following are common **Admission Controllers**:

- **PodSecurityPolicy** (deprecated in newer versions in favor of PodSecurity)
- **SecurityContextDeny** (denies requests that define privileged containers)

**Example: Enabling Pod Security Standards via PodSecurity Admission Controller**

```bash
kubectl create ns dev --dry-run=client -o yaml | kubectl apply -f -
kubectl label --overwrite ns dev pod-security.kubernetes.io/enforce=restricted
```

#### Seccomp, AppArmor, SELinux

These are **security modules** used to enhance the security of Kubernetes by restricting the system calls and resources available to containers. They are used to ensure containers cannot perform actions that may compromise the node's security.

- **Seccomp (Secure Computing Mode)**: Limits the system calls available to containers. You can enforce a seccomp profile to restrict actions such as network access, file access, etc.
- **AppArmor**: Provides mandatory access control (MAC) to restrict program behavior by defining security profiles.
- **SELinux (Security-Enhanced Linux)**: Uses MAC policies to restrict container actions.

**Example: Seccomp Profile in Pod**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  containers:
    - name: app
      image: nginx
      securityContext:
        seccompProfile:
          type: Localhost
          localhostProfile: /profiles/seccomp.json
```

#### Image Security

**Image Security** ensures that container images do not contain vulnerabilities that can be exploited. Tools like **Trivy**, **Cosign**, and **Notary** can be used to scan container images for known security vulnerabilities.

- **Trivy**: Scans container images for vulnerabilities.
- **Cosign**: Provides image signing and verification to ensure the image has not been tampered with.
- **Notary**: Ensures that images are signed and have not been modified after they were pushed.

**Example: Scanning Image with Trivy**

```bash
trivy image my-image:latest
```

#### Securing etcd

**etcd** is a critical component in Kubernetes, storing the cluster’s state and configurations. To ensure security:

- **etcd Encryption**: Enable encryption for sensitive data in etcd to protect it at rest.
- **Access Control**: Limit who can access etcd, using secure authentication methods.
- **Backup**: Regularly back up etcd to ensure you can restore the cluster state in case of failure.

**Example: Enabling etcd Encryption**

```yaml
apiVersion: etcd.database.coreos.com/v1
kind: EtcdCluster
metadata:
  name: etcd-cluster
spec:
  spec:
    encryption:
      enable: true
```

### 4.3 Network Security

#### Network Policies

**Network Policies** in Kubernetes allow you to control the communication between pods and services within the cluster. By default, all pods can communicate with each other. Network policies provide a way to restrict traffic between pods, defining **allow** and **deny** rules.

**Example: Creating a Network Policy**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: frontend
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: backend
```

In this example, the **frontend** pods are allowed to receive traffic only from **backend** pods.

#### Service Mesh Security

A **Service Mesh** like **Istio** or **Linkerd** adds a layer of security, monitoring, and routing to manage microservices communication within a cluster. **mTLS (Mutual TLS)** is commonly used within service meshes to secure communication between services by encrypting traffic and authenticating both the client and the server.

**Example: Enabling mTLS with Istio**

```bash
istioctl install --set profile=demo
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
```

This sets up mTLS between services in an Istio-enabled cluster, ensuring that communication between microservices is encrypted and authenticated.
