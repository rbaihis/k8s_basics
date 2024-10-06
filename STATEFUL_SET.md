# Understanding StatefulSet in Kubernetes

StatefulSets are a powerful feature in Kubernetes designed to manage stateful applications. They provide a stable identity for each of the pods they manage and help maintain the state of the application across various scaling and deployment scenarios.

## Key Characteristics

- **Stable Network Identity**: Each pod in a StatefulSet gets a unique, stable hostname that persists through rescheduling and scaling.
- **Stable Storage**: StatefulSets can be configured with PersistentVolumeClaims (PVCs) that are automatically created and managed for each pod, ensuring data durability.
- **Ordered Deployment and Scaling**: Pods in a StatefulSet are deployed and scaled in a specific order, allowing for controlled updates and scaling operations.
- **Pod Management Policy**: StatefulSets support two policies—`OrderedReady` (default) and `Parallel`. `OrderedReady` ensures that pods are started in order, while `Parallel` allows for simultaneous pod creation.

## Common Use Cases for StatefulSet

- **Databases**: StatefulSets are ideal for deploying databases that require stable identities and persistent storage, such as MongoDB, Cassandra, and MySQL.
- **Distributed Applications**: Applications like Kafka or Zookeeper, which require specific network identities and state management, benefit from StatefulSets.
- **Message Queues**: Systems that need to maintain order and state, such as RabbitMQ, also utilize StatefulSets.

## Important Fields in StatefulSet

- **spec.serviceName**: The name of the service that governs the StatefulSet, enabling stable network identities.
- **spec.template**: The pod template used for creating the pods, including specifications like container images and resource requests.
- **spec.updateStrategy**: Defines how updates to the StatefulSet are performed (e.g., `RollingUpdate` or `OnDelete`).
- **spec.volumeClaimTemplates**: A template for creating persistent volume claims for each pod in the StatefulSet.

## Considerations and Best Practices in StatefulSet

- **Pod Disruption Budgets**: Define Pod Disruption Budgets (PDBs) to ensure that a minimum number of pods remain available during voluntary disruptions.
- **Storage Management**: Use storage classes that support dynamic provisioning to handle storage needs effectively.
- **Resource Requests and Limits**: Always set resource requests and limits for better resource management.
- **Use Readiness Probes**: Implement readiness probes to manage traffic only to pods that are fully initialized and ready to accept requests.

## How to Create a StatefulSet

Here's an example YAML file to create a basic StatefulSet:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-statefulset
spec:
  serviceName: "my-service"
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
      - name: my-container
        image: my-image:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: my-storage
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: my-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

## Managing StatefulSet

- **Scaling**: Use the command `kubectl scale statefulset my-statefulset --replicas=5` to scale the StatefulSet.
- **Updating**: To update a StatefulSet, modify the pod template in the StatefulSet definition and apply the changes.
- **Deleting**: Use `kubectl delete statefulset my-statefulset` to delete a StatefulSet. Pods will be deleted in reverse order.

## Debugging StatefulSet

- **Pod Status**: Use `kubectl get pods -l app=my-app` to check the status of the pods and identify any issues.
- **Logs**: Access pod logs using `kubectl logs my-statefulset-0` to view logs of a specific pod.
- **Describe**: Use `kubectl describe statefulset my-statefulset` for detailed information about the StatefulSet, including events and conditions.
- **Events**: Check events related to the StatefulSet with `kubectl get events --sort-by='.metadata.creationTimestamp'`.


---
___

# Example PostgreSQL StatefulSet Example

 example of a StatefulSet for deploying PostgreSQL in Kubernetes, where environment variables are sourced from a ConfigMap. This setup allows you to manage your PostgreSQL configuration separately from the application deployment.


## Step 2: Create a Secret
  as best practice, store sensitive data such as passwords in a Secret instead of a ConfigMap.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: pg-secret
type: Opaque
data:
  POSTGRES_PASSWORD: bXlwYXNzd29yZA==  # Base64 encoded password (example: mypassword)
```

## Step 3: Create a ConfigMap

First, create a ConfigMap to store your PostgreSQL environment variables. You can name it `pg-config` and include postgres envVars.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: pg-config
data:
  POSTGRES_DB: mydatabase
  POSTGRES_USER: myuser
  #POSTGRES_PASSWORD: mypassword
```


## Step 3: Create a ConfigMap

First, create a ConfigMap to store your PostgreSQL environment variables. You can name it `pg-config` and include postgress envVars.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: pg-config
data:
  POSTGRES_DB: mydatabase
  POSTGRES_USER: myuser
  POSTGRES_PASSWORD: mypassword
```
## Step 4:  Create a StatefulSet for PostgreSQL
Now, create a StatefulSet that uses the above ConfigMap for environment variables. (add secret for password as best practice later):

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: "postgres"
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:latest
        ports:
        - containerPort: 5432
        envFrom:
        - configMapRef:
            name: pg-config
        - secretRef:
            name: pg-secret
        volumeMounts:
        - name: pg-data
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: pg-data
        persistentVolumeClaim:
          claimName: pg-data-pvc
  volumeClaimTemplates:
  - metadata:
      name: pg-data-pvc
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

## Step 6: Create a Headless Service
  Create a headless service to enable stable network identities for the PostgreSQL pods.
  
```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  clusterIP: None
  ports:
  - port: 5432
    targetPort: 5432
  selector:
    app: postgres
```

## Step 7: Deploy the Resources
  You can deploy the ConfigMap, StatefulSet, and Service using the following commands:

```bash
kubectl apply -f pg-config.yaml
kubectl apply -f postgres-statefulset.yaml
kubectl apply -f postgres-service.yaml
```

## Step 8: Accessing PostgreSQL
  Once your StatefulSet is up and running, you can connect to PostgreSQL using the following command (replace <pod-name> with one of your pod names):

```yaml
kubectl exec -it <pod-name> -- psql -U myuser -d mydatabase
```

## Summary
  This example demonstrates a basic setup of PostgreSQL using a StatefulSet in Kubernetes with environment variables sourced from a ConfigMap. You can extend this setup by adding further configurations, such as resource limits, health checks, or persistent storage configurations, depending on your production needs.
