# 🚀 Kubernetes ConfigMaps and Secrets Lab Guide

## 📌 Overview

In real-world applications, configuration changes frequently — things like database URLs, ports, feature flags, and API keys. Hardcoding these into container images is a bad practice because it forces you to rebuild and redeploy images for small changes.

Kubernetes solves this using:

### 🔹 ConfigMaps
- Used for **non-sensitive configuration**
- Stores data as **plain text**
- Ideal for:
  - Environment variables
  - Config files (e.g., nginx.conf)

### 🔐 Secrets
- Used for **sensitive data**
- Values are stored in **base64 encoded format**
- NOT encrypted by default
- Ideal for:
  - Passwords
  - API keys
  - Tokens

---

## ⚖️ ConfigMaps vs Secrets

| Feature        | ConfigMap          | Secret              |
|----------------|-------------------|---------------------|
| Data type      | Plain text         | Base64 encoded      |
| Use case       | Non-sensitive      | Sensitive data      |
| Security       | Low                | Medium (RBAC based) |

---

## 🔄 Env Vars vs Volume Mounts

| Method           | Behavior |
|------------------|----------|
| Environment Vars | Loaded once at pod startup |
| Volume Mounts    | Auto-updates when source changes |

---

# 🧪 LAB TASKS

---

## 🧩 Task 1: Create ConfigMap from Literals

### 🔹 Step

[ kubectl create configmap app-config \
--from-literal=APP_ENV=production \
--from-literal=APP_DEBUG=false \
--from-literal=APP_PORT=8080 ]

### 🔍 Inspect

[ kubectl describe configmap app-config ]

[ kubectl get configmap app-config -o yaml ]

### ✅ Verification
- Check if all 3 keys exist
- Notice values are plain text

### 📸 Screenshot Placeholder

<img width="1276" height="715" alt="task1" src="https://github.com/user-attachments/assets/51bf5741-0865-4e28-9283-9a860c087bb7" />


### 🧠 Mentor Note
This is the fastest way to inject simple key-value configurations.

---

## 📁 Task 2: Create ConfigMap from File

### 🔹 Step 1: Create nginx config file

[ echo '
server {
    listen 80;
    location /health {
        return 200 "healthy";
    }
}
' > default.conf ]

### 🔹 Step 2: Create ConfigMap

[ kubectl create configmap nginx-config --from-file=default.conf=default.conf ]

### 🔍 Inspect

[ kubectl get configmap nginx-config -o yaml ]

### ✅ Verification
- File content should appear under data

### 📸 Screenshot Placeholder

<img width="1597" height="665" alt="task2" src="https://github.com/user-attachments/assets/5e9d4fec-3ea9-49f5-afd5-146830cca7e5" />


### 🧠 Mentor Note
Kubernetes treats file key as filename when mounted.

---

## 🚀 Task 3: Use ConfigMaps in Pods

### 🔹 Pod 1: Env Variables Injection

[ 
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: [ "sh", "-c", "env && sleep 3600" ]
    envFrom:
    - configMapRef:
        name: app-config
]

### 🔹 Apply

[ kubectl apply -f env-pod.yaml ]

### 🔍 Verify

[ kubectl exec env-pod -- printenv ]

---

### 🔹 Pod 2: Mount ConfigMap as Volume

[
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/nginx/conf.d
  volumes:
  - name: config-volume
    configMap:
      name: nginx-config
]

### 🔹 Apply

[ kubectl apply -f nginx-pod.yaml ]

### 🔍 Test

[ kubectl exec nginx-pod -- curl -s http://localhost/health ]

### ✅ Verification
- Response should be: healthy

### 📸 Screenshot Placeholder

<img width="882" height="573" alt="task3 1" src="https://github.com/user-attachments/assets/e138481f-65d6-4578-965d-854534f3814b" />

<img width="1476" height="797" alt="task3 2" src="https://github.com/user-attachments/assets/69884520-116f-47f0-8c42-b36ed59c6f88" />


### 🧠 Mentor Note
- Use env vars for simple configs
- Use volumes for full config files

---

## 🔐 Task 4: Create a Secret

### 🔹 Step

[ kubectl create secret generic db-credentials \
--from-literal=DB_USER=admin \
--from-literal=DB_PASSWORD=s3cureP@ssw0rd ]

### 🔍 Inspect

[ kubectl get secret db-credentials -o yaml ]

### 🔹 Decode

[ echo '<base64-value>' | base64 --decode ]

### ✅ Verification
- Password should decode correctly

### 📸 Screenshot Placeholder

<img width="1101" height="423" alt="task4" src="https://github.com/user-attachments/assets/0f0cd4f5-afce-47fa-9c6b-98c3010b792b" />


### 🧠 Mentor Note
Base64 ≠ encryption. It's just encoding.

---

## 🔑 Task 5: Use Secrets in Pod

### 🔹 Pod Manifest

[
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: [ "sh", "-c", "env && sleep 3600" ]
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: DB_USER
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/db-credentials
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: db-credentials
]

### 🔹 Apply

[ kubectl apply -f secret-pod.yaml ]

### 🔍 Verify

[ kubectl exec secret-pod -- ls /etc/db-credentials ]

[ kubectl exec secret-pod -- cat /etc/db-credentials/DB_PASSWORD ]

### ✅ Verification
- Files contain plaintext values

### 📸 Screenshot Placeholder

<img width="1650" height="562" alt="task5" src="https://github.com/user-attachments/assets/ae5a27f9-6fae-493e-882e-cadcb6b0d4ab" />


### 🧠 Mentor Note
Secrets are decoded automatically when mounted.

---

## 🔄 Task 6: ConfigMap Live Update

### 🔹 Create ConfigMap

[ kubectl create configmap live-config --from-literal=message=hello ]

### 🔹 Pod Manifest

[
apiVersion: v1
kind: Pod
metadata:
  name: live-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: [ "sh", "-c", "while true; do cat /config/message; sleep 5; done" ]
    volumeMounts:
    - name: config-volume
      mountPath: /config
  volumes:
  - name: config-volume
    configMap:
      name: live-config
]

### 🔹 Apply

[ kubectl apply -f live-pod.yaml ]

### 🔹 Update ConfigMap

[ kubectl patch configmap live-config --type merge -p '{"data":{"message":"world"}}' ]

### ✅ Verification
- Output changes from "hello" → "world" automatically

### 📸 Screenshot Placeholder

<img width="1618" height="646" alt="task6" src="https://github.com/user-attachments/assets/f6041b4a-9e69-4b4e-844b-e522f2a5f846" />


### 🧠 Mentor Note
- Volume updates automatically
- Env vars DO NOT update

---

## 🧹 Task 7: Cleanup

[ kubectl delete pod env-pod nginx-pod secret-pod live-pod ]

[ kubectl delete configmap app-config nginx-config live-config ]

[ kubectl delete secret db-credentials ]

---

# 📘 Documentation Summary

## ✅ When to Use What

- ConfigMap → Non-sensitive configs
- Secret → Sensitive data

## 🔁 Key Differences

- Env vars → Static
- Volume mounts → Dynamic updates

## 🔐 Base64 Reality

- Base64 is encoding, NOT encryption
- Anyone with access can decode it

## 🔄 Update Behavior

- ConfigMap updates reflect in volumes
- NOT reflected in env variables

---


---

🔥 Keep going. This is exactly how real-world Kubernetes configuration is managed.
