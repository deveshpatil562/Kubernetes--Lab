# 🚀 Kubernetes Lab Guide — Manifests & Your First Pods

## 📌 Objective
You’ve already set up your Kubernetes cluster. Now it’s time to deploy your first real workloads using manifests.

By the end of this lab, you will:
- Understand Kubernetes YAML structure
- Create Pods from scratch
- Debug running containers
- Learn declarative vs imperative approaches
- Use labels effectively

---

# 🧠 Concept: Anatomy of a Kubernetes Manifest

Every Kubernetes resource is defined using a YAML file with 4 key sections:

[apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
  - name: my-container
    image: nginx:latest
    ports:
    - containerPort: 80]

## 🔍 Explanation (Mentor Style)

✅ apiVersion  
- Defines which Kubernetes API to use  
- For Pods → v1  

✅ kind  
- Type of resource  
- Today → Pod  

✅ metadata  
- Identity of resource  
- Includes:
  - name (required)
  - labels (used for filtering)

✅ spec  
- Desired state  
- Defines:
  - containers
  - images
  - ports
  - commands  

👉 Mentor Insight: If you understand "spec", you understand Kubernetes.

---

# 🧪 Task 1: Create Your First Pod (Nginx)

## 📄 Step 1: Create Manifest

[apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80]

---

## ▶️ Step 2: Apply

[kubectl apply -f nginx-pod.yaml]

---

## 🔍 Step 3: Verify

[kubectl get pods]  
[kubectl get pods -o wide]

📸 Screenshot Placeholder  
Add screenshot showing nginx-pod in Running state

---

## 🔎 Step 4: Inspect Pod

[kubectl describe pod nginx-pod]

---

## 📜 Step 5: Logs

[kubectl logs nginx-pod]

---

## 🖥️ Step 6: Exec Into Pod

[kubectl exec -it nginx-pod -- /bin/bash]  
[curl localhost:80]  
[exit]

---

## ✅ Verification

- Pod status → Running  
- Curl output → Nginx welcome page  


<img width="1643" height="952" alt="TASK-1" src="https://github.com/user-attachments/assets/7e7dfdb3-84d8-4ca4-8757-b67845146787" />

<img width="885" height="783" alt="task1-1" src="https://github.com/user-attachments/assets/c5f593da-44f3-469e-8d16-a08ec52fafdf" />


---

## 🧠 Mentor Explanation

- Nginx is running inside container  
- Port 80 is exposed internally  
- Curl proves:
  - Container is alive
  - Application is working

---

# 🧪 Task 2: Create Custom Pod (BusyBox)

## 📄 Step 1: Write Manifest

[apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
  labels:
    app: busybox
    environment: dev
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command: ["sh", "-c", "echo Hello from BusyBox && sleep 3600"]]

---

## ▶️ Step 2: Apply

[kubectl apply -f busybox-pod.yaml]

---

## 🔍 Step 3: Verify

[kubectl get pods]  
[kubectl logs busybox-pod]

📸 Screenshot Placeholder  
Add screenshot showing BusyBox logs

---

## ✅ Verification

Expected output:  
Hello from BusyBox  


<img width="866" height="345" alt="task2" src="https://github.com/user-attachments/assets/76c6319c-c9de-4829-9b45-23a4f370bf64" />

---

## 🧠 Mentor Explanation

- BusyBox exits immediately by default  
- sleep 3600 keeps container alive  
- Without it → CrashLoopBackOff  

👉 Always ensure a long-running process

---

# 🧪 Task 3: Imperative vs Declarative

## ⚡ Step 1: Imperative

[kubectl run redis-pod --image=redis:latest]  
[kubectl get pods]

---

## 📄 Step 2: Extract YAML

[kubectl get pod redis-pod -o yaml]

---

## ⚡ Step 3: Generate YAML

[kubectl run test-pod --image=nginx --dry-run=client -o yaml]

---

## ✅ Verification

- Compare generated YAML with nginx manifest  

<img width="1626" height="907" alt="TASK3" src="https://github.com/user-attachments/assets/8f5d4556-48f9-4b8b-bee2-d7e65a3573c2" />

---

## 🧠 Mentor Explanation

Declarative:
- You define desired state using YAML  

Imperative:
- You run commands directly  

👉 Industry standard → Declarative (GitOps)

👉 Kubernetes auto-adds:
- UID
- timestamps
- status

---

# 🧪 Task 4: Validate Before Applying

## 🧪 Step 1: Client Validation

[kubectl apply -f nginx-pod.yaml --dry-run=client]

---

## 🧪 Step 2: Server Validation

[kubectl apply -f nginx-pod.yaml --dry-run=server]

---

## ❌ Step 3: Break YAML

Remove image field and run again

---

## ✅ Verification

Expected error:  
spec.containers[0].image: Required value  

---

## 🧠 Mentor Explanation

- Validation prevents bad deployments  
- Always use dry-run before applying  

---

# 🧪 Task 5: Labels & Filtering

## 🔍 Step 1: Show Labels

[kubectl get pods --show-labels]

---

## 🎯 Step 2: Filter Pods

[kubectl get pods -l app=nginx]  
[kubectl get pods -l environment=dev]

---

## ➕ Step 3: Add Label

[kubectl label pod nginx-pod environment=production]

---

## ➖ Step 4: Remove Label

[kubectl label pod nginx-pod environment-]

---

## 📄 Step 5: Third Pod Manifest

[apiVersion: v1
kind: Pod
metadata:
  name: multi-label-pod
  labels:
    app: demo
    environment: testing
    team: devops
spec:
  containers:
  - name: nginx
    image: nginx:latest]

---

## ▶️ Step 6: Apply

[kubectl apply -f multi-label-pod.yaml]

---

## 🔍 Step 7: Verify

[kubectl get pods -l team=devops]

📸 Screenshot Placeholder  

<img width="1632" height="916" alt="task5" src="https://github.com/user-attachments/assets/69bd9b32-45da-43f5-9988-129199a4c26e" />

---

## 🧠 Mentor Explanation

- Labels are key-value pairs  
- Used for filtering and grouping  
- Critical for Services and Deployments  

---

# 🧪 Task 6: Cleanup

## 🗑️ Delete Pods

[kubectl delete pod nginx-pod]  
[kubectl delete pod busybox-pod]  
[kubectl delete pod redis-pod]

OR

[kubectl delete -f nginx-pod.yaml]

---

## 🔍 Verify

[kubectl get pods]

📸 Screenshot Placeholder  

<img width="725" height="280" alt="task6" src="https://github.com/user-attachments/assets/8ee55b84-b624-452a-9b78-e3e6ff10b9ef" />

---

## 🧠 Mentor Explanation

- Pods are ephemeral  
- No controller = no auto-recreation  

👉 In production → use Deployments

---

# 📘 Final Summary

You learned:
- Kubernetes manifest structure  
- Pod creation from scratch  
- Debugging with logs & describe  
- Importance of long-running containers  
- Declarative vs Imperative  
- Labels and filtering  
- Validation using dry-run  

---

# 📂 Repository Structure

2026/day-51/
- day-51-pods.md  
- nginx-pod.yaml  
- busybox-pod.yaml  
- multi-label-pod.yaml  

---

# 📸 Final Proof

<img width="1626" height="907" alt="TASK3" src="https://github.com/user-attachments/assets/09926864-9b1f-40fc-b033-2ac91c17df07" />


kubectl get pods showing all pods Running  

---

# 🧠 Mentor Closing Advice

If you can now:
- Write Pod YAML without help  
- Debug issues confidently  
- Explain CrashLoopBackOff  
- Use labels effectively  

👉 You’re building real DevOps skills.

🔥 Next step: Deployments (self-healing & scaling)
