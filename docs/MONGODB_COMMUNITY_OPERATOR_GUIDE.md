# MongoDB Community Kubernetes Operator Guide for forms-flow-ai

This guide details how to install the **MongoDB Community Kubernetes Operator** into your namespace (e.g., `test`) on AWS EKS and deploy **forms-flow-ai** using official or custom MongoDB images without relying on Bitnami.

---

## 📌 Summary of Chart Modifications

The following changes were made to the `forms-flow-ai-charts` repository to replace Bitnami MongoDB with the Operator:

| Component / File | Modification Description |
| :--- | :--- |
| **`charts/forms-flow-ai/Chart.yaml`** | **REMOVED**: Completely removed the Bitnami `mongodb` chart dependency. |
| **`charts/forms-flow-ai/templates/mongodb-community.yaml`** | **NEW TEMPLATE**: Automatically deploys the `MongoDBCommunity` Custom Resource (`apiVersion: mongodbcommunity.mongodb.com/v1`) and user credential secret when `mongodbCommunity.enabled=true`. |
| **`charts/forms-flow-ai/templates/secrets.yaml`** | **UPDATED**: Dynamically builds the multi-member ReplicaSet URI (`mongodb://user:pass@pod-0.svc,pod-1.svc,pod-2.svc/formsflow?replicaSet=...&authSource=admin`) for `NODE_CONFIG` (Formio) and `MONGODB_URI` (WebAPI & Data Layer). Removed legacy Bitnami fallback. |
| **`charts/forms-flow-ai/templates/configmap.yaml`** | **UPDATED**: Cleans up `FORMIO_DB_PORT` to read `mongodbCommunity.port` (default 27017). |
| **`charts/forms-flow-ai/values.yaml`** | **PURGED & REPLACED**: Completely removed the 160+ line legacy Bitnami `mongodb:` block (including `bitnamilegacy/mongodb` images, arbiter, etc.) and replaced with pure `mongodbCommunity:` configuration. |
| **`charts/forms-flow-ai/README.md`** | **UPDATED**: Replaced old Bitnami parameter documentation with `mongodbCommunity` configuration table. |
| **`scripts/install-forms-flow.sh`** | **UPDATED**: Removed obsolete Bitnami flag `--set mongodb.podSecurityContext.enabled=true`. |
| **Subcharts (`forms-flow-forms`, `forms-flow-api`, `forms-flow-data-layer`)** | **NO CHANGES NEEDED**: These components seamlessly consume database configuration from the parent `forms-flow-ai` secret/configmap. |

---

## 🚀 Step 1: Install MongoDB Community Operator in Namespace `test`

Run these commands on your cluster to install the operator scoped to namespace `test`:

### 1.1 Create Namespace
```bash
kubectl create namespace test
```

### 1.2 Add MongoDB Helm Repository
```bash
helm repo add mongodb https://mongodb.github.io/helm-charts
helm repo update
```

### 1.3 Install the Operator in Namespace `test`
```bash
helm upgrade --install mongodb-operator mongodb/community-operator \
  --namespace test \
  --set operator.watchNamespace="test" \
  --wait
```

### 1.4 Verify Operator Pod is Running
```bash
kubectl get pods -n test -l app.kubernetes.io/name=community-operator
```

---

## ⚙️ Step 2: Configure Your Database Settings in `values.yaml`

In `charts/forms-flow-ai/values.yaml`, the `mongodbCommunity` block allows you to customize your database:

```yaml
mongodbCommunity:
  enabled: true
  name: "forms-flow-mongodb"
  
  # Specify the MongoDB version (e.g. "7.0.14", "6.0.19", "8.0.0")
  version: "7.0.14"
  
  # Number of members (3 for HA ReplicaSet, 1 for single-node dev)
  members: 3
  type: ReplicaSet
  port: 27017

  auth:
    database: "formsflow"
    username: "mongodb"
    password: "changeme"
    authSource: "admin"
    existingSecretName: ""
    existingSecretKey: "password"

  # Storage configuration (gp3 on AWS EKS)
  storage:
    storageClassName: "gp3" # Leave empty to use cluster default
    size: "8Gi"

  resources:
    requests:
      cpu: "500m"
      memory: "1Gi"
    limits:
      cpu: "2"
      memory: "2Gi"

  # (Optional) If you have an in-house hardened image in your private registry
  customImage:
    enabled: false
    image: "<your-ecr-or-registry-url>/custom-mongodb:7.0.14"
    imagePullSecrets:
      - name: "custom-mongo-auth"
```

---

## 🚢 Step 3: Deploy `forms-flow-ai`

Deploy the chart into the `test` namespace:

```bash
helm upgrade --install forms-flow-ai ./charts/forms-flow-ai \
  --namespace test \
  --values ./charts/forms-flow-ai/values.yaml
```

If you wish to override values dynamically during install (e.g., custom domain or credentials):
```bash
helm upgrade --install forms-flow-ai ./charts/forms-flow-ai \
  --namespace test \
  --set Domain="your-domain.com" \
  --set mongodbCommunity.version="7.0.14" \
  --set mongodbCommunity.auth.password="YourSecurePassword123"
```

---

## 🔍 Step 4: Verification & Troubleshooting

### 4.1 Verify MongoDB ReplicaSet Pods
Check that the operator created the database pods and services:
```bash
# Check MongoDB pods
kubectl get pods -n test -l app=forms-flow-mongodb-svc

# Check MongoDB service
kubectl get svc -n test forms-flow-mongodb-svc
```

### 4.2 Verify Database Status in Custom Resource
```bash
kubectl get mongodbcommunity -n test
kubectl describe mongodbcommunity forms-flow-mongodb -n test
```
The status should transition to `Phase: Running`.

### 4.3 Verify the Secret Rendered for Microservices
Check that `forms-flow-ai` secret has the correct connection string:
```bash
# View MONGODB_URI
kubectl get secret forms-flow-ai -n test -o jsonpath='{.data.MONGODB_URI}' | base64 -d

# View NODE_CONFIG
kubectl get secret forms-flow-ai -n test -o jsonpath='{.data.NODE_CONFIG}' | base64 -d
```
Expected output:
```text
mongodb://mongodb:changeme@forms-flow-mongodb-0.forms-flow-mongodb-svc.test.svc.cluster.local:27017,forms-flow-mongodb-1.forms-flow-mongodb-svc.test.svc.cluster.local:27017,forms-flow-mongodb-2.forms-flow-mongodb-svc.test.svc.cluster.local:27017/formsflow?replicaSet=forms-flow-mongodb&authSource=admin
```

### 4.4 Check Microservice Logs
```bash
# Verify Formio (forms-flow-forms)
kubectl logs -n test -l app.kubernetes.io/name=forms-flow-forms --tail=50

# Verify WebAPI (forms-flow-api)
kubectl logs -n test -l app.kubernetes.io/name=forms-flow-api --tail=50
```

---

## 🔄 How to Upgrade MongoDB Version Later

To upgrade MongoDB in the future:
1. Update `mongodbCommunity.version` in `values.yaml` (e.g. from `"7.0.14"` to `"8.0.0"`).
2. Run `helm upgrade forms-flow-ai ./charts/forms-flow-ai -n test`.
3. The operator automatically initiates a **rolling upgrade** of the replica set nodes without downtime.
