You're asking how Kubernetes actually works **when you have your own Docker application/container**, rather than the earlier Nginx example where Kubernetes pulled an existing image automatically.

The workflow is:

**Create app → Docker build → Kubernetes deploy → Scale pods**

For your lab, the easiest way to demonstrate this is to build a small application, containerize it, and run it on Kubernetes using **Minikube** (local Kubernetes).

## **1. Understand how Docker connects to Kubernetes**

**Your application code**

`index.html`

**Docker**

`docker build -t myapp:v1 .`

Creates the image

**Kubernetes**

Deployment uses `myapp:v1`

**Pods**

Runs container(s) using your Docker image

Important: Kubernetes doesn't normally manage a container you manually started using `docker run`. Instead, Kubernetes creates and manages its own containers inside pods using the image you built.

# **2. Create a sample Docker application**

Create a folder named `k8s-demo`.

Inside it, create a file called `index.html`:

```
<h1>Hello from Docker + Kubernetes!</h1>
<p>My application is running inside a Kubernetes Pod.</p>
```

Create another file named `Dockerfile` (no extension):

```
FROM nginx
COPY index.html /usr/share/nginx/html/index.html
```

Your folder structure should be:

```
k8s-demo/
│
├── index.html
└── Dockerfile
```

# **3. Build your Docker image**

Open terminal inside that folder.

```
docker build -t myapp:v1 .
```

What this means:

- `docker build` = build an image
- `-t myapp:v1` = give image name `myapp`, version `v1`
- `.` = use current directory as build context

Check whether the image exists:

```
docker images
```

You should see `myapp` in the list.

# **4. Start Kubernetes using Minikube**

If you have Docker Desktop installed and running, start Minikube:

```
minikube start --driver=docker
```

Check Kubernetes:

```
kubectl get nodes
```

You should see a node in `Ready` status.

#### **Why Minikube?**

Kubernetes needs a cluster to run pods. Minikube creates a small Kubernetes cluster on your laptop, so you don't need AWS or a physical server for this demonstration.

# **5. Connect your Docker image to Kubernetes**

This is the important part that makes your own Docker image available to Kubernetes.

Load the image into Minikube:

```
minikube image load myapp:v1
```

Now Minikube's Kubernetes node can access the image you built.

## **6. Create a Kubernetes Deployment**

Create a file called `deployment.yaml`:

```
apiVersion: apps/v1
kind: Deployment

metadata:
  name: myapp

spec:
  replicas: 2

  selector:
    matchLabels:
      app: myapp

  template:
    metadata:
      labels:
        app: myapp

    spec:
      containers:
        - name: mycontainer
          image: myapp:v1
          imagePullPolicy: Never
          ports:
            - containerPort: 80
```

### **What does this YAML do?**

| YAML                     | Meaning                                                      |
| ------------------------ | ------------------------------------------------------------ |
| `kind: Deployment`       | Create a Kubernetes Deployment                               |
| `name: myapp`            | Name of Deployment                                           |
| `replicas: 2`            | Run 2 pods                                                   |
| `labels: app: myapp`     | Identifies the application pods                              |
| `image: myapp:v1`        | Use your Docker image                                        |
| `imagePullPolicy: Never` | Don't download from Docker Hub; use the locally loaded image |
| `containerPort: 80`      | Nginx listens on port 80 inside the container                |

## **7. Apply the deployment**

Run:

```
kubectl apply -f deployment.yaml
```

Kubernetes reads the YAML file and creates the Deployment.

Check:

```
kubectl get deployments
```

Check pods:

```
kubectl get pods
```

You should see **2 pods** running your application.

# **8. Expose your application so you can access it**

Pods are running, but you need a way to access them from your browser. Create a Service.

Run:

```
kubectl expose deployment myapp --type=NodePort --port=80
```

This creates a Kubernetes Service that routes traffic to your pods.

Check it:

```
kubectl get services
```

## **Access the website**

Run:

```
minikube service myapp
```

Minikube will open the application in your browser, or display an access URL.

You should see:

# **Hello from Docker + Kubernetes!**

My application is running inside a Kubernetes Pod.

# **9. Demonstrate scaling**

Initially, our Deployment has 2 replicas.

Scale it to 4 pods:

```
kubectl scale deployment myapp --replicas=4
```

Verify:

```
kubectl get pods
```

You should now see 4 pods.

Scale down to 1:

```
kubectl scale deployment myapp --replicas=1
```

Verify again:

```
kubectl get deployments
```

# **10. Complete commands to remember**

Full Docker + Kubernetes practical sequence

```
# Build Docker image
docker build -t myapp:v1 .

# Start Kubernetes cluster
minikube start --driver=docker

# Load local Docker image into Minikube
minikube image load myapp:v1

# Deploy application
kubectl apply -f deployment.yaml

# Check deployment
kubectl get deployments

# Check pods
kubectl get pods

# Expose application
kubectl expose deployment myapp --type=NodePort --port=80

# Check service
kubectl get services

# Open application
minikube service myapp

# Scale to 4 replicas
kubectl scale deployment myapp --replicas=4

# Check pods
kubectl get pods

# Scale down to 1
kubectl scale deployment myapp --replicas=1
```

## **The difference between the two practicals**

| Earlier Nginx practical              | Your own Docker + Kubernetes       |
| ------------------------------------ | ---------------------------------- |
| `--image=nginx`                      | `--image=myapp:v1`                 |
| Kubernetes pulls image from registry | You build the image                |
| No Dockerfile needed                 | You write a Dockerfile             |
| No image loading required            | Load local image into Minikube     |
| `kubectl create deployment`          | `kubectl apply -f deployment.yaml` |
| Scale pods                           | Scale pods                         |

### **The key concept**

**Docker builds the image. Kubernetes runs and manages containers using that image.**

Docker is for building and packaging. Kubernetes is for orchestration: running pods, maintaining replicas, scaling, restarting failed workloads, and managing deployments.

If you're doing this on your college lab PC, the requirements are Docker Desktop, Minikube, and `kubectl`. If they aren't installed, you may not be able to set up a local cluster from scratch during a timed external exam, so check what your lab already provides.
