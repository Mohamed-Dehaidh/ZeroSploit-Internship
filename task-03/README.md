# Task01-K8s: Kubernetes Deployment, Service, and Ingress

This task demonstrates how to create a Kubernetes deployment, service, and ingress to expose a simple "Hello World" application.

## Overview

1. **Create a Deployment** using a \`crccheck/hello-world\` image.
2. **Create a Service** to expose the deployment internally.
3. **Create an Ingress** to route external traffic to the service using a custom domain.

## Prerequisites

- Kubernetes cluster (e.g., Minikube, GKE, EKS, etc.)
- \`kubectl\` installed and configured
- A domain name or local DNS setup (for testing purposes)

## Steps

### 1. Prepare the Working Space

Create a directory for your project and the necessary YAML files:

\`\`\`bash
mkdir ~/zerosploit
cd ~/zerosploit
\`\`\`

Create the following files:
- \`deployment.yaml\`
- \`internal-svc.yaml\`
- \`ingress.yaml\`

### 2. Write the \`deployment.yaml\` File

This file defines a Kubernetes deployment with 3 replicas of the \`crccheck/hello-world\` image.

\`\`\`yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-deployment
  labels:
    app: helloworld
spec:
  replicas: 3
  selector:
    matchLabels:
      app: helloworld
  template:
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - name: hello-pod
        image: crccheck/hello-world
        ports:
        - containerPort: 8000
\`\`\`

### 3. Write the \`internal-svc.yaml\` File

This file defines a ClusterIP service to expose the deployment internally.

\`\`\`yaml
apiVersion: v1
kind: Service
metadata:
  name: internal-service
spec:
  selector:
    app: helloworld
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
\`\`\`

### 4. Write the \`ingress.yaml\` File

This file defines an ingress resource to route external traffic to the service using a custom domain.

\`\`\`yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  namespace: mohamedhassan
spec:
  ingressClassName: nginx
  rules:
  - host: subdomain.mohamedhassan.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: internal-service
            port:
              number: 80
\`\`\`

### 5. Deploy the Architecture

Apply the YAML files to create the deployment, service, and ingress:

\`\`\`bash
kubectl apply -f deployment.yaml
kubectl apply -f internal-svc.yaml
kubectl apply -f ingress.yaml
\`\`\`

Verify that the resources have been created:

\`\`\`bash
kubectl get deployments
kubectl get services
kubectl get ingress
\`\`\`

### 6. Add the Domain to Your Local Hosts File

To test the setup locally, add the domain to your \`/etc/hosts\` file:

\`\`\`bash
sudo vim /etc/hosts
\`\`\`

Add the following line:

\`\`\`
x.x.x.x subdomain.mohamedhassan.com
\`\`\`

### 7. Test the Setup

Open your browser and navigate to \`http://subdomain.mohamedhassan.com\`. You should see the "Hello World" message.

## Conclusion

This task demonstrates how to deploy a simple application on Kubernetes, expose it internally using a service, and route external traffic to it using an ingress resource. You can modify the domain and other configurations as needed for your environment.
