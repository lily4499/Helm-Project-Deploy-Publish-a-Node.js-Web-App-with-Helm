

# 🚀 Deploy & Publish a Node.js App via Helm (`lilihelmapp`)

---

## 🌍 Real-World Scenario

You are a DevOps engineer at a startup deploying a **Node.js app**. You use **Helm** to package the app, manage environments (dev/prod), and publish it on **Artifact Hub** for others to reuse.

---

## 🧱 Step 1: Containerize Your Node.js App

> Purpose: Package your app with all dependencies into a Docker image for consistent deployment.
Dockerfile defines how the image is built.
docker build creates the image.

docker push uploads it to DockerHub so Helm/Kubernetes can pull it.
### a. App Structure
```bash
lilihelmapp/
├── app/
│   ├── app.js
│   ├── package.json
│   ├── Dockerfile
├── charts/              # Subcharts go here
├── templates/           # Kubernetes manifests (templated)
│   ├── deployment.yaml  # Sample app deployment (we'll edit)
│   ├── service.yaml     # Service to expose the app
│   └── NOTES.txt        # Post-install usage notes
├── values.yaml          # Central values file
├── Chart.yaml           # Helm chart metadata
```

### b. Sample `app.js`
```js
const express = require('express');
const app = express();
app.get('/', (req, res) => res.send('Hello from Lili Helm App!'));
app.listen(80);
```

### c. `package.json`
```json
{
  "name": "lilihelmapp",
  "version": "1.0.0",
  "main": "app.js",
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

### d. `Dockerfile`
```Dockerfile
FROM node:18
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 80
CMD ["node", "app.js"]
```

### e. Build and push the image
```bash
docker build -t laly9999/lilihelmapp:1.0.0 ./app
docker push laly9999/lilihelmapp:1.0.0
```

---

## 🎯 Step 2: Create Helm Chart

```bash
helm create lilihelmapp
cd lilihelmapp
rm -f templates/*.yaml
> values.yaml  # Clear file
```

### ✅ Add Templated Files

#### templates/deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.appName }}
  namespace: {{ .Values.namespace }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: {{ .Values.appName }}
  template:
    metadata:
      labels:
        app: {{ .Values.appName }}
    spec:
      containers:
      - name: {{ .Values.appName }}
        image: {{ .Values.image.name }}:{{ .Values.image.tag }}
        ports:
        - containerPort: 80
```

#### templates/service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.appName }}
spec:
  selector:
    app: {{ .Values.appName }}
  ports:
    - port: 80
      targetPort: 80
  type: LoadBalancer
```

#### templates/NOTES.txt
```txt
🚀 Access your application:

Option 1: Port Forward
  kubectl --namespace {{ .Values.namespace }} port-forward service/{{ .Values.appName }} 8080:80

Option 2: LoadBalancer IP
  kubectl get svc --namespace {{ .Values.namespace }}
```

---

## 📄 Step 3: Define Values

### `values.yaml`
```yaml
appName: lilihelmapp
namespace: default
image:
  name: laly9999/lilihelmapp
  tag: "1.0.0"
```

### `values-dev.yaml`
```yaml
appName: lilihelmapp-dev
namespace: dev
image:
  name: laly9999/lilihelmapp
  tag: "dev"
```

### `values-prod.yaml`
```yaml
appName: lilihelmapp-prod
namespace: prod
image:
  name: laly9999/lilihelmapp
  tag: "latest"
```

---

## 🚀 Step 4: Install Helm Release

```bash
helm install lilihelmapp-release lilihelmapp
```

Check pods and service:
```bash
kubectl get pods
kubectl get svc
```

---

## 🔁 Step 5: Upgrade Helm Release

```bash
helm upgrade lilihelmapp-release lilihelmapp --values lilihelmapp/values.yaml
```

---

## 🧪 Step 6: Dev and Prod Namespaces

```bash
kubectl create namespace dev
kubectl create namespace prod
```

```bash
helm install lilihelmapp-release-dev lilihelmapp/ -f lilihelmapp/values.yaml -f lilihelmapp/values-dev.yaml -n dev
helm install lilihelmapp-release-prod lilihelmapp/ -f lilihelmapp/values.yaml -f lilihelmapp/values-prod.yaml -n prod
```

---

## 📦 Step 7: Package Helm Chart

```bash
cd ..
helm package lilihelmapp
mv lilihelmapp-0.1.0.tgz nodewebapp/
cd nodewebapp
helm repo index .
```

---

## 🌐 Step 8: Publish on GitHub and Artifact Hub

### GitHub Pages Setup
```bash
git init
git remote add origin https://github.com/<username>/nodewebapp.git
git add .
git commit -m "Add Helm Chart for lilihelmapp"
git push origin main

git checkout -b gh-pages
git push origin gh-pages
```

### Artifact Hub Setup

- Sign in → Control Panel → Add Repository
- Type: Helm
- Name: lilihelmapp
- URL: `https://<username>.github.io/nodewebapp/`

---

## 🧹 Step 9: Uninstall and Reinstall from Artifact Hub

```bash
helm uninstall lilihelmapp-release
helm repo add lilihelmapp https://<username>.github.io/nodewebapp
helm install lilihelmapp-release lilihelmapp/lilihelmapp
kubectl get svc
```

---

Would you like me to generate a Python script that creates this entire folder structure and file content in `/home/lilia/VIDEOS/lilihelmapp/`?
