# 🚀 Kubernetes Services Lab Guide

## 📌 Introduction (Mentor Explanation)

Great job getting your Deployment running — now comes the real-world challenge.

👉 Pods are **ephemeral**. Their IPs change whenever they restart.
👉 A Deployment creates **multiple Pods**. So which one do you connect to?

This is where **Kubernetes Services** come in.

### 💡 What is a Service?
A Service provides:
- A **stable IP address**
- A **stable DNS name**
- **Load balancing** across multiple Pods

### 🔁 Traffic Flow
Client → Service (Stable Endpoint) → Pods (Dynamic IPs)

This abstraction is critical in production systems.

---

# 🧪 Lab Tasks

---

## ✅ Task 1: Deploy the Application

### 📖 Concept
Before exposing anything, we need an application running inside the cluster.

We will deploy:
- NGINX
- 3 replicas (Pods)

### 📄 Create Deployment

[apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
]

### ▶️ Apply Deployment

[kubectl apply -f app-deployment.yaml]

[kubectl get pods -o wide]

### 🔍 Verification
- Ensure **3 Pods are running**
- Note their **IP addresses**

### 🧠 Mentor Insight
These Pod IPs will change on restart — this is the core problem Services solve.

### 📸 Screenshot Placeholder

<img width="1646" height="952" alt="task`" src="https://github.com/user-attachments/assets/558fd98d-3880-4640-b8d4-d02850d44e3b" />


---

## ✅ Task 2: ClusterIP Service (Internal Access)

### 📖 Concept
- Default Service type
- Accessible **only inside the cluster**
- Provides **stable internal communication**

### 📄 Create Service

[apiVersion: v1
kind: Service
metadata:
  name: web-app-clusterip
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
]

### ▶️ Apply Service

[kubectl apply -f clusterip-service.yaml]

[kubectl get services]

### 🧪 Test Inside Cluster

[kubectl run test-client --image=busybox:latest --rm -it --restart=Never -- sh]

[wget -qO- http://web-app-clusterip]

[exit]

### 🔍 Verification
- You should see **NGINX welcome page**
- Repeat command → traffic hits different Pods

### 🧠 Mentor Insight
Service is doing **load balancing automatically** across Pods.

### 📸 Screenshot Placeholder

<img width="1613" height="972" alt="task2" src="https://github.com/user-attachments/assets/bac359a1-8c7b-4a3b-96f6-ea2f333aa84c" />


---

## ✅ Task 3: Service Discovery using DNS

### 📖 Concept
Kubernetes has built-in DNS:
- Short name → same namespace
- Full name → cross-namespace

Format:
service-name.namespace.svc.cluster.local

### 🧪 Test DNS

[kubectl run dns-test --image=busybox:latest --rm -it --restart=Never -- sh]

[wget -qO- http://web-app-clusterip]

[wget -qO- http://web-app-clusterip.default.svc.cluster.local]

[nslookup web-app-clusterip]

[exit]

### 🔍 Verification
- DNS resolves to **ClusterIP**
- Matches kubectl get services output

### 🧠 Mentor Insight
In real systems, services talk using DNS — not IPs.

---

## ✅ Task 4: NodePort Service (External Access)

### 📖 Concept
- Exposes app outside cluster
- Uses Node IP + Port
- Range: 30000–32767

### 📄 Create Service

[apiVersion: v1
kind: Service
metadata:
  name: web-app-nodeport
spec:
  type: NodePort
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
]

### ▶️ Apply Service

[kubectl apply -f nodeport-service.yaml]

[kubectl get services]

### 🌐 Access Application

For Docker Desktop:
[curl http://localhost:30080]

For Kind:
[kubectl get nodes -o wide]

Then:
[curl http://<node-ip>:30080]

### 🔍 Verification
- NGINX page should load in browser/terminal

### 🧠 Mentor Insight
NodePort is useful for:
- Testing
- Development
- Not ideal for production

---

## ✅ Task 5: LoadBalancer Service

### 📖 Concept
- Used in cloud (AWS/GCP/Azure)
- Creates external load balancer
- Routes traffic to nodes → pods

### 📄 Create Service

[apiVersion: v1
kind: Service
metadata:
  name: web-app-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
]

### ▶️ Apply Service

[kubectl apply -f loadbalancer-service.yaml]

[kubectl get services]

### 🔍 Verification
- Local cluster → EXTERNAL-IP = pending
- This is expected

### 🧠 Mentor Insight
Why pending?
👉 No cloud provider to provision load balancer

For Minikube:
[minikube tunnel]

---

## ✅ Task 6: Compare All Services

### ▶️ Check Services

[kubectl get services -o wide]

### 📊 Comparison

ClusterIP:
- Internal only
- Used for microservices communication

NodePort:
- External via Node IP
- Good for testing

LoadBalancer:
- External via cloud
- Production-ready

### 🔍 Deep Inspection

[kubectl describe service web-app-loadbalancer]

### 🔍 Verification
- Has ClusterIP ✔
- Has NodePort ✔
- Has LoadBalancer config ✔

### 🧠 Mentor Insight
Hierarchy:
LoadBalancer → NodePort → ClusterIP

---

## ✅ Task 7: Endpoints (Important Concept)

### 📖 Concept
Endpoints = actual Pod IPs behind Service

### ▶️ Check Endpoints

[kubectl get endpoints web-app-clusterip]

### 🧠 Mentor Insight
If endpoints are empty → Service is broken (selector mismatch)

---

## 🧹 Task 8: Cleanup

[kubectl delete -f app-deployment.yaml]

[kubectl delete -f clusterip-service.yaml]

[kubectl delete -f nodeport-service.yaml]

[kubectl delete -f loadbalancer-service.yaml]

[kubectl get pods]

[kubectl get services]

### 🔍 Verification
Only default kubernetes service should remain

---

# 📝 Documentation (day-53-services.md)

## What Problem Services Solve
- Pods have dynamic IPs
- Deployments create multiple Pods
- Services provide stable access + load balancing

## Service Types

### ClusterIP
- Internal communication

### NodePort
- External access via node

### LoadBalancer
- Production external access

## DNS in Kubernetes
- Automatic service discovery
- Uses service-name.namespace.svc.cluster.local

## Endpoints
- Actual Pod IPs behind Service
- Debug using:
[kubectl get endpoints <service-name>]

---

# 📸 Final Screenshot

<img width="1137" height="905" alt="task7" src="https://github.com/user-attachments/assets/0ad07927-244c-4092-ad04-52a1ee6f8246" />


---

# 🎯 Final Mentor Notes

- Always ensure **selector matches labels**
- Use **ClusterIP for internal apps**
- Use **NodePort for testing**
- Use **LoadBalancer in production**
- Debug using:
  - kubectl describe service
  - kubectl get endpoints

---

# 🚀 You Just Learned

- Stable networking in Kubernetes
- Service abstraction
- Load balancing basics
- Internal vs external exposure

This is a **core DevOps + Kubernetes concept** — you’ll use this everywhere.

Keep going 🔥
