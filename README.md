````markdown
# Kubernetes Lab with Kind

This lab sets up a local Kubernetes cluster using Kind, deploys Nginx workloads, exposes them with a Service, and demonstrates ConfigMap usage.

## Prerequisites

- macOS with Homebrew
- Install Kind and kubectl:

```bash
brew install kind kubectl
````

## 1. Create Cluster

```bash
kind create cluster --name=k8s-lab
kubectl get nodes
```

## 2. Deploy a Pod

`nginx-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

```bash
kubectl apply -f nginx-pod.yaml
kubectl get pods
```

## 3. Create a Deployment (3 replicas)

`nginx-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

```bash
kubectl apply -f nginx-deployment.yaml
kubectl get deployments
kubectl get pods -l app=nginx
```

Scale:

```bash
kubectl scale deployment nginx-deployment --replicas=5
kubectl get pods -l app=nginx
```

## 4. Expose with a Service

`nginx-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007
```

```bash
kubectl apply -f nginx-service.yaml
kubectl get services
```

Access in browser: `http://localhost:30007`

## 5. ConfigMap Example (Optional)

`nginx-configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  index.html: |
    <h1>Hello from ConfigMap-powered Nginx 🚀</h1>
```

`nginx-pod-config.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-config
spec:
  containers:
  - name: nginx
    image: nginx:latest
    volumeMounts:
    - name: webcontent
      mountPath: /usr/share/nginx/html
  volumes:
  - name: webcontent
    configMap:
      name: nginx-config
```

```bash
kubectl apply -f nginx-configmap.yaml
kubectl apply -f nginx-pod-config.yaml
kubectl port-forward pod/nginx-pod-config 8080:80
```

Visit `http://localhost:8080`

## Cleanup

Delete all resources:

```bash
kubectl delete -f nginx-service.yaml
kubectl delete -f nginx-deployment.yaml
kubectl delete -f nginx-pod.yaml
kubectl delete -f nginx-configmap.yaml
kubectl delete -f nginx-pod-config.yaml
```

Delete cluster:

```bash
kind delete cluster --name=k8s-lab
```

```
```
