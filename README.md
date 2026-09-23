# 🚀 Amazon EKS Ingress — Path-Based Routing Demo

A simple hands-on demonstration of **Amazon Elastic Kubernetes Service (EKS)** with **Kubernetes NGINX Ingress** and **path-based routing**.

This project demonstrates how a single Ingress endpoint can route incoming HTTP requests to different Kubernetes Services based on the URL path.

For example:

```text
/apple   → Apple Service
/banana  → Banana Service
```

The project is intentionally simple so that the complete request flow can be understood easily.

---

# 📌 Project Overview

This project contains two simple applications:

* 🍎 Apple application
* 🍌 Banana application

Both applications run inside the same Kubernetes namespace and are exposed internally using Kubernetes `ClusterIP` Services.

An NGINX Ingress receives external HTTP traffic and routes the request based on the URL path.

The architecture is:

```text
                         Internet
                            │
                            ▼
                    NGINX Ingress
                            │
              ┌─────────────┴─────────────┐
              │                           │
         /apple                       /banana
              │                           │
              ▼                           ▼
      apple-service                banana-service
              │                           │
              ▼                           ▼
        Apple Pod                    Banana Pod
              │                           │
              ▼                           ▼
       http-echo:5678              http-echo:5678
```

---

# ☁️ Technologies Used

* **Amazon EKS**
* **Kubernetes**
* **NGINX Ingress Controller**
* **Kubernetes Namespace**
* **Kubernetes Deployment**
* **Kubernetes Service**
* **Pods**
* **HashiCorp HTTP Echo**
* **kubectl**

---

# 🏗️ Architecture

The complete Kubernetes architecture is:

```text
                         AWS
                          │
                          ▼
                  Amazon EKS Cluster
                          │
                          ▼
                NGINX Ingress Controller
                          │
                 ┌────────┴────────┐
                 │                 │
             /apple             /banana
                 │                 │
                 ▼                 ▼
          apple-service      banana-service
             ClusterIP          ClusterIP
                 │                 │
                 ▼                 ▼
          Apple Deployment   Banana Deployment
                 │                 │
                 ▼                 ▼
             Apple Pod         Banana Pod
                 │                 │
                 ▼                 ▼
        hashicorp/http-echo  hashicorp/http-echo
             :5678                :5678
```

---

# 🧠 What is Kubernetes Ingress?

A Kubernetes **Ingress** is an API resource that defines rules for routing HTTP/HTTPS traffic from outside the cluster to Kubernetes Services.

Ingress can route traffic based on:

* Hostnames
* URL paths

It can also be used with features such as TLS termination and load balancing, depending on the Ingress controller being used.

In this project, we use **path-based routing**.

---

# 🔀 What is Path-Based Routing?

Path-based routing means that the URL path determines which backend Service receives the request.

For this project:

```text
http://<INGRESS>/apple
```

is routed to:

```text
apple-service:5678
```

while:

```text
http://<INGRESS>/banana
```

is routed to:

```text
banana-service:5678
```

The routing logic is:

```text
                    Request
                       │
              ┌────────┴────────┐
              │                 │
           /apple            /banana
              │                 │
              ▼                 ▼
       apple-service      banana-service
```

Kubernetes Ingress supports path rules such as `Prefix`, which matches a URL path prefix.

---

# 📂 Project Structure

```text
EKS-ingress-simple-Demo/
│
├── 01-namespace.yaml
├── 02-apple-app.yaml
├── 03-banana-app.yaml
└── 04-ingress.yaml
```

Each file has a specific purpose.

---

# 1️⃣ Namespace

File:

```text
01-namespace.yaml
```

This creates:

```text
demo-apps
```

namespace.

The namespace YAML is:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: demo-apps
```

The namespace keeps the resources for this demonstration logically separated from resources in other namespaces.

---

# 2️⃣ Apple Application

File:

```text
02-apple-app.yaml
```

This file creates two Kubernetes resources:

* Deployment
* Service

## Apple Deployment

The Deployment is:

```text
apple-deployment
```

It runs one replica of the application.

The container uses:

```text
hashicorp/http-echo:0.2.3
```

and returns:

```text
🍎 Hello from Apple Service!
```

The application listens on:

```text
5678
```

The actual repository manifest defines the Deployment in the `demo-apps` namespace with the label:

```text
app: apple
```

and exposes container port `5678`.

---

## Apple Service

The Deployment is exposed internally through:

```text
apple-service
```

The Service type is:

```text
ClusterIP
```

and it forwards:

```text
Service Port: 5678
        ↓
Target Port: 5678
```

The Service selects Pods using:

```yaml
selector:
  app: apple
```

Therefore:

```text
apple-service
      │
      ▼
Pods with app=apple
```

---

# 3️⃣ Banana Application

File:

```text
03-banana-app.yaml
```

This file also creates:

* Deployment
* Service

## Banana Deployment

The Deployment is:

```text
banana-deployment
```

It runs:

```text
1 replica
```

The container uses:

```text
hashicorp/http-echo:0.2.3
```

and returns:

```text
🍌 Hello from Banana Service!
```

The container listens on:

```text
5678
```

The repository manifest uses the label:

```text
app: banana
```

to identify the Banana Pod.

---

## Banana Service

The Service is:

```text
banana-service
```

with:

```text
Type: ClusterIP
Port: 5678
TargetPort: 5678
```

It selects:

```yaml
selector:
  app: banana
```

Therefore:

```text
banana-service
      │
      ▼
Pods with app=banana
```

---

# 4️⃣ NGINX Ingress

File:

```text
04-ingress.yaml
```

This is the most important file in the project.

The Ingress resource is:

```text
demo-ingress
```

inside:

```text
demo-apps
```

namespace.

It uses:

```yaml
ingressClassName: nginx
```

which tells Kubernetes that the NGINX Ingress Controller should handle this Ingress resource.

---

# 🔀 Ingress Routing Rules

The Ingress contains two paths.

## Apple Route

```yaml
- path: /apple
  pathType: Prefix
  backend:
    service:
      name: apple-service
      port:
        number: 5678
```

Therefore:

```text
/ apple
   ↓
apple-service
   ↓
Apple Pod
```

---

## Banana Route

```yaml
- path: /banana
  pathType: Prefix
  backend:
    service:
      name: banana-service
      port:
        number: 5678
```

Therefore:

```text
/ banana
   ↓
banana-service
   ↓
Banana Pod
```

These are the actual routing rules defined in the repository.

---

# 🌐 Request Flow

Suppose the NGINX Ingress receives:

```text
http://<INGRESS-IP>/apple
```

The request flow is:

```text
Browser
   │
   ▼
Ingress Endpoint
   │
   ▼
NGINX Ingress Controller
   │
   │ path = /apple
   ▼
apple-service
   │
   ▼
Apple Pod
   │
   ▼
http-echo
   │
   ▼
🍎 Hello from Apple Service!
```

For:

```text
http://<INGRESS-IP>/banana
```

the flow becomes:

```text
Browser
   │
   ▼
Ingress Endpoint
   │
   ▼
NGINX Ingress Controller
   │
   │ path = /banana
   ▼
banana-service
   │
   ▼
Banana Pod
   │
   ▼
http-echo
   │
   ▼
🍌 Hello from Banana Service!
```

---

# 🎯 Why Use Ingress?

Without Ingress, you could expose each application separately.

For example:

```text
Load Balancer 1 → Apple
Load Balancer 2 → Banana
```

With Ingress, a single entry point can route traffic to multiple Services:

```text
                    One Entry Point
                          │
                   NGINX Ingress
                     /        \
                    /          \
               /apple        /banana
                  │              │
                  ▼              ▼
             Apple Service  Banana Service
```

This makes Ingress useful for HTTP/HTTPS routing to multiple backend Services.

---

# 🧩 Why Are the Services ClusterIP?

Both:

```text
apple-service
banana-service
```

use:

```text
type: ClusterIP
```

This means the Services are intended to be accessed internally within the Kubernetes cluster.

The Ingress Controller provides the external HTTP entry point.

Therefore:

```text
Internet
   │
   ▼
Ingress
   │
   ├── ClusterIP → Apple
   │
   └── ClusterIP → Banana
```

This avoids exposing every application Service directly to the internet.

---

# 🔥 Prefix PathType

This project uses:

```yaml
pathType: Prefix
```

For example:

```text
/apple
```

can match requests under that path prefix.

The Kubernetes API defines `Prefix` matching based on URL path elements.

Conceptually:

```text
/apple
/apple/
/apple/test
```

can match the `/apple` prefix depending on the request and Ingress implementation.

Similarly:

```text
/banana
/banana/
/banana/test
```

can match the Banana route.

---

# ☸️ Prerequisites

Before deploying this project, you need:

* AWS account
* AWS CLI
* Kubernetes cluster
* Amazon EKS cluster
* `kubectl`
* NGINX Ingress Controller
* IAM permissions to access the required AWS resources

An Ingress resource requires an **Ingress Controller** to actually implement the routing. Creating the Ingress object alone is not sufficient.

---

# 🔐 Configure AWS CLI

Check AWS CLI:

```bash
aws --version
```

Configure credentials:

```bash
aws configure
```

Verify:

```bash
aws sts get-caller-identity
```

---

# ☁️ Create / Connect to EKS

If the EKS cluster already exists, configure `kubectl`:

```bash
aws eks update-kubeconfig \
  --region <AWS_REGION> \
  --name <EKS_CLUSTER_NAME>
```

Example:

```bash
aws eks update-kubeconfig \
  --region us-east-1 \
  --name my-eks-cluster
```

Verify:

```bash
kubectl get nodes
```

You should see nodes with:

```text
STATUS
------
Ready
```

---

# 🌐 Install NGINX Ingress Controller

This project expects an NGINX Ingress Controller because the manifest specifies:

```yaml
ingressClassName: nginx
```

Verify that an NGINX Ingress Controller exists:

```bash
kubectl get ingressclass
```

You should see something similar to:

```text
NAME
nginx
```

You can also check the controller:

```bash
kubectl get pods -A | grep ingress
```

The controller is responsible for watching Ingress resources and configuring the actual routing/load-balancing implementation.

---

# 🚀 Deployment

Clone the repository:

```bash
git clone https://github.com/Navaneethkrishna-coder/EKS-ingress-simple-Demo.git
```

Enter the directory:

```bash
cd EKS-ingress-simple-Demo
```

---

## Step 1 — Create Namespace

```bash
kubectl apply -f 01-namespace.yaml
```

Verify:

```bash
kubectl get namespace
```

---

## Step 2 — Deploy Apple Application

```bash
kubectl apply -f 02-apple-app.yaml
```

Verify:

```bash
kubectl get pods -n demo-apps
```

Check the Service:

```bash
kubectl get svc -n demo-apps
```

---

## Step 3 — Deploy Banana Application

```bash
kubectl apply -f 03-banana-app.yaml
```

Verify:

```bash
kubectl get pods -n demo-apps
```

You should have Pods for both applications.

---

## Step 4 — Create Ingress

```bash
kubectl apply -f 04-ingress.yaml
```

Check the Ingress:

```bash
kubectl get ingress -n demo-apps
```

More detailed information:

```bash
kubectl describe ingress demo-ingress -n demo-apps
```

---

# 🔎 Verify Everything

Run:

```bash
kubectl get all -n demo-apps
```

You should see resources similar to:

```text
NAME                                    READY
pod/apple-deployment-xxxx               1/1
pod/banana-deployment-xxxx              1/1

NAME                    TYPE
service/apple-service   ClusterIP
service/banana-service  ClusterIP

NAME                               READY
deployment.apps/apple-deployment  1/1
deployment.apps/banana-deployment 1/1
```

---

# 🌍 Get the Ingress Address

Run:

```bash
kubectl get ingress -n demo-apps
```

Depending on the controller and AWS setup, the address may appear as an IP address or hostname.

You can also run:

```bash
kubectl describe ingress demo-ingress -n demo-apps
```

Look for:

```text
Address:
```

Ingress controllers can take some time to provision or update their external address.

---

# 🧪 Test the Routes

Once the Ingress endpoint is available:

## Apple

```bash
curl http://<INGRESS-ADDRESS>/apple
```

Expected response:

```text
🍎 Hello from Apple Service!
```

---

## Banana

```bash
curl http://<INGRESS-ADDRESS>/banana
```

Expected response:

```text
🍌 Hello from Banana Service!
```

---

# 🌐 Browser Testing

Open:

```text
http://<INGRESS-ADDRESS>/apple
```

You should see:

```text
🍎 Hello from Apple Service!
```

Open:

```text
http://<INGRESS-ADDRESS>/banana
```

You should see:

```text
🍌 Hello from Banana Service!
```

---

# 🔍 Troubleshooting

## Check Pods

```bash
kubectl get pods -n demo-apps
```

If a Pod isn't running:

```bash
kubectl describe pod <pod-name> -n demo-apps
```

---

## Check Pod Logs

Apple:

```bash
kubectl logs deployment/apple-deployment -n demo-apps
```

Banana:

```bash
kubectl logs deployment/banana-deployment -n demo-apps
```

---

## Check Services

```bash
kubectl get svc -n demo-apps
```

Make sure:

```text
apple-service
banana-service
```

exist.

---

## Check Endpoints

```bash
kubectl get endpoints -n demo-apps
```

or:

```bash
kubectl get endpointslices -n demo-apps
```

If the Service has no endpoints, check that the Service selector matches the Pod labels.

For Apple:

```text
Service selector:
app: apple
```

Pod label:

```text
app: apple
```

For Banana:

```text
Service selector:
app: banana
```

Pod label:

```text
app: banana
```

These selectors are defined in the repository manifests.

---

# 🔎 Check Ingress

```bash
kubectl get ingress -n demo-apps
```

Then:

```bash
kubectl describe ingress demo-ingress -n demo-apps
```

You should see routes similar to:

```text
Host    Path      Backends
----    ----      --------
*       /apple    apple-service:5678
*       /banana   banana-service:5678
```

This corresponds to the path routing rules in `04-ingress.yaml`.

---

# 🧪 Test Services Without Ingress

You can also test the Services internally.

Start a temporary Pod:

```bash
kubectl run test-pod \
  --image=curlimages/curl \
  -it \
  --rm \
  -n demo-apps \
  -- sh
```

Inside the Pod:

```bash
curl http://apple-service:5678
```

Expected:

```text
🍎 Hello from Apple Service!
```

Test Banana:

```bash
curl http://banana-service:5678
```

Expected:

```text
🍌 Hello from Banana Service!
```

This helps isolate whether a problem is with the application/Service or with the Ingress layer.

---

# 🧹 Delete the Demo

Delete the Ingress:

```bash
kubectl delete -f 04-ingress.yaml
```

Delete Banana:

```bash
kubectl delete -f 03-banana-app.yaml
```

Delete Apple:

```bash
kubectl delete -f 02-apple-app.yaml
```

Delete the namespace:

```bash
kubectl delete -f 01-namespace.yaml
```

Or delete everything using:

```bash
kubectl delete -f .
```

---

# 📊 Resource Relationship

The most important relationship in this project is:

```text
Namespace
    │
    ├── Apple Deployment
    │       │
    │       └── Apple Pod
    │
    ├── Apple Service
    │
    ├── Banana Deployment
    │       │
    │       └── Banana Pod
    │
    ├── Banana Service
    │
    └── Ingress
            │
            ├── /apple  → apple-service
            │
            └── /banana → banana-service
```

---

# 🧠 The Complete Request Flow

Remember this flow for interviews:

```text
User
 │
 │ HTTP Request
 │
 ▼
NGINX Ingress
 │
 │ checks URL path
 │
 ├── /apple ───────► apple-service
 │                       │
 │                       ▼
 │                   Apple Pod
 │
 └── /banana ──────► banana-service
                         │
                         ▼
                     Banana Pod
```

The Ingress does **not** directly route to the Pods.

The normal flow is:

```text
Ingress
   ↓
Service
   ↓
Pod
   ↓
Container
```

---

# 🎯 Key Concepts Demonstrated

This project demonstrates:

* Amazon EKS
* Kubernetes namespaces
* Kubernetes Deployments
* Kubernetes Pods
* Kubernetes Services
* ClusterIP Services
* NGINX Ingress Controller
* Ingress resources
* Path-based routing
* `Prefix` path matching
* Service selectors
* Kubernetes networking
* External HTTP access
* Basic Kubernetes troubleshooting

---

# 🧑‍💻 Interview Questions

### What is Ingress?

Ingress is a Kubernetes API resource that defines HTTP/HTTPS routing rules from outside the cluster to Services inside the cluster.

### What is an Ingress Controller?

The Ingress Controller is the component that actually implements the Ingress rules. Without a controller, creating an Ingress resource alone does not provide the routing functionality.

### What is path-based routing?

Routing traffic to different backend Services based on the URL path.

Example:

```text
/app1 → service1
/app2 → service2
```

### Why use ClusterIP for the Services?

The Services only need to be reachable from inside the cluster because the Ingress provides the external HTTP entry point.

### What happens when `/apple` is requested?

```text
/ apple
   ↓
NGINX Ingress
   ↓
apple-service
   ↓
Apple Pod
   ↓
http-echo
```

### What happens when `/banana` is requested?

```text
/ banana
   ↓
NGINX Ingress
   ↓
banana-service
   ↓
Banana Pod
   ↓
http-echo
```

### Why are labels important?

Services use selectors to find the Pods to which they should send traffic.

Example:

```text
selector:
  app: apple
```

selects Pods with:

```text
app: apple
```

---

# 🔮 Possible Improvements

This basic demo can be extended with:

* [ ] Host-based routing
* [ ] HTTPS/TLS
* [ ] Route 53 DNS
* [ ] AWS Load Balancer Controller
* [ ] AWS Certificate Manager
* [ ] Multiple replicas
* [ ] Horizontal Pod Autoscaler
* [ ] Health checks
* [ ] Readiness probes
* [ ] Liveness probes
* [ ] Helm
* [ ] Terraform
* [ ] Jenkins CI/CD
* [ ] GitHub Actions
* [ ] Argo CD
* [ ] Prometheus
* [ ] Grafana

---

# ⚠️ Important Note

This project uses the Kubernetes **Ingress API** with the NGINX Ingress Controller. Kubernetes currently documents Ingress as stable, but the Ingress API is frozen and Kubernetes recommends considering the newer **Gateway API** for new development. This does not prevent this project from being a useful demonstration of traditional Kubernetes Ingress and path-based routing.

---

# 👨‍💻 Author

**Navaneeth Krishna**

Cloud & DevOps | AWS | Kubernetes | Docker | Terraform | Jenkins | AI/ML

---

# ⭐ Summary

This project demonstrates how a single NGINX Ingress can route requests to multiple Kubernetes Services using URL paths.

```text
                     Internet
                        │
                        ▼
                NGINX Ingress
                        │
              ┌─────────┴─────────┐
              │                   │
          /apple               /banana
              │                   │
              ▼                   ▼
       apple-service        banana-service
              │                   │
              ▼                   ▼
         Apple Pod           Banana Pod
              │                   │
              ▼                   ▼
       Apple Response       Banana Response
```

The key concept is:

> **Ingress receives the external HTTP request, examines the URL path, and forwards the request to the appropriate Kubernetes Service, which then sends the request to the matching Pod.**
