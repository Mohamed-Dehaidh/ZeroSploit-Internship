# Task03-K8s

## Overview
This project deploys DFIR-IRIS, an open-source platform designed for Digital Forensics and Incident Response (DFIR). IRIS runs in Docker containers and is managed via Kubernetes.

## Prerequisites
Ensure you have the following installed:
- Kubernetes Cluster
- Kubectl
- nginx ingress controller
- "rook-cephfs" storage class

## Deployment Steps

### Step 1: Prepare the Working Space
Create the necessary directories:
```sh
mkdir -p ~/zerosploit/iris-app ~/zerosploit/psql ~/zerosploit/rabbitmq ~/zerosploit/worker
```
Create configuration files:
```sh
touch ~/zerosploit/iris-app/{app-deployment.yaml,app-svc.yaml,ingress.yaml,app-secrets.yaml,app-configmap.yaml}
touch ~/zerosploit/psql/{psql-statefulset.yaml,psql-headless.yaml,psql-secrets.yaml}
touch ~/zerosploit/rabbitmq/{rabbitmq-deployment.yaml,rabbitmq-svc.yaml}
touch ~/zerosploit/worker/{worker-deploymant.yaml,worker-svc.yaml,worker-secrets.yaml,worker-configmap.yaml}
touch ~/zerosploit/pvc.yaml
```

### Step 2: Deploy IRIS App
#### Create `app-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iris-app-deployment
  labels:
    app: iris-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: iris-app
  template:
    metadata:
      labels:
        app: iris-app
    spec:
      containers:
      - name: iris-app
        image: ghcr.io/dfir-iris/iriswebapp_app:latest
        ports:
        - containerPort: 8000
```
Apply the deployment:
```sh
kubectl apply -f ~/zerosploit/iris-app/app-deployment.yaml
```

#### Create `app-svc.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: iris-app-service
spec:
  selector:
    app: iris-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
  type: ClusterIP
```
Apply the service:
```sh
kubectl apply -f ~/zerosploit/iris-app/app-svc.yaml
```

### Step 3: Configure Ingress
#### Create `ingress.yaml`
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: iris-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: iris.mohamedhassan.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: iris-app-service
            port:
              number: 80
```
Apply ingress:
```sh
kubectl apply -f ~/zerosploit/iris-app/ingress.yaml
```

### Step 4: Deploy PostgreSQL
#### Create `psql-statefulset.yaml`
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: iris-psql-db-deployment
spec:
  serviceName: iris-psql-service
  replicas: 1
  selector:
    matchLabels:
      app: iris-psql
  template:
    metadata:
      labels:
        app: iris-psql
    spec:
      containers:
      - name: iris-psql-db
        image: ghcr.io/dfir-iris/iriswebapp_db:latest
        ports:
        - containerPort: 5432
```
Apply PostgreSQL:
```sh
kubectl apply -f ~/zerosploit/psql/psql-statefulset.yaml
```

### Step 5: Deploy RabbitMQ
#### Create `rabbitmq-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iris-rabbitmq-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: iris-rabbitmq
  template:
    metadata:
      labels:
        app: iris-rabbitmq
    spec:
      containers:
      - name: iris-rabbitmq
        image: rabbitmq:3-management-alpine
        ports:
        - containerPort: 5672
```
Apply RabbitMQ:
```sh
kubectl apply -f ~/zerosploit/rabbitmq/rabbitmq-deployment.yaml
```

### Step 6: Deploy Worker
#### Create `worker-deploymant.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iris-worker-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: iris-worker
  template:
    metadata:
      labels:
        app: iris-worker
    spec:
      containers:
      - name: iris-worker
        image: ghcr.io/dfir-iris/iriswebapp_app:latest
```
Apply Worker App:
```sh
kubectl apply -f ~/zerosploit/worker/worker-deploymant.yaml
```

### Step 7: Configure Domain
Edit your `/etc/hosts` file:
```sh
sudo vim /etc/hosts
```
Add the following entry:
```sh
198.96.95.206 iris.mohamedhassan.com
```

### Step 8: Test Deployment
Open your browser and visit:
```
http://iris.mohamedhassan.com
```

## Conclusion
This README provides a structured guide to deploying DFIR-IRIS using Kubernetes. The setup ensures a scalable, containerized environment for digital forensics investigations. Modify the configurations as needed to fit your cluster environment.

