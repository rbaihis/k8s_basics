# Understanding DaemonSets in Kubernetes

A **DaemonSet** is a Kubernetes resource that ensures a copy of a specific Pod runs on all (or a subset of) nodes in a cluster. This functionality is particularly useful for deploying applications or services that need to be present on every node, such as monitoring agents, log collectors, or storage daemons.

## Key Characteristics of DaemonSets

1. **Node Coverage**: DaemonSets guarantee that a Pod runs on each node. As nodes are added or removed, the DaemonSet automatically adjusts by deploying or terminating Pods accordingly.

2. **Selective Deployment**: You can configure a DaemonSet to run Pods only on a specific group of nodes using node selectors or node affinity rules. This flexibility allows you to control where your Pods are deployed.

3. **Lifecycle Management**: DaemonSets handle the lifecycle of the Pods they manage, ensuring they are running as expected. If a node goes down or a Pod crashes, the DaemonSet controller automatically creates a new Pod to replace it.

4. **Rolling Updates**: DaemonSets can be updated without downtime. You can modify the DaemonSet definition (e.g., updating the container image), and Kubernetes will roll out the changes gradually to each node.

## Common Use Cases

- **Logging and Monitoring**: Deploying agents like Fluentd, Logstash, or Prometheus Node Exporter across all nodes to collect logs or metrics.

- **Network and Storage Management**: Running network plugins, storage drivers, or other infrastructure services that need to run on every node to manage cluster resources.

- **System Daemons**: Executing system-level processes or scripts that require a presence on each node, such as security agents or backup services.


## Important Fields in the DaemonSet Manifest

- **apiVersion**: Specifies the API version (for DaemonSets, it is `apps/v1`).

- **kind**: Defines the resource type, which is `DaemonSet` in this case.

- **metadata**: Contains the metadata for the DaemonSet, including its name and labels.

- **spec**: Specifies the desired state for the DaemonSet. The key components include:
  - **selector**: Defines how to identify the Pods managed by this DaemonSet.
  - **template**: Contains the Pod template, specifying the containers, volumes, and other configurations that will be created.


## Considerations and Best Practices

- **Resource Management**: Be mindful of resource allocation for Pods running on all nodes. DaemonSets can consume significant resources, especially in large clusters.

- **Impact on Node Scaling**: If nodes are frequently added or removed, the DaemonSet's behavior might affect the cluster's performance and stability.

- **Node Affinity and Taints**: Use node affinity and tolerations effectively to control which nodes the DaemonSet Pods can run on, ensuring they only run where necessary.

- **Health Checks**: Implement readiness and liveness probes in the Pod configuration to ensure that your DaemonSet Pods are healthy and operational.



## How to Create a DaemonSet

Creating a DaemonSet involves defining a YAML manifest that specifies the desired configuration. Here’s a simple example:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: my-daemonset
  labels:
    app: my-app
spec:
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: my-image:latest
        ports:
        - containerPort: 8080
```

## Managing DaemonSets

- **Viewing DaemonSets**: You can check the status of a DaemonSet with the following command:

  ```bash
  kubectl get daemonsets
  ```
  
- **Updating a DaemonSet**: YTo update a DaemonSet, modify the manifest and apply it again using:

  ```bash
  kubectl apply -f my-daemonset.yaml

  ```
  
- **Deleting a DaemonSet**: If you no longer need a DaemonSet, you can delete it with:

  ```bash
  kubectl delete daemonset my-daemonset
  ```
---
---
# Debugging a DaemonSet in Kubernetes
  Debugging a DaemonSet in Kubernetes involves several steps and tools to identify issues related to its deployment and performance. Here’s a comprehensive guide on how to debug DaemonSets effectively:

- **Check DaemonSet Status**: If you no longer need a DaemonSet, you can delete it with:

  ```bash
  kubectl get daemonset <daemonset-name> -n <namespace>
  ```
  This command provides information about the number of desired Pods, current Pods, and Pods that are up and running.

  
- **Deleting a DaemonSet**: Begin by inspecting the status of the DaemonSet itself:

  ```bash
  kubectl delete daemonset my-daemonset
  ```
    
- **Describe the DaemonSet**: Use the describe command to get detailed information about the DaemonSet, including events and potential error messages:

  ```bash
  kubectl describe daemonset <daemonset-name> -n <namespace>
  ```
  Look for:
    - Conditions indicating any issues.
    - Events related to Pod scheduling, creation, or failures.

- **Inspect DaemonSet Pods**: List all Pods created by the DaemonSet:

  ```bash
  kubectl get pods -l app=<label-selector> -n <namespace>
  ```
  Replace <label-selector> with the appropriate label used in your DaemonSet.

- **Check Pod Status and Logs**: Inspect the status of individual Pods. If a Pod is in a crash loop or is not running, check its logs:

  ```bash
  kubectl logs <pod-name> -n <namespace>
  ```
  For Pods with multiple containers, specify the container name:
  ```bash
  kubectl logs <pod-name> -c <container-name> -n <namespace>
  ```
  
- **Use kubectl exec to Access Pods**: If you need to troubleshoot further inside the running Pod, use kubectl exec to access its shell:

  ```bash
  kubectl exec -it <pod-name> -n <namespace> -- /bin/sh

  ```
  From within the Pod, you can check configurations, run commands, and investigate issues.
  
- **Examine Events and Resource Quotas**: 
Check for any resource constraints or limits set on the namespace that might affect Pod scheduling:

  ```bash
  kubectl get events -n <namespace>
  kubectl describe resourcequota -n <namespace>
  ```
  
- **Node Conditions and Status**: Sometimes issues can stem from the nodes themselves. Inspect the node conditions to ensure they are healthy:
  
  ```bash
  kubectl get nodes
  kubectl describe node <node-name>
  ```
  Look for issues like NotReady status or taints that might prevent Pods from being scheduled.
  
- **Check Taints and Tolerations**: If your DaemonSet Pods are not starting, ensure the Pods have the necessary tolerations to run on tainted nodes. Review the node taints:

  ```bash
  kubectl describe node <node-name> | grep -i taint
  ```
  
- **Validate Pod Configuration**: Make sure the Pod template in the DaemonSet is correctly configured. Check for issues like:
  - Incorrect image name or tag.
  - Missing environment variables or volume mounts.
  - Incorrect command or arguments.
You can fetch the Pod template definition using:
  ```bash
  kubectl get daemonset <daemonset-name> -n <namespace> -o yaml
  ```
  
- **Look for Admission Controller Issues**: Sometimes, Kubernetes Admission Controllers can block Pod creation due to policy violations. Check the cluster events for any admission errors:

  ```bash
  kubectl get events --all-namespaces
  ```
  
- **Review Resource Requests and Limits**: Verify that resource requests and limits specified in the DaemonSet are reasonable. Under-resourced Pods can lead to failures:

  ```bash
  kubectl describe daemonset <daemonset-name> -n <namespace> | grep -A 5 "Resources"
  ```
  
- **Monitor Cluster Resources**: 
  If many DaemonSet Pods are running on the same node, it might cause resource exhaustion. Use metrics server or monitoring tools (like Prometheus, Grafana) to analyze CPU and memory usage.

- **Check for Network Issues**
  If the DaemonSet Pods communicate with other services, network issues could lead to failures. You can check network connectivity using tools like ping, curl, or nc from within the Pods.
  
---
---
# Exmple Node Exporter as DemonSet from A-to-Z (best practice)

## Create a Namespace (Optional but Recommended)
```bash
kubectl create namespace monitoring
```

## Define the Node Exporter DaemonSet
Create the DaemonSet YAML file (e.g., node-exporter-daemonset.yaml) with the following content:
*
*  Key Points of the DaemonSet:
  * hostNetwork: true: Node Exporter uses the host's network to bind directly to port 9100 on each node.
  * Security Context: Runs as a non-root user (65534 is typically nobody), following least privilege principles.
  * ReadOnly Root Filesystem: The filesystem is mounted as read-only, enhancing security.
  * Resources: Specifies resource requests and limits to avoid resource contention.
*

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
  labels:
    app: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet
      containers:
      - name: node-exporter
        image: prom/node-exporter:v1.6.1 # Use a specific stable version for production
        ports:
        - name: metrics
          containerPort: 9100
          hostPort: 9100
          protocol: TCP
        resources:                   # Resource requests and limits
          requests:
            memory: "30Mi"
            cpu: "100m"
          limits:
            memory: "150Mi"
            cpu: "200m"
        args:
          - --path.procfs=/proc
          - --path.sysfs=/sys
          - --web.listen-address=0.0.0.0:9100
          - --collector.filesystem.ignored-mount-points
          - "^/(dev|proc|sys|var/lib/docker/.+)($|/)"
        securityContext:             # Security best practices
          runAsUser: 65534           # Run as a non-root user (nobody)
          runAsGroup: 65534
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
        volumeMounts:
        - name: proc
          mountPath: /host/proc
          readOnly: true
        - name: sys
          mountPath: /host/sys
          readOnly: true
        - name: rootfs
          mountPath: /rootfs
          readOnly: true
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
      - name: rootfs
        hostPath:
          path: /
  updateStrategy:                # Strategy for rolling updates
    type: RollingUpdate

```

## Service for Node Exporter
To expose Node Exporter metrics to Prometheus, create a service that allows Prometheus to scrape the /metrics endpoint from all the nodes.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: node-exporter
  namespace: monitoring
  labels:
    app: node-exporter
spec:
  ports:
  - name: metrics
    port: 9100
    targetPort: 9100
    protocol: TCP
  selector:
    app: node-exporter
  type: ClusterIP

```

## ServiceMonitor for Prometheus Operator
If you're using the Prometheus Operator, you can create a ServiceMonitor resource so Prometheus can automatically detect and scrape Node Exporter Pods:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: node-exporter
  namespace: monitoring
  labels:
    app: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  endpoints:
  - port: metrics
    interval: 30s
    scrapeTimeout: 10s
  namespaceSelector:
    matchNames:
    - monitoring
```

## Deploy Node Exporter DaemonSet
Now, deploy the DaemonSet and the accompanying Service:
```bash
kubectl apply -f node-exporter-daemonset.yaml
kubectl apply -f node-exporter-service. yaml
kubectl apply -f node-exporter-servicemonitor.yaml  # If using Prometheus Operator
```

## Verify the Deployment
*  Check the status of the DaemonSet to ensure that it is running on all nodes:
```bash
kubectl get daemonset node-exporter -n monitoring
```
You should see the number of desired, current, and ready Pods matching the number of nodes in your cluster.

*  Check the Pods created by the DaemonSet:

```bash
kubectl get pods -n monitoring -l app=node-exporter
```

*  Verify that the Node Exporter is accessible:
```bash
kubectl port-forward <node-exporter-pod> 9100:9100 -n monitoring
```

## Lastly on Prometheus server
If you're not using Prometheus Operator, you’ll need to update your Prometheus configuration manually. Add the following scrape configuration to your `prometheus.yml` file:
```yaml
scrape_configs:
  - job_name: 'node-exporter'
    kubernetes_sd_configs:
      - role: endpoints
    relabel_configs:
      - source_labels: [__meta_kubernetes_service_label_app]
        action: keep
        regex: node-exporter
    scrape_interval: 30s
```
