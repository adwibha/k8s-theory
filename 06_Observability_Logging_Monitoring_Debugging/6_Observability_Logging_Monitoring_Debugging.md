## 6. Observability: Logging, Monitoring & Debugging

Effective observability in Kubernetes ensures the reliability, performance, and health of your applications. It involves monitoring, logging, and troubleshooting techniques to detect issues early and respond quickly. Below, we will cover the essentials of logging, monitoring, and debugging Kubernetes clusters.

### 6.1 Logging in Kubernetes

Logs are essential for understanding the behavior of applications and troubleshooting issues. Kubernetes provides several methods for logging, from viewing individual pod logs to aggregating and analyzing logs across the entire cluster.

#### kubectl logs

The `kubectl logs` command allows you to view logs for a specific pod or container. You can use this command to fetch the logs of a running or terminated container.

**Basic Command:**

```bash
kubectl logs <pod-name>
```

- Fetch logs for the main container in the pod.

**For Specific Containers:**

```bash
kubectl logs <pod-name> -c <container-name>
```

- Use the `-c` flag if the pod contains multiple containers and you need to specify which container's logs to retrieve.

**For Previous Container Instances:**

```bash
kubectl logs <pod-name> -c <container-name> --previous
```

- Use `--previous` to view logs from the previous container instance, useful for troubleshooting after container crashes.

#### Centralized Logging

In larger Kubernetes environments, it becomes difficult to manage logs at the individual pod level. Centralized logging solutions collect and store logs from all nodes and pods in a centralized system for analysis and visualization.

- **Fluentd**: Fluentd is a popular open-source log collector and aggregator. It collects logs from Kubernetes nodes and containers and forwards them to centralized logging backends such as Elasticsearch, Kafka, or a cloud service.
- **Loki**: Developed by Grafana Labs, Loki is designed for aggregating logs in a Kubernetes environment. It integrates well with Grafana for log visualization.
- **ELK Stack (Elasticsearch, Logstash, Kibana)**: The ELK stack is widely used for storing, processing, and visualizing logs. It is commonly used to aggregate logs from various services and visualize them using Kibana.

**Example: Fluentd Configuration**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
  namespace: kube-system
data:
  fluentd.conf: |
    <source>
      @type tail
      path /var/log/containers/*.log
      pos_file /var/log/fluentd.pos
      tag kubernetes.*
      format json
    </source>
    <match kubernetes.**>
      @type elasticsearch
      host "elasticsearch-logging.kube-system.svc.cluster.local"
      port 9200
      logstash_format true
    </match>
```

This example sets up Fluentd to collect logs from containers and send them to Elasticsearch.

### 6.2 Monitoring Kubernetes

Monitoring Kubernetes clusters is essential for gaining insights into resource usage, application health, and performance metrics. Kubernetes offers native solutions and integrates well with third-party tools for monitoring.

#### Metrics Server

The **Metrics Server** is a lightweight and scalable solution that collects resource usage metrics from Kubernetes nodes and pods, including CPU and memory usage. It provides data that can be used for scaling decisions (e.g., with Horizontal Pod Autoscaler) or for manual inspection.

To install Metrics Server:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

**Viewing Metrics**:

- To see the CPU and memory usage for nodes:
  ```bash
  kubectl top nodes
  ```
- To see the CPU and memory usage for pods:
  ```bash
  kubectl top pods
  ```

#### Prometheus & Grafana

**Prometheus** is an open-source monitoring and alerting toolkit designed for reliability and scalability. It is commonly used with **Grafana** for visualizing time-series data.

- **Prometheus**: Collects metrics from Kubernetes nodes, pods, and other components, including custom metrics.
- **Grafana**: Used for visualizing the data collected by Prometheus. Grafana provides dashboards and graphs to monitor cluster performance and application metrics.

**Installation via Helm (Prometheus Operator):**

```bash
helm install prometheus prometheus-community/kube-prometheus-stack
```

After installation, Prometheus will start scraping metrics from Kubernetes and exporting them. Grafana will automatically be configured with pre-built dashboards to visualize the collected metrics.

**Accessing Prometheus and Grafana Dashboards:**

- Prometheus can be accessed at `<prometheus-service-url>/`.
- Grafana provides an intuitive UI to visualize metrics at `<grafana-service-url>/`.

**Example Grafana Dashboard**:

- **Cluster Resource Usage**: Visualize CPU, memory, and disk usage across nodes and pods.
- **Pod Metrics**: Track resource usage for individual pods, including metrics like CPU load and memory consumption.

#### Liveness & Readiness Probes

Kubernetes provides **probes** to check the health of pods and ensure that traffic is directed only to healthy pods.

- **Liveness Probe**: Determines if a pod is still running. If the probe fails, Kubernetes will restart the pod.
- **Readiness Probe**: Determines if a pod is ready to receive traffic. If it fails, Kubernetes will stop sending traffic to the pod.

**Example: Liveness & Readiness Probes**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
    - name: my-app-container
      image: my-app:latest
      livenessProbe:
        httpGet:
          path: /healthz
          port: 8080
        initialDelaySeconds: 3
        periodSeconds: 5
      readinessProbe:
        httpGet:
          path: /readiness
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 10
```

In this example:

- The **liveness probe** checks the `/healthz` endpoint on port 8080 to determine if the container is running.
- The **readiness probe** checks the `/readiness` endpoint on port 8080 to determine if the container is ready to serve traffic.

### 6.3 Troubleshooting Kubernetes Issues

Troubleshooting in Kubernetes involves diagnosing issues with pods, containers, nodes, and the overall cluster. Several Kubernetes commands and debugging techniques help investigate and resolve issues.

#### kubectl describe

The `kubectl describe` command provides detailed information about Kubernetes resources, such as pods, nodes, and deployments. It includes information on events, resource limits, and error messages.

**Example: Describing a Pod**

```bash
kubectl describe pod <pod-name>
```

This command will provide detailed information on the pod's state, including events, logs, and error messages.

#### kubectl logs

The `kubectl logs` command is useful for accessing container logs to understand application behavior and issues. It can be used to fetch logs for running or terminated containers.

**Example: Fetch Logs from a Pod**

```bash
kubectl logs <pod-name>
```

#### kubectl exec

Use the `kubectl exec` command to execute commands inside a running pod. This is helpful for debugging and inspecting the pod's filesystem, network, and processes.

**Example: Executing a Command in a Pod**

```bash
kubectl exec -it <pod-name> -- /bin/bash
```

This command opens a shell inside the pod, allowing you to inspect and debug it interactively.

#### Debugging Node Failures & Pod Crashes

1. **Node Failures**: Use `kubectl describe node <node-name>` to investigate node failures. Look for resources like CPU and memory usage, as well as pod eviction events.
2. **Pod Crashes**: If a pod has crashed or is failing, look at the logs using `kubectl logs <pod-name>`. If the pod is in a CrashLoopBackOff state, use `kubectl describe pod <pod-name>` to view the events and possible causes (e.g., out of memory, missing environment variables).

#### Using kubectl debug

Starting an **ephemeral container** is a useful debugging technique. It allows you to run a temporary container within an existing pod for advanced debugging without restarting the pod.

**Example: Debugging with kubectl debug**

```bash
kubectl debug <pod-name> -it --image=busybox
```

This command starts an ephemeral container using the `busybox` image inside the specified pod. You can then troubleshoot the pod interactively without affecting its state.

### Conclusion

Kubernetes observability encompasses **logging**, **monitoring**, and **debugging** to ensure the health and performance of applications. Kubernetes provides tools like `kubectl logs`, centralized logging systems, **Prometheus**, **Grafana**, and health probes to monitor and troubleshoot applications. When issues arise, Kubernetes also offers powerful debugging tools like `kubectl describe`, `kubectl exec`, and ephemeral containers to help resolve problems efficiently.
