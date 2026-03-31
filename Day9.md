# 🚀 Kubernetes Lab: Metrics Server & Horizontal Pod Autoscaler (HPA)

Welcome 👨‍💻  
In this lab, you’ll move from **static resource configuration → dynamic autoscaling**, just like real production systems.

You will:
- Install Metrics Server (real-time usage data)
- Use kubectl top to observe resource usage
- Configure HPA to scale automatically under load

---

# 🧠 Core Concepts

## 🔹 What is Metrics Server?
- Collects CPU & Memory usage from kubelets
- Provides data for:
  - kubectl top
  - Horizontal Pod Autoscaler (HPA)

👉 Without Metrics Server → HPA will NOT work

---

## 🔹 What is HPA?
HPA automatically scales Pods based on usage.

### Formula:
desiredReplicas = ceil(currentReplicas × (currentUsage / targetUsage))

---

## 🔹 Key Rule
👉 HPA NEEDS resources.requests  
Without it → TARGETS = <unknown>

---

## 🔹 autoscaling/v1 vs v2
- v1 → CPU only
- v2 → CPU + Memory + Custom metrics + behavior control

---

# 🧪 Task 1: Install Metrics Server

## ▶️ Check if already installed

[kubectl get pods -n kube-system | grep metrics-server]

---

## ▶️ Install Metrics Server

### Minikube
[minikube addons enable metrics-server]

### Kind / kubeadm
[kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml]

---

## ⚠️ Local Cluster Fix (if needed)

Edit deployment and add:
[--kubelet-insecure-tls]

---

## ⏳ Wait and Verify

[kubectl top nodes]

[kubectl top pods -A]

---

## 📸 Screenshot Placeholder

<img width="1658" height="457" alt="task1" src="https://github.com/user-attachments/assets/661c3e4d-99c6-47fe-ad55-1e0559b68b98" />


---

## ✔️ Verification
Q: What is current CPU & memory usage of node?  
A: Check from kubectl top nodes output

---

# 🧪 Task 2: Explore kubectl top

## ▶️ Run Commands

[kubectl top nodes]

[kubectl top pods -A]

[kubectl top pods -A --sort-by=cpu]

---

## 🧠 Mentor Insight
- kubectl top = real usage
- kubectl describe = configured values

---

## 📸 Screenshot Placeholder

<img width="1107" height="858" alt="task2" src="https://github.com/user-attachments/assets/8802e44f-f011-429a-b271-7c593ea5bd05" />


---

## ✔️ Verification
Q: Which pod is using most CPU?  
A: First entry in sorted output

---

# 🧪 Task 3: Create Deployment with CPU Requests

## 🎯 Important
HPA depends on CPU requests to calculate %

---

## 📄 Deployment Manifest

[apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: php-apache
  template:
    metadata:
      labels:
        app: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "200m"]

---

## ▶️ Apply

[kubectl apply -f php-apache.yaml]

---

## 🌐 Expose Service

[kubectl expose deployment php-apache --port=80]

---

## 📸 Screenshot Placeholder

<img width="1598" height="916" alt="task3" src="https://github.com/user-attachments/assets/4e6b0905-b29f-4087-aa26-671f68013914" />


---

## ✔️ Verification
Q: What is current CPU usage?  
A: Check using kubectl top pods

---

# 🧪 Task 4: Create HPA (Imperative)

## ▶️ Create HPA

[kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10]

---

## ▶️ Check HPA

[kubectl get hpa]

[kubectl describe hpa php-apache]

---

## ⏳ Wait 30 sec if TARGETS shows <unknown>

---

## 🧠 Mentor Insight
HPA triggers when:
- CPU > 50% → scale up
- CPU < 50% → scale down

---

## 📸 Screenshot Placeholder

<img width="1098" height="281" alt="task4" src="https://github.com/user-attachments/assets/5c0efc68-b826-41d8-b026-cef988c20b7d" />


---

## ✔️ Verification
Q: What does TARGETS show?  
A: Current CPU usage vs target (example: 60%/50%)

---

# 🧪 Task 5: Generate Load & Watch Scaling

## ▶️ Create Load Generator

[kubectl run load-generator --image=busybox:1.36 --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://php-apache; done"]

---

## ▶️ Watch HPA

[kubectl get hpa php-apache --watch]

---

## ▶️ Observe Scaling

- CPU increases
- Replicas increase
- System stabilizes

---

## 🛑 Stop Load

[kubectl delete pod load-generator]

---

## 🧠 Mentor Insight
- Scale UP = fast ⚡
- Scale DOWN = slow (5 min stabilization)

---

## 📸 Screenshot Placeholder

<img width="1110" height="307" alt="task5" src="https://github.com/user-attachments/assets/5702fc86-f83a-4d9e-a4ad-b1425a02356f" />


---

## ✔️ Verification
Q: How many replicas did it scale to?  
A: Check REPLICAS column

---

# 🧪 Task 6: HPA using YAML (Declarative)

## ▶️ Delete old HPA

[kubectl delete hpa php-apache]

---

## 📄 HPA Manifest (autoscaling/v2)

[apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
    scaleDown:
      stabilizationWindowSeconds: 300]

---

## ▶️ Apply

[kubectl apply -f hpa.yaml]

---

## ▶️ Verify

[kubectl describe hpa php-apache]

---

## 🧠 Mentor Insight
Behavior controls:
- Scale-up speed
- Scale-down delay

---

## 📸 Screenshot Placeholder

<img width="1643" height="937" alt="task6" src="https://github.com/user-attachments/assets/4b4a3684-5f7e-4983-833f-47fdd39e402f" />


---

## ✔️ Verification
Q: What does behavior section control?  
A: Scaling speed and stabilization windows

---

# 🧪 Task 7: Clean Up

[kubectl delete hpa php-apache]

[kubectl delete svc php-apache]

[kubectl delete deployment php-apache]

[kubectl delete pod load-generator]

---

# 📘 Documentation (For GitHub)

## File: day-58-metrics-hpa.md

### Include:
- What is Metrics Server
- Why HPA needs it
- HPA formula
- autoscaling/v1 vs v2
- Screenshots:
  - kubectl top
  - HPA scaling
  - Events

---

# 🏁 Final Summary

| Component | Role |
|----------|------|
| Metrics Server | Provides usage data |
| kubectl top | Shows real-time usage |
| HPA | Auto scales pods |
| Requests | Required for scaling |

---

# 💡 Pro Tips (Real Interview Level)

- No requests → HPA broken ❌  
- CPU scaling is percentage-based  
- HPA checks every 15 sec  
- Scale down is intentionally slow  

---

