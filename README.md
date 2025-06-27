## 🚀 Helm Chart Deployment for Node TODO App

This guide walks through deploying the `Node TODO App` on Kubernetes using Helm, enabling autoscaling, and generating load to trigger Horizontal Pod Autoscaler (HPA).

---
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

### 📦 1. Create Namespace and Set Context

```bash
kubectl create ns hc
kubectl config set-context --current --namespace=hc
```

#

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

#

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

#

### 🌐 4. Access the App

Use `port-forward` to access locally:

```bash
kubectl port-forward service/nodetodo 8080:80
```

Then visit [http://localhost:8080](http://localhost:8080)

#

### 📈 5. Generate Load to Test HPA

```bash
kubectl run -i --tty load-generator --image=busybox -n hc -- /bin/sh
```

Then inside the container:

```sh
while true; do wget -q -O - http://nodetodo.hc.svc.cluster.local; done
```

#

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

### ✅ Success

You've successfully deployed your Node.js app using Helm with autoscaling enabled. The pods scaled automatically under CPU load using HPA and Metrics Server.
---