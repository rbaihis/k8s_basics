
# Kubernetes Ingress Full Documentation with Production Setup

## 1. Overview

Ingress in Kubernetes is an API object that manages external access to services within a cluster, typically HTTP and HTTPS. Ingress may provide load balancing, SSL termination, and name-based virtual hosting.

### Key Benefits:
- **Centralized Traffic Management:** Direct external traffic into your cluster and distribute it across multiple services.
- **SSL Termination:** Offload SSL encryption from individual services to the Ingress Controller.
- **Load Balancing:** Distribute traffic across multiple pods, ensuring high availability.

---

## 2. Ingress Use Cases

- **Routing HTTP/S Traffic:** Ingress routes external traffic to services within the cluster.
- **Multiple Services on One IP:** Route different domains or paths to different services under the same IP address.
- **SSL Termination:** Ingress manages SSL certificates and encryption for services.
  
---

## 3. When to Use Ingress

- When you need **path-based** or **domain-based routing**.
- If you want to expose multiple services under a single IP address.
- To offload **SSL/TLS** certificate management to the Ingress controller.
- For applications that require **custom routing rules**, such as routing based on subdomains or subpaths.

---

## 4. Important Fields in Ingress Resource

### Key fields:
- **`spec.rules`**: Define routing rules.
  ```yaml
  rules:
    - host: example.com
      http:
        paths:
          - path: /app
            backend:
              serviceName: my-service
              servicePort: 80
  ```
  
- **`spec.tls`**: Define SSL/TLS settings.
  ```yaml
  tls:
    - hosts:
        - example.com
      secretName: tls-secret
  ```

- **`spec.backend`**: Define a default backend for unmatched routes.
- **`metadata.annotations`**: Add custom settings, such as timeouts or rewrites.
  ```yaml
  metadata:
    annotations:
      nginx.ingress.kubernetes.io/rewrite-target: /
  ```

---

## 5. Production-Ready Ingress Controller Setup

### Choosing a Production Ingress Controller

For a production environment, **NGINX Ingress Controller** is a common and reliable choice. It is battle-tested, has wide community support, and is capable of handling a variety of routing needs and advanced features such as rate-limiting, retries, and blue-green deployments.

Other production controllers:
- **Traefik**: Easy configuration, supports dynamic routing, has a built-in dashboard.
- **HAProxy**: Powerful and feature-rich, high performance.
- **AWS ALB Ingress Controller**: Best for AWS environments with native integration.

### Prerequisites
1. A running **Kubernetes cluster** (e.g., managed clusters like GKE, EKS, AKS or self-hosted).
2. **kubectl** configured to communicate with your cluster.
3. A **domain name** and access to a DNS provider.
4. Optional: **cert-manager** for automatic SSL certificate provisioning.

### Step-by-Step Setup of NGINX Ingress Controller

1-ViaHelm: **Install NGINX Ingress Controller**:
   ```bash
   helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
   helm repo update
   helm install nginx-ingress ingress-nginx/ingress-nginx      --namespace ingress-nginx --create-namespace      --set controller.replicaCount=2      --set controller.nodeSelector."kubernetes\.io/os"=linux      --set defaultBackend.nodeSelector."kubernetes\.io/os"=linux
   ```
1-ViaManifest: **Install NGINX Ingress Controller**:
   ```bash
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
   ```

2. **Verify the Installation**:
   Ensure the NGINX Ingress Controller is running:
   ```bash
   kubectl get pods -n ingress-nginx
   ```

3. **Create a DNS Record**:
   Get the external IP of the ingress controller and create a DNS `A` record:
   ```bash
   kubectl get svc -n ingress-nginx
   ```
   Example: `api.example.com` → `EXTERNAL-IP`.

4. **Install cert-manager (Optional)**:
   Install cert-manager for automatic SSL certificate management:
   ```bash
   kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.8.0/cert-manager.yaml
   ```

5. **Create Ingress Resource for Services**:
   Example for a microservices application:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: ecommerce-ingress
     annotations:
       nginx.ingress.kubernetes.io/rewrite-target: /
       cert-manager.io/cluster-issuer: letsencrypt-prod
   spec:
     tls:
       - hosts:
           - api.myecommerce.com
         secretName: api-tls-secret
     rules:
       - host: api.myecommerce.com
         http:
           paths:
             - path: /user
               pathType: Prefix
               backend:
                 service:
                   name: user-service
                   port:
                     number: 80
             - path: /product
               pathType: Prefix
               backend:
                 service:
                   name: product-service
                   port:
                     number: 80
             - path: /order
               pathType: Prefix
               backend:
                 service:
                   name: order-service
                   port:
                     number: 80
   ```

6. **Apply the Ingress Configuration**:
   ```bash
   kubectl apply -f ecommerce-ingress.yaml
   ```

7. **Verify DNS and SSL Setup**:
   Ensure the DNS is correctly pointed to the Ingress controller IP and SSL certificates are issued properly:
   ```bash
   kubectl describe ingress ecommerce-ingress
   ```

---

## 6. Best Practices for Production

- **Enable TLS/SSL**: Always secure traffic using TLS, either by manually managing certificates or automating with cert-manager.
- **Use Annotations for Fine-tuning**: Leverage Ingress Controller annotations for optimizing behavior (e.g., timeouts, load balancing strategies).
- **Monitor the Ingress Controller**: Integrate with Prometheus and Grafana for monitoring performance and reliability.
- **Limit Service Exposure**: Avoid exposing all services to the public. Only expose what’s necessary and use internal services for backend communication.
- **Ensure High Availability**: Deploy multiple replicas of the Ingress controller to avoid a single point of failure.

---

## 7. Monitoring and Observability

For monitoring Ingress in production, integrate with tools like:
- **Prometheus**: For collecting metrics.
- **Grafana**: For visualizing metrics from the Ingress controller.
- **Alertmanager**: For triggering alerts when there are issues (e.g., downtime, slow responses).

---

## 8. Conclusion

Using Kubernetes Ingress in production allows you to efficiently manage HTTP/S traffic to your services, handle SSL termination, and enforce routing rules across multiple services. NGINX Ingress Controller is a reliable choice, with built-in support for features like SSL, rate-limiting, and advanced routing configurations. Always monitor and ensure that your ingress resources are secure and well-maintained.
