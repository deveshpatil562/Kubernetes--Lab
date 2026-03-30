# 🚀 Kubernetes DevOps Lab Guide: Resource Management & Probes

Welcome 👨‍💻  
This lab is designed to give you **real production-level understanding** of how Kubernetes handles:
- Resource management (CPU/Memory)
- Pod lifecycle issues
- Health checks (Liveness, Readiness, Startup)

We will not just run commands — we will **observe behavior like a real DevOps engineer**.

---

# 🧠 Core Concepts (Before You Start)

## 🔹 Requests vs Limits
- Requests → Minimum guaranteed resources (used by scheduler)
- Limits → Maximum allowed usage (enforced by kubelet)

## 🔹 QoS Classes
- Guaranteed → Requests = Limits
- Burstable → Requests ≠ Limits
- BestEffort → No requests/limits

## 🔹 CPU vs Memory Behavior
- CPU → Throttled if exceeded
- Memory → Killed if exceeded (OOMKilled)

---

# 🧪 Task 1: Resource Requests and Limits

## 🎯 Objective
Understand how Kubernetes assigns QoS and schedules Pods.

## 📄 Create Pod Manifest

[apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "250m"
        memory: "256Mi"]

---

## ▶️ Apply the Pod

[kubectl apply -f resource-demo.yaml]

---

## 🔍 Inspect the Pod

[kubectl describe pod resource-demo]

---

## ✅ What to Look For
- Requests section
- Limits section
- QoS Class → **Burstable**

---

## 🧠 Mentor Insight
Since requests ≠ limits → Kubernetes classifies this as **Burstable**, meaning:
- It gets guaranteed minimum resources
- Can burst when extra resources are available

---

## 📸 Screenshot Placeholder

<img width="887" height="867" alt="task1" src="https://github.com/user-attachments/assets/714e0298-8b79-4ded-96d9-7dcc0a0e3aa9" />

---

## ✔️ Verification
Q: What QoS class does your Pod have?  
A: **Burstable**

---

# 🧪 Task 2: OOMKilled — Exceeding Memory Limits

## 🎯 Objective
See what happens when a container exceeds memory limits.

---

## 📄 Create Pod Manifest

[apiVersion: v1
kind: Pod
metadata:
  name: oom-demo
spec:
  containers:
  - name: stress
    image: polinux/stress
    resources:
      limits:
        memory: "100Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "200M", "--vm-hang", "1"]]

---

## ▶️ Apply and Watch

[kubectl apply -f oom-demo.yaml]

[kubectl get pods -w]

---

## 🔍 Describe Pod

[kubectl describe pod oom-demo]

---

## ✅ Expected Behavior
- Container gets killed immediately
- Reason: **OOMKilled**
- Exit Code: **137**

---

## 🧠 Mentor Insight
- Memory limit breach = instant kill (no negotiation)
- Exit code 137 = SIGKILL (128 + 9)

---

## 📸 Screenshot Placeholder

<img width="1622" height="917" alt="task2" src="https://github.com/user-attachments/assets/08b57743-427f-454a-a378-f74bc420700e" />


---

## ✔️ Verification
Q: What exit code does an OOMKilled container have?  
A: **137**

---

# 🧪 Task 3: Pending Pod — Over Requesting Resources

## 🎯 Objective
Understand scheduling failures.

---

## 📄 Create Pod Manifest

[apiVersion: v1
kind: Pod
metadata:
  name: huge-request
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: "100"
        memory: "128Gi"]

---

## ▶️ Apply

[kubectl apply -f huge-request.yaml]

---

## 🔍 Check Status

[kubectl get pods]

---

## 🔍 Describe Pod

[kubectl describe pod huge-request]

---

## ✅ Expected Behavior
- Pod stays in **Pending**
- Scheduler cannot place it

---

## 🧠 Mentor Insight
Scheduler checks:
- Node capacity
- Available resources

If no node satisfies → Pod stays Pending forever

---

## 📸 Screenshot Placeholder

<img width="1650" height="812" alt="task3" src="https://github.com/user-attachments/assets/9ede3734-c0a6-4517-9a3d-208048c6976b" />


---

## ✔️ Verification
Q: What event message does scheduler produce?  
A: **Insufficient cpu / memory**

---

# 🧪 Task 4: Liveness Probe

## 🎯 Objective
Automatically restart unhealthy containers.

---

## 📄 Create Pod Manifest

[apiVersion: v1
kind: Pod
metadata:
  name: liveness-demo
spec:
  containers:
  - name: busybox
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      touch /tmp/healthy;
      sleep 30;
      rm -f /tmp/healthy;
      sleep 3600;
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
      failureThreshold: 3]

---

## ▶️ Apply

[kubectl apply -f liveness-demo.yaml]

---

## 👀 Watch Pod

[kubectl get pod liveness-demo -w]

---

## ✅ Expected Behavior
- File exists initially → healthy
- After 30 sec → file removed
- Probe fails 3 times → container restarts

---

## 🧠 Mentor Insight
Liveness probe = **self-healing mechanism**
Kubernetes assumes:
"If it's unhealthy → restart it"

---

## 📸 Screenshot Placeholder

<img width="1382" height="683" alt="task4" src="https://github.com/user-attachments/assets/999f1046-d645-44df-8774-dd9190533166" />


---

## ✔️ Verification
Q: How many times has the container restarted?  
A: Check RESTARTS column (should increment)

---

# 🧪 Task 5: Readiness Probe

## 🎯 Objective
Control traffic flow without restarting container.

---

## 📄 Create Pod

[apiVersion: v1
kind: Pod
metadata:
  name: readiness-demo
spec:
  containers:
  - name: nginx
    image: nginx
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5]

---

## ▶️ Apply

[kubectl apply -f readiness-demo.yaml]

---

## 🌐 Expose Service

[kubectl expose pod readiness-demo --port=80 --name=readiness-svc]

---

## 🔍 Check Endpoints

[kubectl get endpoints readiness-svc]

---

## 💥 Break the App

[kubectl exec readiness-demo -- rm /usr/share/nginx/html/index.html]

---

## 🔍 Observe Again

[kubectl get pods]

[kubectl get endpoints readiness-svc]

---

## ✅ Expected Behavior
- Pod shows 0/1 READY
- Endpoints become empty
- Container is NOT restarted

---

## 🧠 Mentor Insight
Readiness = **traffic control, not healing**

---

## 📸 Screenshot Placeholder

<img width="1627" height="680" alt="task5" src="https://github.com/user-attachments/assets/9314bb22-8d68-4655-ba6e-fd620f44e0ab" />


---

## ✔️ Verification
Q: Was the container restarted?  
A: **No**

---

# 🧪 Task 6: Startup Probe

## 🎯 Objective
Handle slow-starting applications.

---

## 📄 Create Pod Manifest

[apiVersion: v1
kind: Pod
metadata:
  name: startup-demo
spec:
  containers:
  - name: busybox
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      sleep 20;
      touch /tmp/started;
      sleep 3600;
    startupProbe:
      exec:
        command:
        - cat
        - /tmp/started
      periodSeconds: 5
      failureThreshold: 12
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/started
      periodSeconds: 5]

---

## ▶️ Apply

[kubectl apply -f startup-demo.yaml]

---

## 👀 Watch

[kubectl get pod startup-demo -w]

---

## ✅ Expected Behavior
- Startup probe runs first
- Liveness probe waits
- Once file exists → normal checks begin

---

## 🧠 Mentor Insight
Startup probe = **grace period protector**
Without it, slow apps get killed early

---

## 📸 Screenshot Placeholder

<img width="1515" height="722" alt="task6" src="https://github.com/user-attachments/assets/4596dcd1-db23-4ec1-adf0-64f2f374a79e" />


---

## ✔️ Verification
Q: What if failureThreshold = 2?  
A: Pod would restart before app finishes starting

---

# 🏁 Final Summary

| Feature         | Purpose |
|----------------|--------|
| Requests       | Scheduling guarantee |
| Limits         | Runtime enforcement |
| Liveness       | Restart unhealthy container |
| Readiness      | Control traffic |
| Startup        | Protect slow apps |

---

# 💡 Pro Tip from Mentor

In real production:
- Always define requests & limits
- Always use readiness probes
- Use liveness carefully (avoid restart loops)
- Use startup probes for Java / slow apps

---
