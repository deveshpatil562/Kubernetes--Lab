# 📦 Kubernetes StatefulSets Lab Guide

## 📘 Overview

In Kubernetes, **Deployments** are perfect for stateless applications like web servers. However, when it comes to **stateful applications** such as databases (MySQL, PostgreSQL, Kafka), we need:

- Stable pod identities
- Ordered startup and shutdown
- Persistent storage tied to each pod

👉 This is where **StatefulSets** come in.

---

## 🧠 Concept First: Why StatefulSets?

| Feature              | Deployment                  | StatefulSet                          |
|---------------------|---------------------------|--------------------------------------|
| Pod Names           | Random (app-xyz-abc)      | Stable (app-0, app-1, app-2)         |
| Startup Order       | Parallel                  | Ordered                              |
| Storage             | Shared / Optional         | Dedicated PVC per Pod                |
| Network Identity    | Dynamic                   | Stable DNS per Pod                   |

---

## 🎯 Lab Goals

By the end of this lab, you will have:

✅ StatefulSet with 3 replicas  
✅ Stable pod names (web-0, web-1, web-2)  
✅ DNS resolution verified  
✅ Persistent storage validated  
✅ Ordered scaling behavior understood  

---

# 🚀 Task 1: Understand the Problem

## Step 1: Create a Deployment

[ kubectl create deployment nginx-deploy --image=nginx --replicas=3 ]

## Step 2: Check Pods

[ kubectl get pods ]

👉 You will see random names like:
- nginx-deploy-abc123
- nginx-deploy-def456

## Step 3: Delete a Pod

[ kubectl delete pod <pod-name> ]

👉 A new pod will be created with a **different name**

---

## 🔍 Mentor Insight

This randomness is **fine for stateless apps**, but:

❌ Databases need fixed identity  
❌ Replicas need predictable names  
❌ Clusters rely on stable network identity  

---

## ✅ Verify

**Q: Why are random pod names bad for databases?**

✔ Because databases depend on stable identity for replication, clustering, and communication.

---

## Step 4: Cleanup Deployment

[ kubectl delete deployment nginx-deploy ]

---

# 🌐 Task 2: Create a Headless Service

## 📘 Concept

A **Headless Service** does NOT load balance.  
Instead, it provides **direct DNS entries for each pod**.

---

## Step 1: Create Service Manifest

[ 
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  clusterIP: None
  selector:
    app: nginx
  ports:
    - port: 80
]

## Step 2: Apply Service

[ kubectl apply -f service.yaml ]

## Step 3: Verify

[ kubectl get svc ]

---

## ✅ Verify

✔ CLUSTER-IP should show:

👉 None

---


# 🧱 Task 3: Create StatefulSet

## 📘 Concept

StatefulSet ensures:

- Ordered pod creation
- Stable naming
- Persistent storage per pod

---

## Step 1: Create StatefulSet Manifest

[ 
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: web
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
        image: nginx
        volumeMounts:
        - name: web-data
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: web-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 100Mi
]

## Step 2: Apply

[ kubectl apply -f statefulset.yaml ]

## Step 3: Watch Pods

[ kubectl get pods -l app=nginx -w ]

---

## 🔍 Mentor Insight

Observe carefully:

✔ web-0 starts first  
✔ web-1 waits until web-0 is ready  
✔ web-2 waits until web-1 is ready  

👉 This is **ordered deployment**

---

## Step 4: Check PVCs

[ kubectl get pvc ]

Expected:

- web-data-web-0  
- web-data-web-1  
- web-data-web-2  

---

## ✅ Verify

**Pod Names:**
- web-0
- web-1
- web-2

**PVC Names:**
- web-data-web-0
- web-data-web-1
- web-data-web-2

---

## 📸 Screenshot Placeholder

<img width="1652" height="823" alt="task3" src="https://github.com/user-attachments/assets/c1294ca6-d747-4bdb-ba00-a37717138f29" />


---

# 🌍 Task 4: Stable Network Identity

## 📘 Concept

Each pod gets DNS:

👉 <pod-name>.<service-name>.<namespace>.svc.cluster.local

---

## Step 1: Run Busybox

[ kubectl run -it --rm busybox --image=busybox -- sh ]

## Step 2: DNS Lookup

[ nslookup web-0.web.default.svc.cluster.local ]

[ nslookup web-1.web.default.svc.cluster.local ]

[ nslookup web-2.web.default.svc.cluster.local ]

## Step 3: Compare IPs

[ kubectl get pods -o wide ]

---

## ✅ Verify

✔ DNS IP matches Pod IP  


---

# 💾 Task 5: Persistent Storage

## 📘 Concept

Each pod has its own storage (PVC).  
Deleting pod does NOT delete data.

---

## Step 1: Write Data

[ kubectl exec web-0 -- sh -c "echo 'Data from web-0' > /usr/share/nginx/html/index.html" ]

## Step 2: Delete Pod

[ kubectl delete pod web-0 ]

## Step 3: Wait for Recreation

[ kubectl get pods ]

## Step 4: Verify Data

[ kubectl exec web-0 -- cat /usr/share/nginx/html/index.html ]

---

## 🔍 Mentor Insight

Even after deletion:

✔ Pod recreated  
✔ Same PVC attached  
✔ Data persists  

---

## ✅ Verify

✔ Data remains: "Data from web-0"

---

# 📈 Task 6: Ordered Scaling

## Step 1: Scale Up

[ kubectl scale statefulset web --replicas=5 ]

## Step 2: Observe

Pods created in order:
- web-3
- web-4

---

## Step 3: Scale Down

[ kubectl scale statefulset web --replicas=3 ]

Pods deleted in reverse:
- web-4
- web-3

---

## Step 4: Check PVCs

[ kubectl get pvc ]

---

## ✅ Verify

✔ All 5 PVCs still exist  

---

## 🔍 Mentor Insight

Kubernetes keeps PVCs for safety  
👉 Data is never lost accidentally  

---

# 🧹 Task 7: Cleanup

## Step 1: Delete StatefulSet

[ kubectl delete statefulset web ]

## Step 2: Delete Service

[ kubectl delete svc web ]

## Step 3: Check PVCs

[ kubectl get pvc ]

👉 PVCs still exist

## Step 4: Delete PVCs

[ kubectl delete pvc --all ]

---

## ✅ Verify

✔ PVCs are NOT auto-deleted  

---

# 🧠 Final Understanding

## When to Use StatefulSets?

Use StatefulSets when:

✔ You need stable identity  
✔ You need persistent storage  
✔ You need ordered deployment  
✔ You are running databases or clustered apps  

---

## 🔥 Key Takeaways

- StatefulSets solve **stateful application challenges**
- Each pod gets:
  - Unique name
  - Stable DNS
  - Dedicated storage
- Scaling is controlled and predictable
- Data survives pod deletion

---
<img width="797" height="551" alt="task1" src="https://github.com/user-attachments/assets/dd9ddc30-10ad-46c1-bfb5-128d06ac9091" />

<img width="1652" height="823" alt="task3" src="https://github.com/user-attachments/assets/e9fb4fcf-1aed-454a-a60c-d9c495fa0812" />

<img width="616" height="362" alt="task5" src="https://github.com/user-attachments/assets/cbc88e06-3598-484c-a5e3-3c326ed476b9" />

<img width="1316" height="555" alt="task6" src="https://github.com/user-attachments/assets/e1cde689-5827-4d16-8bbb-f6dd78e50e91" />




👉 Next step: Try StatefulSets with MySQL or PostgreSQL

Happy Learning 🚀
