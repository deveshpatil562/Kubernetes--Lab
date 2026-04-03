# 🚀 Kubernetes Capstone Lab: Deploy WordPress + MySQL

## 📌 Overview

This capstone brings together everything you’ve learned so far in Kubernetes.

You will deploy a **real-world application stack**:
- WordPress (Frontend)
- MySQL (Database)

And apply **12 core Kubernetes concepts**:
- Namespace
- Secrets
- ConfigMaps
- StatefulSets
- Deployments
- Services (Headless + NodePort)
- Persistent Volumes (PVC)
- Resource Management
- Probes (Liveness & Readiness)
- Autoscaling (HPA)
- Helm (comparison)

---

## 🧠 Architecture Understanding

- WordPress connects to MySQL using internal DNS
- MySQL runs as a **StatefulSet** with persistent storage
- WordPress runs as a **Deployment** with replicas
- Secrets store sensitive DB credentials
- ConfigMap stores DB connection config
- Services expose applications internally and externally

---

# 🧩 Task 1: Create Namespace

## ✅ Step

[ kubectl create namespace capstone ]

[ kubectl config set-context --current --namespace=capstone ]

## 🔍 Verify

[ kubectl get ns ]

✔ You should see `capstone` listed

---

# 🧩 Task 2: Deploy MySQL (StatefulSet + Storage)

## 🧠 Concept

- StatefulSet ensures stable identity
- Headless Service enables DNS resolution
- PVC ensures data persistence

---

## 🔐 Create Secret

[ 
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
stringData:
  MYSQL_ROOT_PASSWORD: rootpass
  MYSQL_DATABASE: wordpress
  MYSQL_USER: wpuser
  MYSQL_PASSWORD: wppass
]

[ kubectl apply -f mysql-secret.yaml ]

---

## 🌐 Headless Service

[
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - port: 3306
]

---

## 🗄 StatefulSet

[
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        envFrom:
        - secretRef:
            name: mysql-secret
        resources:
          requests:
            cpu: 250m
            memory: 512Mi
          limits:
            cpu: 500m
            memory: 1Gi
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
]

---

## 🔍 Verify MySQL

[ kubectl get pods ]

[ kubectl exec -it mysql-0 -- mysql -u wpuser -pwppass -e "SHOW DATABASES;" ]

✔ You should see `wordpress` database


---

# 🧩 Task 3: Deploy WordPress

## 🧠 Concept

- Deployment manages stateless apps
- ConfigMap for config
- Secret for credentials

---

## 📦 ConfigMap

[
apiVersion: v1
kind: ConfigMap
metadata:
  name: wordpress-config
data:
  WORDPRESS_DB_HOST: mysql-0.mysql.capstone.svc.cluster.local:3306
  WORDPRESS_DB_NAME: wordpress
]

---

## 🚀 Deployment

[
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: wordpress
        image: wordpress:latest
        envFrom:
        - configMapRef:
            name: wordpress-config
        env:
        - name: WORDPRESS_DB_USER
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_USER
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_PASSWORD
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /wp-login.php
            port: 80
          initialDelaySeconds: 30
        readinessProbe:
          httpGet:
            path: /wp-login.php
            port: 80
          initialDelaySeconds: 15
]

---

## 🔍 Verify

[ kubectl get pods ]

✔ Both pods should show `1/1 Running`


---

# 🧩 Task 4: Expose WordPress

## 🌐 NodePort Service

[
apiVersion: v1
kind: Service
metadata:
  name: wordpress
spec:
  type: NodePort
  selector:
    app: wordpress
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
]

---

## 🌍 Access

Minikube:
[ minikube service wordpress -n capstone ]

Kind:
[ kubectl port-forward svc/wordpress 8080:80 ]

✔ Open browser → WordPress setup page


---

# 🧩 Task 5: Self-Healing & Persistence

## 🔁 Test Self-Healing

[ kubectl delete pod <wordpress-pod> ]

✔ Pod recreated automatically

---

## 🗄 Test Persistence

[ kubectl delete pod mysql-0 ]

✔ StatefulSet recreates pod

✔ Data remains intact

👉 Refresh site → Blog still exists

---

# 🧩 Task 6: HPA (Autoscaling)

[
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: wordpress-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: wordpress
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
]

---

## 🔍 Verify

[ kubectl get hpa ]

✔ Check min/max and CPU target

---

# 🧩 Task 7: Helm Comparison

## 🚀 Install via Helm

[ helm install wp-helm bitnami/wordpress -n helm-test --create-namespace ]

## 🔍 Compare

[ kubectl get all -n capstone ]

[ kubectl get all -n helm-test ]

### 💡 Mentor Insight

- Helm = Fast & automated
- Manual = Deep control + learning

---

## 🧹 Cleanup Helm

[ helm uninstall wp-helm -n helm-test ]

---

# 🧩 Task 8: Cleanup

## 🔍 Final Check

[ kubectl get all -n capstone ]

---

## 🧹 Delete Everything

[ kubectl delete namespace capstone ]

[ kubectl config set-context --current --namespace=default ]

✔ Everything removed automatically

---

# 📊 Concept Mapping

| Concept            | Used In Task |
|------------------|-------------|
| Namespace         | Task 1 |
| Secret            | Task 2 |
| ConfigMap         | Task 3 |
| StatefulSet       | Task 2 |
| Deployment        | Task 3 |
| Service           | Task 2,4 |
| PVC               | Task 2 |
| Resource Limits   | Task 2,3 |
| Probes            | Task 3 |
| HPA               | Task 6 |
| Helm              | Task 7 |

---

# 🧠 Final Reflection (Mentor Notes)

### 🔥 What Was Hard
- StatefulSet DNS & storage concepts
- Debugging MySQL startup delays

### 💡 What Clicked
- How Kubernetes components connect together
- Real-world architecture thinking

### 🚀 What To Improve (Production)
- Use Ingress instead of NodePort
- Add TLS (HTTPS)
- Use external managed database
- Add monitoring (Prometheus + Grafana)

---

# 🎯 Final Result

✔ Full WordPress + MySQL deployed  
✔ Self-healing verified  
✔ Persistent storage working  
✔ Autoscaling configured  

---

📸 Final Screenshots:

<img width="1635" height="677" alt="Screenshot 2026-04-03 115542" src="https://github.com/user-attachments/assets/e68da849-b742-4287-a5e1-b0439b1b4d8e" />

<img width="782" height="952" alt="Screenshot 2026-04-03 115633" src="https://github.com/user-attachments/assets/47bcfe32-88e9-47ab-a290-76405e7b681e" />

<img width="623" height="947" alt="Screenshot 2026-04-03 115657" src="https://github.com/user-attachments/assets/ff3f5914-4da6-4dee-8e3d-7dfe13180a7b" />

<img width="1918" height="931" alt="task1" src="https://github.com/user-attachments/assets/df68fa06-63e7-4e09-b876-74c5bcf4b706" />

---

