# Node Todo App – CI/CD with Helm

This project is a Node.js-based To-Do application integrated with Docker, GitHub Actions for CI/CD, and Kubernetes deployment via Helm.

The [`nodetodo`](https://github.com/atmikvirani/nodetodocicd-helmchart) folder contains a custom Helm chart designed to manage the Kubernetes deployment process.

Developed and deployed with ❤️ — by [**Atmik Virani**](https://github.com/atmikvirani)

## 🚀 Helm Chart Deployment for Node TODO App

This guide walks through deploying the `Node TODO App` on Kubernetes using Helm, enabling autoscaling, and generating load to trigger Horizontal Pod Autoscaler (HPA).

### 🔧 Pre-Cloning Instructions
After cloning this repository, rename the project folder to `nodetodo` to match Helm chart expectations:

```bash
git clone https://github.com/atmikvirani/nodetodocicd-helmchart.git
mv nodetodocicd-helmchart nodetodo
```

#

### 🛠️ Prerequisites

- Kubernetes cluster (e.g. Minikube or kind)
- Helm installed
- `kubectl` configured to point to your cluster

---

### 🌐 Optional: Ingress Setup

To expose your Node TODO app via an Ingress resource, follow these steps:

1. **Enable Ingress in `values.yaml`** inside the `nodetodo` folder:

```yaml
ingress:
  enabled: true
  className: nginx
  hosts:
    - host: nodetodo.local
      paths:
        - path: /
          pathType: Prefix
```

2. **Install NGINX Ingress Controller** on your Kubernetes cluster (if not already installed):

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx
```

3. **Update your local `/etc/hosts` file** (for local clusters like Minikube or kind) to map the host to your cluster IP, for example:

```
127.0.0.1 nodetodo.local
```

4. **Deploy or upgrade your Helm release** to apply ingress settings:

```bash
helm upgrade --install nodetodo ./nodetodo -f ./nodetodo/values.yaml -n hc
```

5. **Access your app** in the browser at:

```
http://nodetodo.local
```

---

### 📁 Project Structure
```
node-todo-cicd
├── app.js
├── azure-pipelines.yml
├── DevSecOps
│   ├── Jenkinsfile
│   └── README.md
├── docker-compose.yaml
├── Dockerfile
├── Jenkinsfile
├── k8s
│   ├── deployment.yml
│   ├── pod.yml
│   ├── replica-sets.yml
│   └── service.yml
├── kustomize
│   ├── base
│   │   ├── app-1
│   │   │   ├── app-1.yml
│   │   │   └── kustomization.yml
│   │   └── ingress
│   │       ├── ingress.yml
│   │       └── kustomization.yml
│   ├── overlays
│   │   ├── dev
│   │   │   ├── dev-ingress-patch.json
│   │   │   └── kustomization.yml
│   │   └── prd
│   │       ├── kustomization.yml
│   │       └── prd-ingress-patch.json
│   └── README.md
├── nodetodo
│   ├── Chart.yaml
│   ├── charts
│   ├── README.md
│   ├── templates
│   │   ├── _helpers.tpl
│   │   ├── deployment.yaml
│   │   ├── hpa.yaml
│   │   ├── ingress.yaml
│   │   ├── NOTES.txt
│   │   ├── service.yaml
│   │   ├── serviceaccount.yaml
│   │   └── tests
│   │       └── test-connection.yaml
│   └── values.yaml
├── package-lock.json
├── package.json
├── README.md
├── sonar-project.properties
├── terraform
│   ├── main.tf
│   └── terraform.tf
├── test.js
└── views
    ├── edititem.ejs
    └── todo.ejs
```

---

### 📦 1. Create Namespace and Set Context

```bash
kubectl create ns hc
kubectl config set-context --current --namespace=hc
```

---

### 🚢 2. Deploy the App using Helm

```bash
helm install nodetodo ./nodetodo -f ./nodetodo/values.yaml -n hc
```

**Output:**
```
NAME: nodetodo
LAST DEPLOYED: Thu Jun 26 15:12:25 2025
NAMESPACE: hc
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace hc -o jsonpath="{.spec.ports[0].nodePort}" services nodetodo)
  export NODE_IP=$(kubectl get nodes --namespace hc -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```

---

### 🔍 3. Verify All Resources

```bash
kubectl get all
```

**Example Output:**
```
NAME                            READY   STATUS    RESTARTS   AGE
pod/nodetodo-77c447bbb5-l8r4h   1/1     Running   0          54s

NAME               TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/nodetodo   NodePort   10.106.7.153   <none>        80:30003/TCP   54s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nodetodo   1/1     1            1           54s

NAME                                           REFERENCE             TARGETS              MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/nodetodo   Deployment/nodetodo   cpu: <unknown>/80%   1         5         0          54s
```

---

### 🌐 4. Access the App

Use `port-forward` to access locally:

```bash
kubectl port-forward service/nodetodo 8080:80
```

Then visit [http://localhost:8080](http://localhost:8080)

---

### 📈 5. Generate Load to Test HPA

```bash
kubectl run -i --tty load-generator --image=busybox -n hc -- /bin/sh
```

Then inside the container, run this infinite loop to generate traffic:

```sh
while true; do wget -q -O - http://nodetodo.hc.svc.cluster.local; done
```

> ⚠️ **Press `Ctrl+C` to stop the load generation once scaling is observed.**

---

### 📊 6. Observe Autoscaling

Open a new terminal and run:

```bash
watch kubectl get all
```

**Example Output after load:**
```
NAME                            READY   STATUS    RESTARTS   AGE
pod/load-generator              1/1     Running   0          83s
pod/nodetodo-77c447bbb5-gjgn4   1/1     Running   0          9s
pod/nodetodo-77c447bbb5-l8r4h   1/1     Running   0          11m
pod/nodetodo-77c447bbb5-nr9r7   1/1     Running   0          9s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nodetodo   3/3     3            3           11m

NAME                                           REFERENCE             TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/nodetodo   Deployment/nodetodo   cpu: 200%/80%   1         5         3          11m
```

#

### 🌐 7. Ingress Setup

**Install NGINX Ingress Controller on your Kubernetes cluster:**

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx
```

**Update your local `/etc/hosts` file**:

```bash
127.0.0.1 nodetodo.local
```

**Visit the app at:**

```
http://nodetodo.local
```
---

### ✅ Success

The Node.js To-Do application was successfully deployed using a Helm-managed Kubernetes workflow. Key features include:

- **Helm** for templated, repeatable deployments
- **GitHub Actions** for continuous integration and delivery (CI/CD)
- **Horizontal Pod Autoscaler (HPA)** with **Metrics Server** for automatic scaling based on CPU utilization
- **NGINX Ingress Controller** for clean and manageable routing using custom hostnames
- Local development access via **port-forwarding** and/or **Ingress-based domain mapping**

This deployment demonstrates a robust DevOps pipeline with production-grade infrastructure practices.


---

◉ Helm chart for the repository [node-todo-cicd](https://github.com/LondheShubham153/node-todo-cicd) by Shubham Londhe.



