
# Kubernetes Services: A Compact Guide

Kubernetes (K8s) **Service** is an abstraction that defines a logical set of pods and a policy by which they can be accessed. It decouples pod lifecycle from how they're accessed by clients inside and outside the cluster.

## Types of Kubernetes Services

### 1. ClusterIP (Default):
- Exposes the service on a cluster-internal IP.
- Pods within the cluster can communicate via the service.
- **Use case**: Internal communication between microservices.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

### 2. NodePort:
- Exposes the service on each node's IP at a static port (the NodePort).
- Allows external traffic via `<NodeIP>:<NodePort>`.
- **Use case**: External access without cloud integration.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30007  # Custom NodePort (range: 30000-32767)
```

### 3. LoadBalancer:
- Exposes the service externally using a cloud provider's load balancer.
- **Use case**: Services needing external load balancing in cloud environments (AWS, GCP, Azure).

**Dependencies**: LoadBalancer requires a cloud provider that supports it (e.g., AWS, GCP).

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

**For GCP example** (use `externalTrafficPolicy` for client IP preservation):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer-service
spec:
  type: LoadBalancer
  externalTrafficPolicy: Local
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

### 4. ExternalName:
- Maps the service to an external DNS name (CNAME record).
- **Use case**: When services outside the cluster need to be accessed.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-dns-service
spec:
  type: ExternalName
  externalName: example.com
```

---

## Headless Service (Example with StatefulSet)

A **Headless Service** allows direct access to individual StatefulSet pods. It is crucial for stateful applications requiring stable network identities (e.g., databases).

### 1. Headless Service Definition (No `clusterIP`):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-headless-service
spec:
  clusterIP: None  # No Cluster IP is assigned
  selector:
    app: my-stateful-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

### 2. StatefulSet Definition (With DNS-based pod access):
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-statefulset
spec:
  serviceName: "my-headless-service"  # Reference to the headless service
  replicas: 3
  selector:
    matchLabels:
      app: my-stateful-app
  template:
    metadata:
      labels:
        app: my-stateful-app
    spec:
      containers:
      - name: my-container
        image: my-image
        ports:
        - containerPort: 8080
  volumeClaimTemplates:
  - metadata:
      name: my-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

### 3. Accessing Individual Pods:
- Pods will have stable DNS names like `my-statefulset-0.my-headless-service.namespace.svc.cluster.local`, allowing applications to address each pod directly.

---

## LoadBalancer Service (Cloud Provider Example with Dependencies)

LoadBalancer services are supported by cloud providers. For example, on AWS or GCP, an external load balancer is automatically provisioned.

### Example on AWS:

#### 1. Service Definition:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

#### 2. AWS Configuration (Pre-requisites):
- Ensure the cluster is configured with the correct **IAM roles** for load balancer provisioning.
- You might need to define annotations for specific load balancer types (e.g., NLB for AWS):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-aws-loadbalancer-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"  # Use Network Load Balancer
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

**GCP** and **Azure** provide similar integrations where the cloud platform provisions the external load balancer. Ensure appropriate permissions and provider-specific annotations are in place.

---

## Conclusion

Kubernetes Services abstract pod networking and external access, offering multiple types depending on use cases. For StatefulSets, **Headless Services** provide stable networking, while **LoadBalancer Services** offer external access in cloud environments, often with provider-specific dependencies like IAM roles or annotations for specialized load balancer types.
