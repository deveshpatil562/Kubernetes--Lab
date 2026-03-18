# 🚀 Kubernetes Architecture and Cluster Setup

> Moving from containers → orchestration. This is where real DevOps begins.

---

# 📘 Concepts First

## 🔥 Why Kubernetes?

Docker helps run containers on a single machine.

But in real-world systems, you need:
- Multi-server deployments
- Auto-healing
- Scaling
- Load balancing

👉 Kubernetes solves this by acting as a **container orchestrator**

---

## 🧠 Kubernetes Story

Kubernetes was created by Google based on their internal system Borg. It helps manage containerized applications at scale by automating deployment, scaling, and recovery. The word "Kubernetes" means **helmsman (ship pilot)**.

---

# 🏗️ Kubernetes Architecture

## 📊 Clean Architecture Diagram

<img width="1877" height="806" alt="Kubernetes Architecture" src="https://github.com/user-attachments/assets/4310672a-e4f5-425e-b96c-181ea0ee5357" />

---

## 🔄 Request Flow

Command:
[ kubectl apply -f pod.yaml ]

Flow:
1. kubectl → API Server  
2. API Server → etcd  
3. Scheduler → selects node  
4. kubelet → runs container  
5. Controller → maintains state  

---

## ⚠️ Failure Handling

- API Server down → ❌ No cluster operations possible  
- Worker node down → ✅ Pods rescheduled automatically  

---

# 🛠️ Hands-On Lab

---

# ✅ Task 1: Kubernetes Recall

Kubernetes is used to manage containerized applications at scale.
It was created by Google based on Borg.
It automates deployment, scaling, and self-healing.
Kubernetes means "helmsman".


---

# 🎯 Task 2: Architecture Verification

✔️ kubectl apply flow → Verified  
✔️ API server failure → Cluster unusable  
✔️ Node failure → Pods rescheduled  

---

# 🧰 Task 3: Install kubectl

## Linux

[
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
]

## Verify

[ kubectl version --client ]


---

# ⚙️ Task 4: Setup Cluster (Kind)

## Install

[
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
]

## Create Cluster

[
kind create cluster --name devops-cluster
]

## Verify

[
kubectl cluster-info
kubectl get nodes
]


## Why Kind?

Kind is lightweight, fast, and runs inside Docker which makes it easy to use.


---

# 🔍 Task 5: Explore Cluster

## Commands

[
kubectl cluster-info
kubectl get nodes
kubectl describe node <node-name>
kubectl get namespaces
kubectl get pods -A
]

## System Pods

[
kubectl get pods -n kube-system
]

## Components Mapping

- kube-apiserver → API Server  
- etcd → Database  
- kube-scheduler → Scheduler  
- kube-controller-manager → Controller  
- kube-proxy → Networking  
- coredns → DNS  


---

# 🔁 Task 6: Cluster Lifecycle

## Delete

[
kind delete cluster --name devops-cluster
]

## Recreate

[
kind create cluster --name devops-cluster
]

## Verify

[ kubectl get nodes ]

---

## Useful Commands

[
kubectl config current-context
kubectl config get-contexts
kubectl config view
]

---

## 🧠 kubeconfig

- Stores cluster access details
- Default location:



---

# 📘 GitHub Documentation Section

## Kubernetes History

Kubernetes is an orchestration tool designed to manage containerized applications across multiple systems. It was developed by Google based on Borg. It automates deployment, scaling, and recovery of applications.

---

## Tool Used

Kind (Kubernetes in Docker)


---

## Final Outcome

✅ Cluster created  
✅ Architecture understood  
✅ kubectl practiced  
✅ Components verified  

---

# 🚀 LinkedIn Post

Started my Kubernetes journey today 🚀  
Set up a local cluster and explored architecture.

The orchestration chapter begins.

#90DaysOfDevOps #DevOpsKaJosh #TrainWithShubham

---

# 💡 Mentor Tip

Understand flow > memorize commands  
Kubernetes interviews test your thinking, not just syntax.

---

