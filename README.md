

# 🚀 Deploy & Publish a Node.js App via Helm (`lilihelmapp`)

---

## 🌍 Real-World Scenario

You are a DevOps engineer at a startup deploying a **Node.js app**. You use **Helm** to package the app, manage environments (dev/prod), and publish it on **Artifact Hub** for others to reuse.

---
## Project Structure
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
---

## file-setup.py

```python


```


---

## 🧱 Step 1: Containerize Your Node.js App
> Package your app with all dependencies into a Docker image for consistent deployment.

### a. Sample `app.js`
```js
const express = require('express');
const app = express();
app.get('/', (req, res) => res.send('Hello from Lili Helm App!'));
app.listen(80);
```

### b. `package.json`
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

### c. `Dockerfile`
> Dockerfile defines how the image is built.
```Dockerfile
FROM node:18
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 80
CMD ["node", "app.js"]
```

### d. Build and push the image
> docker build creates the image.
> docker push uploads it to DockerHub so Helm/Kubernetes can pull it.
```bash
docker build -t laly9999/lilihelmapp:1.0.0 ./app
docker push laly9999/lilihelmapp:1.0.0
```

---

## 🎯 Step 2: Create Helm Chart
> Bootstraps a default Helm chart with recommended structure, templates, and values. Saves time setting up.
```bash
helm create lilihelmapp
cd lilihelmapp
rm -f templates/*.yaml
> values.yaml  # Clear file
```

### ✅ Add Templated Files
> Replaces generic nginx manifests with your custom app’s Kubernetes manifests, e.g., for your Node.js app.

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
> Give users helpful info after installation (e.g., how to access the app).
> Automatically printed by Helm after install/upgrade.

```txt
🚀 Access your application:

Option 1: Port Forward
  kubectl --namespace {{ .Values.namespace }} port-forward service/{{ .Values.appName }} 8080:80

Option 2: LoadBalancer IP
  kubectl get svc --namespace {{ .Values.namespace }}
```

---

## 📄 Step 3: Define Values
> Store config for different environments in a structured, version-controlled way.

### `values.yaml`   # Default/base values
```yaml
appName: lilihelmapp
namespace: default
image:
  name: laly9999/lilihelmapp
  tag: "1.0.0"
```
Use these in templates like:
```yaml
{{ .Values.appName }}
{{ .Values.namespace }}
{{ .Values.image.name }}:{{ .Values.image.tag }}

```

### `values-dev.yaml`   #Overrides for the dev namespace
```yaml
appName: lilihelmapp-dev   
namespace: dev
image:
  name: laly9999/lilihelmapp
  tag: "dev"
```

### `values-prod.yaml`   #Overrides for the prod namespace
```yaml
appName: lilihelmapp-prod
namespace: prod
image:
  name: laly9999/lilihelmapp
  tag: "latest"
```

---

## 🚀 Step 4: Install Helm Release
> Deploy the chart to Kubernetes for the first time.
> Helm renders templates using values and applies them via kubectl.
> Your app is deployed in its namespace with appropriate configs.


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
> Apply updates to your app when templates or values change.
> Avoids deleting and re-installing.
> Helm compares the current state with desired and patches the resources.
```bash
helm upgrade lilihelmapp-release lilihelmapp --values lilihelmapp/values.yaml
```

---

## 🧪 Step 6: Dev and Prod Namespaces
>Isolate environments (dev, prod) in separate namespaces.
>Ensures resource separation.
 Prevents dev changes from affecting prod workloads.
```bash
kubectl create namespace dev
kubectl create namespace prod
```
Install releases:
> Deploy the same chart in different environments with different values.
> Promotes reusability and consistency across environments.
> No need to duplicate YAML files.
```bash
helm install lilihelmapp-release-dev lilihelmapp/ -f lilihelmapp/values.yaml -f lilihelmapp/values-dev.yaml -n dev
helm install lilihelmapp-release-prod lilihelmapp/ -f lilihelmapp/values.yaml -f lilihelmapp/values-prod.yaml -n prod
```
Check all:
```bash
helm ls -A
```
Rollback:
> Roll back to a previous release version if something goes wrong.
> Ensures quick recovery.
> Helm tracks revision history automatically.
```bash
helm rollback  lilihelmapp-release-dev 1

```
---
---
--------------------------------------------------Publish to Artifact Hub-------------------------------------------

## 📦 Step 7: Package Helm Chart

###  Create external directory
> You need a clean repo directory for hosting.

```bash
mkdir nodewebapp && cd nodewebapp
pwd  # Note this path
> Bundle your chart into a .tgz archive for distribution.
> Required for publishing to Artifact Hub or GitHub Pages.
> Can be versioned and reused.
```bash
cd ..
helm package lilihelmapp
mv lilihelmapp-0.1.0.tgz nodewebapp/

### Create index.yaml
> helm repo index generates index.yaml relative to this folder.
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
