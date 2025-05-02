## Deploying Nextcloud with PostgreSQL on Kubernetes

This lab guide walks you through deploying Nextcloud with a PostgreSQL backend on Kubernetes.

🏗️ Prerequisites

✅ A running Kubernetes cluster (minikube, kind, EKS, GKE, etc.)✅ kubectl configured for your cluster✅ (Optional) Ingress controller installed if you plan to use Ingress

### Step 1: Create Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nextcloud
```
Apply:
```
kubectl apply -f namespace.yaml
```
### Step 2: Create Persistent Volume Claims (PVCs)

Nextcloud PVC:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nextcloud-pvc
  namespace: nextcloud
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```
PostgreSQL PVC:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: nextcloud
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```
Apply:
```
kubectl apply -f nextcloud-pvc.yaml
```
```
kubectl apply -f postgres-pvc.yaml
```

### Step 3: Deploy PostgreSQL

Deployment:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: nextcloud
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:15
          env:
            - name: POSTGRES_DB
              value: nextcloud
            - name: POSTGRES_USER
              value: nextcloud
            - name: POSTGRES_PASSWORD
              value: nextcloudpass
          ports:
            - containerPort: 5432
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: postgres-storage
      volumes:
        - name: postgres-storage
          persistentVolumeClaim:
            claimName: postgres-pvc
```
Service:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: nextcloud
spec:
  ports:
    - port: 5432
      targetPort: 5432
  selector:
    app: postgres
  type: ClusterIP
```
Apply:
```
kubectl apply -f postgres-deployment.yaml
```
```
kubectl apply -f postgres-service.yaml
```

### Step 4: Deploy Nextcloud

Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nextcloud
  namespace: nextcloud
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nextcloud
  template:
    metadata:
      labels:
        app: nextcloud
    spec:
      containers:
        - name: nextcloud
          image: nextcloud:27
          env:
            - name: POSTGRES_HOST
              value: postgres
            - name: POSTGRES_DB
              value: nextcloud
            - name: POSTGRES_USER
              value: nextcloud
            - name: POSTGRES_PASSWORD
              value: nextcloudpass
          ports:
            - containerPort: 80
          volumeMounts:
            - mountPath: /var/www/html
              name: nextcloud-storage
      volumes:
        - name: nextcloud-storage
          persistentVolumeClaim:
            claimName: nextcloud-pvc
```
Service:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nextcloud
  namespace: nextcloud
spec:
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: nextcloud
  type: ClusterIP
```
Apply:
```
kubectl apply -f nextcloud-deployment.yaml
```
```
kubectl apply -f nextcloud-service.yaml
```

### Step 5: (Optional) Expose Nextcloud Externally

Option 1: NodePort
```yaml
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```
Option 2: Ingress (Recommended for Production)

Make sure you have an ingress controller installed (like nginx).
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nextcloud-ingress
  namespace: nextcloud
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: nextcloud.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nextcloud
                port:
                  number: 80
```
Apply:
```
kubectl apply -f nextcloud-ingress.yaml
```

### Step 6: Access Nextcloud

✅ If using NodePort:Access via: http://<NodeIP>:30080

✅ If using Ingress:Configure DNS to point nextcloud.example.com to your ingress IP, then access: http://nextcloud.example.com

⚠️ Notes

Change all passwords (POSTGRES_PASSWORD) to secure values before production.

Use Kubernetes Secrets instead of plain environment variables for sensitive data.

For production storage, use proper dynamic provisioners (like AWS EBS, GCP PD, or NFS).

Add TLS to your Ingress for HTTPS access.
