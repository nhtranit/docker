
# Kubernetes Deployment and Ingress Setup Guide

## Table of Contents

1. [Deploy an Image](#deploy-an-image)
2. [Configure Ingress for Two Services](#configure-ingress-for-two-services)
3. [Common CLI Commands](#common-cli-commands)

---

## 1. Deploy an Image

### 1.1 Create a Deployment

To deploy an image in Kubernetes, define a `Deployment` YAML file. Here’s an example for deploying an image:

**deployment.yaml:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-image:latest
        ports:
        - containerPort: 80
```

Apply the deployment:

```bash
kubectl apply -f deployment.yaml
```

### 1.2 Expose the Deployment with a Service

Create a `Service` to expose the deployment:

**service.yaml:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    nodePort: 30007 # range 30000-32767
  type: NodePort
```

Apply the service:

```bash
kubectl apply -f service.yaml
```

## 2. Configure Ingress for Two Services

### Install an Ingress Controller

Ensure you have an ingress controller installed. For example, you can install the NGINX Ingress

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

### Verify the Installation

Check that the NGINX Ingress Controller pods are running:

```
kubectl get pods -n ingress-nginx
```

Ensure that the service for the Ingress Controller is created and exposed:

```
kubectl get services -n ingress-nginx
```

### 2.1 Create Services for Multiple Deployments

Ensure you have at least two services for different deployments. Here’s an example:

**service1.yaml:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service-1
spec:
  selector:
    app: my-app-1
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

**service2.yaml:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service-2
spec:
  selector:
    app: my-app-2
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

Apply both services:

```bash
kubectl apply -f service1.yaml
kubectl apply -f service2.yaml
```

### 2.2 Create an Ingress Resource

Define an Ingress resource to route traffic to your services:

**ingress.yaml:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: localhost
    http:
      paths:
      - path: /service1
        pathType: Prefix
        backend:
          service:
            name: my-app-service-1
            port:
              number: 80
      - path: /service2
        pathType: Prefix
        backend:
          service:
            name: my-app-service-2
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service-1
            port:
              number: 80
```

Apply the Ingress resource:

```bash
kubectl apply -f ingress.yaml
```

## 3. Common CLI Commands

### 3.1 Check Deployment Status

To see the status of your deployments:

```bash
kubectl get deployments
kubectl describe deployment my-app
```

### 3.2 Check Service Status

To see the status of your services:

```bash
kubectl get services
kubectl describe service my-app-service-1
kubectl describe service my-app-service-2
```

### 3.3 Check Ingress Status

To see the status of your Ingress resource:

```bash
kubectl get ingress
kubectl describe ingress my-app-ingress
```

### 3.4 View Logs

To view logs of a specific pod:

```bash
kubectl logs <pod-name>
```

### 3.5 List All Pods

To list all pods in the namespace:

```bash
kubectl get pods
```

---

For more details on Kubernetes configurations and commands, refer to the [Kubernetes documentation](https://kubernetes.io/docs/home/).
