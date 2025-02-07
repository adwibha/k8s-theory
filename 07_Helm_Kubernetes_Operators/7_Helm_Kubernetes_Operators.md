## 7. Helm & Kubernetes Operators

In Kubernetes, **Helm** and **Operators** are tools that simplify the management and automation of complex applications, making deployment and lifecycle management easier and more efficient. Below is a detailed guide on both tools, including how to use them, their advantages, and practical examples.

### 7.1 Helm – Kubernetes Package Manager

Helm is a package manager for Kubernetes, similar to apt or yum in the Linux ecosystem. It simplifies the deployment and management of applications in Kubernetes by using **charts**—pre-configured Kubernetes resources that define an application’s deployment.

#### Helm Charts & Templates

A **Helm chart** is a collection of files that describe a set of Kubernetes resources. It allows you to package Kubernetes applications in a reusable format. Helm charts contain:

- **Chart.yaml**: Defines metadata about the chart, such as its name, version, description, and dependencies.
- **values.yaml**: A configuration file where default values for the chart’s parameters are specified.
- **templates/**: A directory containing Kubernetes YAML files that define the resources (e.g., Pods, Services, Deployments). These templates can contain placeholders that are dynamically replaced during installation based on values from `values.yaml`.

**Example of a Chart Directory:**

```bash
my-chart/
  ├── Chart.yaml
  ├── values.yaml
  └── templates/
      ├── deployment.yaml
      ├── service.yaml
      └── ingress.yaml
```

- **deployment.yaml** might define a Kubernetes Deployment, with placeholders like `{{ .Values.replicaCount }}` for dynamic configuration.

**Example of a values.yaml:**

```yaml
replicaCount: 3
image:
  repository: my-app
  tag: latest
service:
  type: ClusterIP
  port: 80
```

#### Helm Repositories & values.yaml

A **Helm repository** is a collection of Helm charts stored in a central location, where you can find and share charts. You can use public repositories (like **Artifact Hub**), or create your own private repositories.

To add a repository:

```bash
helm repo add <repo-name> <repo-url>
helm repo update
```

Once the repository is added, you can install charts with:

```bash
helm install <release-name> <chart-name>
```

You can customize the installation of a Helm chart by modifying the `values.yaml` file. You can specify configuration overrides using the `--values` or `-f` flag:

```bash
helm install <release-name> <chart-name> -f custom-values.yaml
```

#### Helm 3 vs Helm 2

- **Helm 2**: Used a component called **Tiller**, which ran inside the Kubernetes cluster and had full control over the cluster’s resources. Tiller required elevated privileges, which posed a security risk.
- **Helm 3**: Removed **Tiller** and operates directly with Kubernetes using the permissions of the user executing the Helm commands. Helm 3 is more secure and does not require server-side components, making it simpler and easier to manage.

**Key Differences between Helm 3 and Helm 2**:

- **No Tiller** in Helm 3—Helm 3 interacts directly with the Kubernetes API server, eliminating the need for the Tiller component that Helm 2 required.
- **Helm 3 uses Namespaces**—Helm 3 supports namespace scoping directly without relying on Tiller's management.
- **Helm 3 uses improved release management** with new commands and flags.

**Command to Install Helm 3**:

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### 7.2 Kubernetes Operators

Kubernetes **Operators** extend Kubernetes' native capabilities by automating the deployment, scaling, and management of complex stateful applications. Operators are a way to package, deploy, and manage applications and resources in a Kubernetes-native manner.

#### What are Operators?

An **Operator** is a method of packaging, deploying, and managing a Kubernetes application. It leverages Kubernetes APIs, controllers, and custom resources to handle routine operational tasks (e.g., backups, scaling, upgrades, failover) for specific applications.

Operators use **Custom Resource Definitions (CRDs)** to extend Kubernetes and define custom objects (resources) that are managed by the Operator.

#### Custom Resource Definitions (CRDs)

A **Custom Resource Definition (CRD)** allows you to define custom Kubernetes resources. This extends Kubernetes’ native capabilities and allows you to create resources tailored to specific needs.

**Example of a CRD**:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: postgresqls.myorg.com
spec:
  group: myorg.com
  names:
    kind: PostgreSQL
    plural: postgresqls
    singular: postgresql
  scope: Namespaced
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                replicas:
                  type: integer
                  minimum: 1
                  maximum: 10
                image:
                  type: string
```

In this example, a custom resource `PostgreSQL` is defined, which has `replicas` and `image` as its properties. The operator will manage instances of this resource.

#### Writing an Operator with Operator SDK

The **Operator SDK** simplifies the process of writing Kubernetes Operators. It provides tools, libraries, and APIs to create custom operators and automate the lifecycle of applications.

**Steps to create an Operator using Operator SDK:**

1. **Install the Operator SDK**:

   ```bash
   curl -sSL https://github.com/operator-framework/operator-sdk/releases/download/v1.18.0/operator-sdk-v1.18.0-x86_64-linux-gnu.tar.gz | tar -xz -C /usr/local/bin/
   ```

2. **Create a New Operator**:

   ```bash
   operator-sdk init --domain myorg.com --repo github.com/myorg/myoperator
   ```

3. **Create API and Controller**:

   ```bash
   operator-sdk create api --group apps --version v1 --kind MyApp --resource --controller
   ```

4. **Implement the Controller Logic**: The controller watches the custom resources and performs specific tasks when the resource state changes (e.g., scaling the application or upgrading its version).

5. **Build and Deploy the Operator**:
   ```bash
   make install
   make run
   ```

After deploying, the Operator will start managing your custom resources in the cluster.

#### Examples of Kubernetes Operators

1. **Prometheus Operator**: Automates the deployment and management of Prometheus instances in Kubernetes. It uses CRDs to define `Prometheus`, `Alertmanager`, and `ServiceMonitor` resources.

2. **PostgreSQL Operator**: Automates the management of PostgreSQL databases, including tasks like creating, scaling, and upgrading PostgreSQL clusters.

**Example of PostgreSQL Custom Resource**:

```yaml
apiVersion: postgresql.crunchydata.com/v1
kind: PostgreSQLCluster
metadata:
  name: example-cluster
spec:
  replicas: 3
  image: "crunchydata/crunchy-postgres:centos7-12.4-4.3.0"
```

The PostgreSQL Operator watches for `PostgreSQLCluster` resources and ensures the PostgreSQL deployment matches the specification (e.g., scaling the number of replicas to 3).

### Conclusion

**Helm** is a powerful tool for managing Kubernetes applications via reusable charts and templates. It simplifies deployments by abstracting configurations and managing them in a versioned format. With **Helm 3**, users no longer need to worry about Tiller, and the security and usability of the tool have greatly improved.

**Operators**, on the other hand, extend Kubernetes’ capabilities to manage stateful applications by automating operational tasks like scaling, backups, and upgrades. Operators use **Custom Resource Definitions (CRDs)** to define new resource types and manage them through custom controllers. The **Operator SDK** provides an easy-to-use framework for building custom operators.

These tools work hand-in-hand to help streamline application management in Kubernetes, providing both automation and scalability for complex workloads.
