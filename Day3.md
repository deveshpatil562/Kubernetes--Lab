# 🚀 Day 52 – Kubernetes Namespaces and Deployments

## 📌 Overview

Yesterday, you created standalone Pods — but they had a major limitation:
👉 If a Pod dies, it is gone forever.

Today, we fix that using:
- **Namespaces** → Logical isolation inside a cluster
- **Deployments** → Self-healing, scalable application management

This is how real-world Kubernetes workloads are run.

---

# 🧠 Concepts First (Understand Like a Pro)

## 🔹 What is a Namespace?

A **Namespace** is a virtual cluster inside Kubernetes.

It helps you:
- Organize resources (dev, staging, production)
- Avoid naming conflicts
- Apply access control and quotas

👉 Think of it like folders inside a system.

---

## 🔹 Default Namespaces

Kubernetes comes with built-in namespaces:

- **default** → Where resources go if not specified
- **kube-system** → Internal Kubernetes components (DO NOT TOUCH)
- **kube-public** → Public resources
- **kube-node-lease** → Node heartbeat tracking

---

## 🔹 What is a Deployment?

A **Deployment** ensures:
- Desired number of Pods always running
- Automatic recovery if Pods fail
- Easy scaling
- Zero-downtime updates

👉 Deployment = Desired State Manager

---

## 🔹 Deployment vs Pod

| Feature | Pod | Deployment |
|--------|-----|-----------|
| Self-healing | ❌ | ✅ |
| Scaling | ❌ | ✅ |
| Rolling updates | ❌ | ✅ |
| Recommended for production | ❌ | ✅ |

---

# 🛠️ LAB TASKS

---

# ✅ Task 1: Explore Default Namespaces

## 🔧 Command

[ kubectl get namespaces ]

## 🔍 Check Pods in kube-system

[ kubectl get pods -n kube-system ]

## ✅ Verification

- You should see multiple system pods running
- These are cluster-critical components

👉 Mentor Tip:
Never modify anything inside `kube-system` unless you know exactly what you're doing.

📸 Screenshot Placeholder:

<img width="885" height="532" alt="task1" src="https://github.com/user-attachments/assets/38b39bf2-38a9-4346-ae05-f43b2680bbf7" />


---

# ✅ Task 2: Create and Use Custom Namespaces

## 🔧 Create Namespaces

[ kubectl create namespace dev ]
[ kubectl create namespace staging ]

## 🔍 Verify

[ kubectl get namespaces ]

---

## 🔧 Create Namespace via YAML

[ 
apiVersion: v1
kind: Namespace
metadata:
  name: production
]

[ kubectl apply -f namespace.yaml ]

---

## 🔧 Run Pods in Namespaces

[ kubectl run nginx-dev --image=nginx:latest -n dev ]
[ kubectl run nginx-staging --image=nginx:latest -n staging ]

---

## 🔍 List Pods

[ kubectl get pods ]  
👉 Only default namespace

[ kubectl get pods -A ]  
👉 All namespaces

---

## ✅ Verification

- `kubectl get pods` → ❌ Does NOT show dev/staging pods
- `kubectl get pods -A` → ✅ Shows all pods

📸 Screenshot Placeholder:

<img width="885" height="885" alt="task2" src="https://github.com/user-attachments/assets/120df7f7-f84c-4a05-8fbf-531abc7e7248" />


---

# ✅ Task 3: Create Your First Deployment

## 🔧 Deployment YAML

[ 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
  labels:
    app: nginx
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
        image: nginx:1.24
        ports:
        - containerPort: 80
]

---

## 🔧 Apply Deployment

[ kubectl apply -f nginx-deployment.yaml ]

---

## 🔍 Verify

[ kubectl get deployments -n dev ]
[ kubectl get pods -n dev ]

---

## 🧠 Mentor Explanation

- **READY** → Running pods / desired pods
- **UP-TO-DATE** → Pods matching latest config
- **AVAILABLE** → Pods ready to serve traffic

📸 Screenshot Placeholder:

<img width="1462" height="925" alt="task3 4" src="https://github.com/user-attachments/assets/481ef045-df76-4748-8d25-d8bc5f15ab9d" />


---

# ✅ Task 4: Self-Healing Test

## 🔧 Delete a Pod

[ kubectl get pods -n dev ]

[ kubectl delete pod <pod-name> -n dev ]

[ kubectl get pods -n dev ]

---

## ✅ Verification

- A new pod is automatically created
- Pod name will be DIFFERENT

👉 Mentor Insight:
Deployment maintains the desired state (replicas = 3)

---

# ✅ Task 5: Scale the Deployment

## 🔧 Scale Up

[ kubectl scale deployment nginx-deployment --replicas=5 -n dev ]

[ kubectl get pods -n dev ]

---

## 🔧 Scale Down

[ kubectl scale deployment nginx-deployment --replicas=2 -n dev ]

[ kubectl get pods -n dev ]

---

## 🧠 Mentor Explanation

- Scaling up → New Pods created
- Scaling down → Extra Pods TERMINATED

👉 Kubernetes ensures desired state is always maintained.

---

## 🔧 Declarative Scaling

Edit YAML:

[ replicas: 4 ]

Then apply:

[ kubectl apply -f nginx-deployment.yaml ]

---

## ✅ Verification

- Pods increase/decrease automatically

---

# ✅ Task 6: Rolling Update & Rollback

## 🔧 Update Image

[ kubectl set image deployment/nginx-deployment nginx=nginx:1.25 -n dev ]

---

## 🔍 Watch Rollout

[ kubectl rollout status deployment/nginx-deployment -n dev ]

---

## 🔍 History

[ kubectl rollout history deployment/nginx-deployment -n dev ]

---

## 🔧 Rollback

[ kubectl rollout undo deployment/nginx-deployment -n dev ]

[ kubectl rollout status deployment/nginx-deployment -n dev ]

---

## 🔍 Verify Image

[ kubectl describe deployment nginx-deployment -n dev | grep Image ]

---

## 🧠 Mentor Explanation

- Rolling update = zero downtime deployment
- Pods updated one-by-one
- Rollback restores previous stable version

👉 Final Answer:
Image after rollback → **nginx:1.24**

---

# ✅ Task 7: Clean Up

## 🔧 Delete Resources

[ kubectl delete deployment nginx-deployment -n dev ]

[ kubectl delete pod nginx-dev -n dev ]
[ kubectl delete pod nginx-staging -n staging ]

[ kubectl delete namespace dev staging production ]

---

## 🔍 Verify Cleanup

[ kubectl get namespaces ]
[ kubectl get pods -A ]

---

## ✅ Verification

- All custom namespaces removed
- No pods left

👉 Mentor Warning:
Deleting a namespace deletes EVERYTHING inside it.

---

# 📚 Documentation Section (For Your GitHub Repo)

## 🔹 What are Namespaces?

Namespaces are used to logically isolate resources within a Kubernetes cluster. They help in organizing workloads like dev, staging, and production environments.

---

## 🔹 Deployment Breakdown

- metadata → Name and namespace
- replicas → Desired number of pods
- selector → Matches pods
- template → Blueprint for pods
- containers → Application details

---

## 🔹 Pod vs Deployment Behavior

- Pod → Dies permanently
- Deployment → Recreates automatically

---

## 🔹 Scaling

- Imperative → kubectl scale
- Declarative → Modify YAML

---

## 🔹 Rolling Updates

- Updates pods gradually
- No downtime
- Ensures availability

---

## 🔹 Rollback

- Reverts to previous version
- Useful in failed deployments

---

# 📸 Final Screenshots Required

<img width="897" height="225" alt="task5" src="https://github.com/user-attachments/assets/48f899d3-4ab3-46f5-b5ad-81fe77fa1673" />

<img width="993" height="222" alt="task6" src="https://github.com/user-attachments/assets/a5eabf59-bf8e-4ef5-a06d-89d52578b3b1" />

<img width="850" height="257" alt="task6-1" src="https://github.com/user-attachments/assets/c4f5ba8c-669a-41b3-9841-3cbe817113bb" />

<img width="1057" height="532" alt="task7" src="https://github.com/user-attachments/assets/df5ab98d-b471-422a-9f35-60effa762d13" />


---

# 📂 Submission Structure

[ 
2026/
 └── day-52/
      ├── day-52-namespaces-deployments.md
      ├── nginx-deployment.yaml
      └── namespace.yaml
]

---

# 🏁 Final Mentor Note

Today you moved from:
❌ Manual Pods → ✅ Production-grade Deployments

This is a BIG step toward real DevOps engineering.

Next level topics will build on this:
- Services
- Ingress
- Helm
- CI/CD with Kubernetes

Keep going 🚀
