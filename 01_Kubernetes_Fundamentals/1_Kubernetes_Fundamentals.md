### 1. Kubernetes Fundamentals

#### 1.1 Introduction to Kubernetes

##### What is Kubernetes?

Kubernetes (often abbreviated as K8s) is an open-source platform that automates the deployment, scaling, and management of containerized applications. It abstracts away the underlying infrastructure, enabling you to manage your applications across a cluster of machines. Kubernetes is often referred to as a container orchestration platform because it ensures that containerized applications are deployed and managed in a reliable and efficient manner.

**Key Features of Kubernetes:**

- **Automated Scaling**: Kubernetes allows dynamic scaling of applications based on resource utilization.
- **Self-Healing**: Kubernetes automatically replaces failed pods to ensure continuous availability.
- **Load Balancing**: Distributes traffic among containers to ensure optimal performance.
- **Storage Orchestration**: Manages persistent storage for containers across nodes.
- **Rolling Updates**: Supports rolling updates and rollbacks, enabling zero-downtime deployments.
- **Multi-Cloud Support**: Kubernetes can run on a variety of environments, including on-premise, cloud (AWS, GCP, Azure), and hybrid cloud.

##### Why Use Kubernetes?

Kubernetes offers a robust solution to problems commonly faced in managing large-scale applications. Below are some reasons why Kubernetes is widely used:

- **Scalability**: It scales applications automatically to handle changes in traffic or demand.
- **High Availability & Fault Tolerance**: Kubernetes provides built-in mechanisms for ensuring the availability and reliability of applications, such as automatic pod replacement when a failure occurs.
- **Load Balancing**: It evenly distributes network traffic to ensure that no pod is overwhelmed, improving application performance and reliability.
- **Zero-Downtime Deployments**: Kubernetes supports rolling updates and rollbacks, ensuring continuous availability during application updates.
- **Multi-cloud and Hybrid-cloud Portability**: It provides a consistent and unified platform for deploying applications across different cloud environments.

##### Benefits of Kubernetes Over Traditional Deployments

| **Feature**             | **Traditional Deployments**                         | **Kubernetes**                                |
| ----------------------- | --------------------------------------------------- | --------------------------------------------- |
| **Scalability**         | Manual, often slow to respond to traffic changes    | Auto-scaling (Horizontal & Vertical)          |
| **High Availability**   | Downtime in case of failure                         | Self-healing, auto-replacement of failed pods |
| **Load Balancing**      | Requires external tools for balancing traffic       | Built-in load balancing (via services)        |
| **Rolling Updates**     | Requires manual intervention                        | Zero-downtime updates and rollbacks           |
| **Resource Management** | Static allocation of resources                      | Dynamic allocation based on demand            |
| **Cloud Portability**   | Tied to a specific cloud provider or infrastructure | Runs on any cloud or on-prem environment      |

##### Kubernetes vs. Docker Swarm, OpenShift, and Nomad

| **Feature**     | **Kubernetes**                  | **Docker Swarm**                 | **OpenShift**                | **Nomad**                    |
| --------------- | ------------------------------- | -------------------------------- | ---------------------------- | ---------------------------- |
| **Complexity**  | High (feature-rich, flexible)   | Low (easy setup, fewer features) | High (enterprise features)   | Medium (lightweight, simple) |
| **Scalability** | High (auto-scaling support)     | Medium (basic scaling)           | High (similar to Kubernetes) | Medium (supports scaling)    |
| **Security**    | Strong (RBAC, network policies) | Weak (basic access control)      | Strong (built-in security)   | Medium (supports ACLs)       |
| **Multi-cloud** | Yes                             | Limited                          | Yes                          | Yes                          |
| **Ease of Use** | Medium (requires learning)      | Easy (CLI-based, quick setup)    | Medium (web UI + CLI)        | Simple API-based management  |
| **Use Case**    | Enterprise-grade apps           | Small-scale deployments          | Enterprise DevOps            | Hybrid workloads             |

**Docker Swarm**: Simpler to set up but lacks advanced features such as auto-healing, scaling, and intricate networking options that Kubernetes offers.

**OpenShift**: An enterprise version of Kubernetes that includes additional security features and a web-based UI, making it a great choice for enterprise environments.

**Nomad**: A lightweight orchestration tool designed to handle both Docker and non-Docker workloads. It is less feature-rich than Kubernetes but may be sufficient for simpler use cases.

#### 1.2 Kubernetes Architecture

Kubernetes architecture consists of two main components: **Master Nodes** (Control Plane) and **Worker Nodes** (Data Plane).

##### Master Node Components

The **Master Node** is responsible for managing the Kubernetes cluster, handling cluster-wide activities such as scheduling, monitoring, and maintaining the desired state of applications. The key components of the Master Node are:

1. **API Server (kube-apiserver)**:

   - The API server is the front-end of the Kubernetes control plane. It serves the Kubernetes API and acts as the gateway for interacting with the cluster.
   - The API server is responsible for validating and processing API requests, including from `kubectl` commands, other components, or external applications.
   - **Command**: To check the status of the API server, use the following command:
     ```bash
     kubectl get componentstatuses
     ```

2. **etcd (Cluster Database)**:

   - **etcd** is a distributed key-value store that holds the configuration data and state of the Kubernetes cluster, including information about nodes, pods, deployments, services, and more.
   - It is highly available and serves as the source of truth for the Kubernetes cluster.
   - **Command**: To check the health of `etcd`, use the following command:
     ```bash
     ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 endpoint health
     ```

3. **Controller Manager (kube-controller-manager)**:

   - The controller manager is responsible for ensuring that the desired state of the cluster matches the current state. It runs several controllers such as the ReplicaSet controller, Node controller, and Job controller.
   - **Command**: To check the running controllers, use the following command:
     ```bash
     kubectl get endpoints
     ```

4. **Scheduler (kube-scheduler)**:
   - The scheduler is responsible for assigning newly created pods to worker nodes based on resource availability and various constraints such as node affinity, taints, and pod priorities.
   - **Command**: To view scheduling events, use the following command:
     ```bash
     kubectl get events --sort-by=.metadata.creationTimestamp
     ```

##### Worker Node Components

The **Worker Nodes** are responsible for running the application workloads. They host the pods and communicate with the Master Node to maintain cluster health and report status.

1. **Kubelet**:

   - The Kubelet is a node agent that ensures containers are running inside the pods on each worker node. It monitors the pod lifecycle and communicates with the API server.
   - **Command**: To check node health, use:
     ```bash
     kubectl get nodes
     ```

2. **Kube Proxy**:

   - Kube Proxy manages network communication and load balancing within the Kubernetes cluster. It ensures that requests to a service are routed to the correct pod by modifying iptables rules or using IPVS.
   - **Command**: To view the logs of Kube Proxy, use:
     ```bash
     kubectl logs -n kube-system kube-proxy-<node-name>
     ```

3. **Container Runtime (Docker, containerd, CRI-O)**:
   - The container runtime is responsible for running containers within the pods. Kubernetes supports several container runtimes, including Docker (deprecated in favor of containerd), containerd, and CRI-O.
   - **Command**: To check which container runtime is being used, run:
     ```bash
     kubectl get nodes -o wide
     ```

##### Diagram: Kubernetes Architecture Overview

![Kubernetes-Architecture](https://kubernetes.io/images/docs/kubernetes-cluster-architecture.svg)

#### 1.3 Installation & Setup

There are several methods for setting up a Kubernetes environment depending on your use case:

1. **Minikube**:

   - Minikube is a tool that runs a single-node Kubernetes cluster locally on your machine, ideal for testing and development.
   - **Command**: To start Minikube, run:
     ```bash
     minikube start
     ```
   - To interact with the cluster using `kubectl`:
     ```bash
     kubectl get nodes
     ```

2. **Kind (Kubernetes in Docker)**:

   - Kind allows you to run Kubernetes clusters in Docker containers, making it a lightweight solution for testing and development.
   - **Command**: To create a Kind cluster, run:
     ```bash
     kind create cluster
     ```

3. **kubeadm**:

   - **kubeadm** is a tool that simplifies the deployment of a multi-node Kubernetes cluster, useful for setting up clusters in production environments.
   - **Command**: Initialize the cluster with:
     ```bash
     kubeadm init
     ```

4. **Cloud Kubernetes Services**:  
   Cloud providers like AWS, GCP, and Azure offer managed Kubernetes services, making it easier to set up production-grade clusters.
   - **Amazon EKS**: Elastic Kubernetes Service for easy setup and management on AWS.
   - **Google GKE**: Google Kubernetes Engine for Kubernetes on Google Cloud.
   - **Azure AKS**: Azure Kubernetes Service for Kubernetes on Azure.
   - **DigitalOcean Kubernetes**: Simplified Kubernetes for DigitalOcean users.
