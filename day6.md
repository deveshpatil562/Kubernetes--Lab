# 🚀 Kubernetes Lab Guide — Persistent Volumes (PV) & Persistent Volume Claims (PVC)

---

## 📌 Introduction (Mentor Explanation)

Containers are **ephemeral by design**. This means:
- When a Pod is deleted → all data inside it is lost
- This is fine for stateless apps, but **dangerous for databases, logs, uploads**

👉 Kubernetes solves this using:
- **PersistentVolume (PV)** → Actual storage (disk)
- **PersistentVolumeClaim (PVC)** → Request for storage
- **StorageClass** → Automates storage provisioning

---

## 🧠 Core Concepts

### 🔹 Persistent Volume (PV)
- Cluster-wide storage resource
- Created by admin or dynamically
- Lifecycle:
  Available → Bound → Released

### 🔹 Persistent Volume Claim (PVC)
- A request for storage by a Pod
- Namespaced
- Binds to matching PV

### 🔹 Access Modes
- RWO → ReadWriteOnce (single node)
- ROX → ReadOnlyMany
- RWX → ReadWriteMany

### 🔹 Reclaim Policies
- Retain → Keeps data after deletion
- Delete → Deletes storage automatically

---

# 🧪 TASK 1 — Problem: Data Loss (emptyDir)

## 📌 Concept
emptyDir is temporary storage:
- Exists only during Pod lifecycle
- Destroyed when Pod dies

---

## 🛠 Step 1: Create Pod

[apiVersion: v1
kind: Pod
metadata:
  name: ephemeral-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "while true; do date >> /data/message.txt; sleep 5; done"]
    volumeMounts:
    - name: temp-storage
      mountPath: /data
  volumes:
  - name: temp-storage
    emptyDir: {}]

---

## ▶️ Apply

[kubectl apply -f pod.yaml]

---

## 🔍 Verify Data

[kubectl exec -it ephemeral-pod -- cat /data/message.txt]

📸 Screenshot Placeholder:

<img width="1638" height="852" alt="task1" src="https://github.com/user-attachments/assets/aa69c377-f7c6-4b98-a5ec-011bb67b8a3c" />


---

## 🔄 Delete & Recreate Pod

[kubectl delete pod ephemeral-pod]

Reapply manifest

---

## 🔍 Verify Again

[kubectl exec -it ephemeral-pod -- cat /data/message.txt]

---

## ✅ Result (Mentor Insight)
- Old data is gone ❌
- New timestamps appear

👉 **Answer:** Timestamp is DIFFERENT

---

# 🧪 TASK 2 — Create PersistentVolume (Static)

## 📌 Concept
Static provisioning = Admin creates PV manually

---

## 🛠 PV Manifest

[apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-static-demo
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /tmp/k8s-pv-data]

---

## ▶️ Apply

[kubectl apply -f pv.yaml]

---

## 🔍 Verify

[kubectl get pv]

📸 Screenshot Placeholder:

<img width="1673" height="701" alt="task2" src="https://github.com/user-attachments/assets/9650befa-1ead-4910-a654-bab8bb259e75" />


---

## ✅ Result
👉 STATUS = Available

---

# 🧪 TASK 3 — Create PVC

## 📌 Concept
PVC requests storage from available PV

---

## 🛠 PVC Manifest

[apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-demo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi]

---

## ▶️ Apply

[kubectl apply -f pvc.yaml]

---

## 🔍 Verify

[kubectl get pvc]
[kubectl get pv]

📸 Screenshot Placeholder:

<img width="1643" height="640" alt="task3" src="https://github.com/user-attachments/assets/5a1862c1-201d-4ddd-9767-5846a47e983f" />


---

## ✅ Result
- PVC → Bound
- PV → Bound

👉 **Answer:** VOLUME column shows: pv-static-demo

---

# 🧪 TASK 4 — Use PVC in Pod

## 📌 Concept
Now storage survives Pod lifecycle

---

## 🛠 Pod Manifest

[apiVersion: v1
kind: Pod
metadata:
  name: persistent-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "while true; do date >> /data/message.txt; sleep 5; done"]
    volumeMounts:
    - name: persistent-storage
      mountPath: /data
  volumes:
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: pvc-demo]

---

## ▶️ Apply

[kubectl apply -f pod-pvc.yaml]

---

## 🔍 Verify Data

[kubectl exec -it persistent-pod -- cat /data/message.txt]

---

## 🔄 Delete & Recreate Pod

[kubectl delete pod persistent-pod]

Reapply manifest

---

## 🔍 Verify Again

[kubectl exec -it persistent-pod -- cat /data/message.txt]

📸 Screenshot Placeholder:

<img width="1610" height="912" alt="task4" src="https://github.com/user-attachments/assets/232ebf1b-29cd-4b44-a031-a8de21512c84" />


---

## ✅ Result (Mentor Insight)
- Data persists ✅
- File contains old + new entries

👉 **Answer:** YES — data from both Pods exists

---

# 🧪 TASK 5 — StorageClasses

## 📌 Concept
StorageClass = Dynamic provisioning template

---

## ▶️ Check StorageClasses

[kubectl get storageclass]
[kubectl describe storageclass]

---

## 🔍 Observe
- provisioner
- reclaim policy
- volumeBindingMode

---

## ✅ Result
👉 Identify default StorageClass (marked with “(default)”)

---

# 🧪 TASK 6 — Dynamic Provisioning

## 📌 Concept
PVC automatically creates PV

---

## 🛠 Dynamic PVC

[apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-dynamic-demo
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 500Mi]

---

## ▶️ Apply

[kubectl apply -f pvc-dynamic.yaml]

---

## ⚠️ Important (Mentor Tip)
If stuck in Pending:
👉 StorageClass uses WaitForFirstConsumer

---

## 🛠 Create Pod

[apiVersion: v1
kind: Pod
metadata:
  name: dynamic-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "echo Dynamic Storage Working >> /data/message.txt && sleep 3600"]
    volumeMounts:
    - name: storage
      mountPath: /data
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-dynamic-demo]

---

## ▶️ Apply

[kubectl apply -f pod-dynamic.yaml]

---

## 🔍 Verify

[kubectl get pv]

📸 Screenshot Placeholder:

<img width="1662" height="967" alt="task6" src="https://github.com/user-attachments/assets/45ab5093-dafd-4baa-9546-04a267249eb1" />


---

## ✅ Result
👉 New PV created automatically

👉 **Answer:**
- Total PVs = 2
- Manual → pv-static-demo
- Dynamic → auto-created PV

---

# 🧪 TASK 7 — Cleanup

---

## ▶️ Delete Pods

[kubectl delete pod --all]

---

## ▶️ Delete PVCs

[kubectl delete pvc --all]

---

## 🔍 Check PVs

[kubectl get pv]

---

## 🧠 Expected Behavior

- Dynamic PV → Deleted automatically ✅
- Static PV → Released state ❗

---

## 🛠 Manual Cleanup

[kubectl delete pv pv-static-demo]

---

## ✅ Final Answer

👉 Dynamic PV deleted because:
- ReclaimPolicy = Delete

👉 Static PV retained because:
- ReclaimPolicy = Retain

---

# ⚠️ Errors You Faced (Mentor Breakdown)

### ❌ PVC Immutability
- Cannot edit storageClassName after creation
✔ Fix → Delete & recreate PVC

---

### ❌ Patch Formatting Error
✔ Correct command:

[kubectl patch pv pv-static-demo -p '{"spec":{"claimRef":null}}']

---

### ❌ PVC Terminating Issue
- Caused by Retain policy
✔ Fix → Remove claimRef manually

---

### ❌ PVC Pending (Dynamic)
- Due to WaitForFirstConsumer
✔ Fix → Create Pod

---

# 🧾 Documentation Section (For GitHub)

## 📌 Why Persistent Storage?
- Containers lose data on restart
- Required for:
  - Databases
  - Logs
  - File uploads

---

## 📌 PV vs PVC

| Component | Role |
|----------|------|
| PV | Actual storage |
| PVC | Request for storage |

---

## 📌 Static vs Dynamic

| Type | Description |
|------|------------|
| Static | Admin creates PV |
| Dynamic | Auto via StorageClass |

---

## 📌 Key Learnings

- Data is ephemeral by default
- PVC binds based on:
  - Capacity
  - Access mode
- Reclaim policies control lifecycle
- Dynamic provisioning simplifies storage

---


---

🔥 Great job — this is a **core Kubernetes concept**. Master this and you're already ahead of many beginners.
