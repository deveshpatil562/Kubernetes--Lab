# 🚀 DevOps Lab Guide — Helm (Kubernetes Package Manager)

## 📌 Introduction

Over the past labs, you manually created multiple Kubernetes resources like Deployments, Services, ConfigMaps, Secrets, and PVCs.

👉 In real-world production:
- Applications consist of **dozens of YAML files**
- Managing them individually becomes complex and error-prone

💡 **Helm solves this problem.**

### 🔹 What is Helm?
Helm is a **package manager for Kubernetes**, just like:
- apt → Ubuntu
- yum → RHEL
- npm → Node.js

With Helm, you can:
- Install complete applications using **one command**
- Customize configurations easily
- Upgrade and rollback safely

---

## 🧠 Core Concepts (Must Know)

### 1. Chart
A **Helm package** containing Kubernetes YAML templates.

👉 Example:
Instead of writing Deployment + Service + ConfigMap manually, Helm bundles them.

---

### 2. Release
A **running instance of a chart** in your cluster.

👉 Example:
You can install nginx multiple times:
- my-nginx
- test-nginx

Each is a separate **release**

---

### 3. Repository
A **collection of charts**, similar to package repositories.

👉 Example:
Bitnami repo contains production-ready charts.

---

# 🧪 Task 1: Install Helm

## Step 1: Install Helm

### Linux (curl)
[ curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash ]

### Mac (brew)
[ brew install helm ]

### Windows (choco)
[ choco install kubernetes-helm ]

---

## Step 2: Verify Installation

[ helm version ]
[ helm env ]

---

## ✅ Verification

✔ What version is installed?  
👉 Check output of helm version

---

## 📸 Screenshot Placeholder

<img width="1283" height="586" alt="task1" src="https://github.com/user-attachments/assets/fdb63b64-c636-4994-9e0a-d63fee0932d3" />


---

# 🧪 Task 2: Add Repository & Search

## Step 1: Add Bitnami Repo

[ helm repo add bitnami https://charts.bitnami.com/bitnami ]

---

## Step 2: Update Repo

[ helm repo update ]

---

## Step 3: Search Charts

[ helm search repo nginx ]
[ helm search repo bitnami ]

---

## ✅ Verification

✔ How many charts Bitnami has?  
👉 Check count in output

---

## 📸 Screenshot Placeholder

<img width="1182" height="332" alt="task2" src="https://github.com/user-attachments/assets/cc07a075-7c21-4a4f-a250-7ba23db70200" />


---

# 🧪 Task 3: Install a Chart

## Step 1: Install NGINX

[ helm install my-nginx bitnami/nginx ]

---

## Step 2: Check Resources

[ kubectl get all ]

---

## Step 3: Inspect Release

[ helm list ]
[ helm status my-nginx ]
[ helm get manifest my-nginx ]

---

## 🧠 Mentor Insight

🔥 One Helm command created:
- Deployment
- Service
- ConfigMap
- Pods

👉 This replaces writing multiple YAML files manually.

---

## ✅ Verification

✔ How many Pods are running?  
✔ What Service type is created?


---

# 🧪 Task 4: Customize with Values

## Step 1: View Default Values

[ helm show values bitnami/nginx ]

---

## Step 2: Install with CLI Overrides

[ helm install custom-nginx bitnami/nginx --set replicaCount=3 --set service.type=NodePort ]

---

## Step 3: Create Values File

Create file: custom-values.yaml

[ 
replicaCount: 3

service:
  type: NodePort

resources:
  limits:
    cpu: "200m"
    memory: "256Mi"
  requests:
    cpu: "100m"
    memory: "128Mi"
]

---

## Step 4: Install Using File

[ helm install file-nginx bitnami/nginx -f custom-values.yaml ]

---

## Step 5: Verify Values

[ helm get values file-nginx ]

---

## 🧠 Mentor Insight

💡 Two ways to customize:
- --set → quick changes
- values.yaml → production-level configs

---

## ✅ Verification

✔ Are replicas = 3?  
✔ Is service type NodePort?

---

## 📸 Screenshot Placeholder

<img width="1620" height="947" alt="task4" src="https://github.com/user-attachments/assets/3e1f8b45-e1cc-4db5-940c-7d1af837c7e8" />


---

# 🧪 Task 5: Upgrade & Rollback

## Step 1: Upgrade Release

[ helm upgrade my-nginx bitnami/nginx --set replicaCount=5 ]

---

## Step 2: Check History

[ helm history my-nginx ]

---

## Step 3: Rollback

[ helm rollback my-nginx 1 ]

---

## Step 4: Check History Again

[ helm history my-nginx ]

---

## 🧠 Mentor Insight

🔥 Important:
- Rollback **creates a new revision**
- It does NOT overwrite history

👉 Similar to Kubernetes rollout but at **application level**

---

## ✅ Verification

✔ How many revisions exist after rollback?

---

## 📸 Screenshot Placeholder

<img width="1037" height="262" alt="task5" src="https://github.com/user-attachments/assets/4d9cb31b-67d6-4a2d-b6f3-6b70882c291c" />


---

# 🧪 Task 6: Create Your Own Chart

## Step 1: Scaffold Chart

[ helm create my-app ]

---

## Step 2: Explore Structure

Key files:
- Chart.yaml → metadata
- values.yaml → default config
- templates/ → Kubernetes manifests

---

## Step 3: Edit values.yaml

[ 
replicaCount: 3

image:
  repository: nginx
  tag: "1.25"
]

---

## Step 4: Validate Chart

[ helm lint my-app ]

---

## Step 5: Preview Templates

[ helm template my-release ./my-app ]

---

## Step 6: Install Chart

[ helm install my-release ./my-app ]

---

## Step 7: Upgrade Chart

[ helm upgrade my-release ./my-app --set replicaCount=5 ]

---

## 🧠 Understanding Templates

Helm uses Go templating:

Examples:
- {{ .Values.replicaCount }}
- {{ .Chart.Name }}
- {{ .Release.Name }}

👉 These dynamically inject values into YAML

---

## ✅ Verification

✔ After install → 3 replicas?  
✔ After upgrade → 5 replicas?

---

## 📸 Screenshot Placeholder

<img width="1332" height="951" alt="task6" src="https://github.com/user-attachments/assets/f92826b2-d1cc-40a0-b722-71dc002a9aed" />


---

# 🧪 Task 7: Cleanup

## Step 1: Uninstall Releases

[ helm uninstall my-nginx ]
[ helm uninstall custom-nginx ]
[ helm uninstall file-nginx ]
[ helm uninstall my-release ]

---

## Step 2: Verify

[ helm list ]

---

## 🧠 Mentor Insight

💡 Use this for auditing:
[ helm uninstall <name> --keep-history ]

---

## ✅ Verification

✔ Does helm list show zero releases?


---

# 📄 Documentation Section (For GitHub)

## 🔹 What is Helm?

Helm is a Kubernetes package manager that simplifies deploying and managing applications using charts.

---

## 🔹 Core Concepts

- Chart → Application package
- Release → Running instance
- Repository → Collection of charts

---

## 🔹 Key Commands Learned

- Install → helm install
- Upgrade → helm upgrade
- Rollback → helm rollback
- Delete → helm uninstall

---

## 🔹 Chart Structure

- Chart.yaml → metadata
- values.yaml → configuration
- templates/ → Kubernetes manifests

---

## 🔹 Custom Values File Explanation

- replicaCount → number of pods
- service.type → ClusterIP / NodePort
- resources → CPU & memory limits


---

# 🎯 Final Mentor Note

If Kubernetes is the **engine**, Helm is the **automation system**.

👉 Mastering Helm means:
- Faster deployments
- Standardized environments
- Production-ready DevOps skills

🔥 This is a MUST-HAVE skill for real DevOps engineers.

Happy Learning 🚀
