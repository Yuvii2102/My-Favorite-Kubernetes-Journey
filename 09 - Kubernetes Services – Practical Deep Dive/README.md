# Kubernetes Services – Practical Deep Dive

## 1. Introduction

In this lesson, I explored Kubernetes Services in more detail and understood how they work in a real Kubernetes environment.

In the previous lesson, I learned that Kubernetes Services provide three important capabilities:

1. Service Discovery
2. Load Balancing
3. Exposing applications outside the cluster

The main goal of this practical was to understand how Services communicate with Pods and how traffic reaches applications inside and outside a Kubernetes cluster.

## 2. Understanding the Application Architecture

For this practical, I used a Python Django sample application.

The application was deployed inside Kubernetes using a Deployment.

The Deployment creates a ReplicaSet, and the ReplicaSet manages the required number of Pods.

The application used port `8000` and the context path `/demo`.

### Application flow

```text
Kubernetes Deployment
        |
        v
    ReplicaSet
        |
        v
   Two NGINX? No
   Python Django Pods
        |
        v
  Kubernetes Service
        |
        v
  Application Access
```

The application used in this lesson was a Python Django application, not NGINX.

## 3. Preparing the Application

The first step was to prepare the application and build its Docker image.

The instructor used the Docker repository `Docker-0-to-0` containing sample Python and Golang applications.

The Python application was selected for this demonstration.

### Step 1: Build the Docker image

The Docker image was created using the Dockerfile.

Example command:

```bash
docker build -t python-sample-application-demo:v1 .
```

Here:

- `docker build` builds a Docker image.
- `-t` assigns a name and tag to the image.
- `python-sample-application-demo` is the image name.
- `v1` is the image version.
- `.` specifies the current directory as the build context.

## 4. Creating the Kubernetes Deployment

After building the Docker image, the next step was to deploy the application inside Kubernetes.

A Deployment YAML file was created.

The instructor used two replicas to demonstrate how Kubernetes Services distribute traffic between multiple Pods.

### Deployment configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-python-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sample-python-application
  template:
    metadata:
      labels:
        app: sample-python-application
    spec:
      containers:
        - name: python-app
          image: python-sample-application-demo:v1
          ports:
            - containerPort: 8000
```

### Important points

- `replicas: 2` creates two application Pods.
- `selector` identifies Pods using labels.
- `template` defines how the Pods should be created.
- `containerPort: 8000` represents the port used by the application.

**Important:** Labels and selectors play a major role in Kubernetes Services. The Service must use the same label as the Pods it needs to discover.

### Step 2: Deploy the application

```bash
kubectl apply -f deployment.yaml
```

Check the Deployment:

```bash
kubectl get deployments
```

Check the Pods:

```bash
kubectl get pods
```

Check Pod IP addresses:

```bash
kubectl get pods -o wide
```

## 5. Understanding Pod IP Changes

One important problem with accessing applications directly through Pod IP addresses is that Pod IPs are not permanent.

For example, initially, two Pods might have these IP addresses:

```text
Pod 1: 172.17.0.5
Pod 2: 172.17.0.6
```

If one Pod is deleted, the ReplicaSet creates a replacement Pod.

The new Pod may receive a different IP address:

```text
Pod 1: 172.17.0.5
Pod 2: 172.17.0.7
```

This creates a problem if users or other applications are directly accessing the old Pod IP.

The old IP might no longer point to the application.

### Why do we need Services?

Kubernetes Services solve this problem by identifying Pods through labels and selectors instead of depending on individual Pod IP addresses.

Even if a Pod is replaced and receives a new IP address, the Service can discover it as long as it has the matching label and is ready to receive traffic.

## 6. Understanding Service Discovery

Service Discovery is one of the main responsibilities of Kubernetes Services.

A Service uses labels and selectors to identify its backend Pods.

### Example

Suppose our Deployment creates Pods with this label:

```yaml
labels:
  app: sample-python-application
```

The Service uses the same label in its selector:

```yaml
selector:
  app: sample-python-application
```

Kubernetes identifies all matching Pods and uses them as Service endpoints.

### What happens when a Pod is replaced?

1. A Pod is deleted or fails.
2. The ReplicaSet creates a replacement Pod.
3. The new Pod receives the matching label.
4. Kubernetes updates the Service's endpoints.
5. The Service can send traffic to the new Pod when it is ready.

This is how Service Discovery helps avoid depending on changing Pod IP addresses.

## 7. Understanding NodePort Service

Now we come to the first major practical demonstration of Services.

A Pod is normally accessible through its cluster network. If we want users inside our organization's network to access the application through a Kubernetes node, we can use NodePort.

### What is NodePort?

NodePort exposes an application through a port on the Kubernetes nodes.

Users who can reach the node IP address and the assigned NodePort can access the application.

The default NodePort range is:

```text
30000 – 32767
```

### Step 1: Create the Service YAML file

Create a file:

```bash
vim service.yaml
```

Example configuration:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: python-django-app-service
spec:
  type: NodePort
  selector:
    app: sample-python-application
  ports:
    - port: 80
      targetPort: 8000
      nodePort: 30007
```

### Understanding the configuration

| Field | Purpose |
|---|---|
| `type: NodePort` | Exposes the application through a node port |
| `selector` | Finds the matching application Pods |
| `port: 80` | Port exposed by the Service |
| `targetPort: 8000` | Port where the application runs inside the container |
| `nodePort: 30007` | Port exposed on the Kubernetes nodes |

### Step 2: Create the Service

```bash
kubectl apply -f service.yaml
```

Check the Service:

```bash
kubectl get svc
```

Check its details:

```bash
kubectl describe service python-django-app-service
```

### Step 3: Access the application

Get the Minikube IP:

```bash
minikube ip
```

Access the application using:

```text
http://<MINIKUBE-IP>:30007/demo
```

The request follows this path:

```text
User
 |
 v
Node IP : 30007
 |
 v
Kubernetes Service
 |
 v
Matching Application Pod
 |
 v
Python Django Application
```

The Service forwards traffic to the application Pods on port `8000`.

### Important point

NodePort does not automatically make an application publicly accessible to everyone.

Users must have network access to the Kubernetes node. Firewall rules, cloud security groups and network configuration can also affect access.

## 8. Understanding LoadBalancer Service

Now suppose we want external customers outside our organization to access the application.

NodePort alone may not be enough for this requirement.

For this purpose, Kubernetes provides the LoadBalancer Service type.

### What is LoadBalancer?

A LoadBalancer Service requests an external load balancer from a supported cloud provider or load balancer implementation.

On cloud platforms such as AWS, Azure and GCP, the corresponding cloud integration can provision the required load balancer.

### Step 1: Change the Service type

The instructor demonstrated changing the existing Service from NodePort to LoadBalancer.

The command used was:

```bash
kubectl edit svc python-django-app-service
```

Find:

```yaml
type: NodePort
```

Change it to:

```yaml
type: LoadBalancer
```

Save and exit.

### Step 2: Check the Service

```bash
kubectl get svc
```

On a supported cloud environment, Kubernetes can request an external load balancer and display its external address.

### Why did LoadBalancer not receive an external IP in Minikube?

The instructor used Minikube for the demonstration.

Minikube does not automatically provision an AWS, Azure or GCP load balancer.

Therefore, the external IP may remain:

```text
<pending>
```

This is expected without a suitable load balancer implementation.

### How does it work on AWS?

When a LoadBalancer Service is created on a supported AWS Kubernetes environment, the cloud integration can communicate with AWS and provision the appropriate load balancer.

The external endpoint can then be used to access the application.

**In simple words:** LoadBalancer provides an external endpoint for applications through a supported load balancer implementation.

## 9. Understanding How Labels and Selectors Control Service Discovery

The instructor also demonstrated what happens when the Service selector does not match the Pod labels.

Initially, the Pod had this label:

```yaml
app: sample-python-application
```

The Service selector also used:

```yaml
selector:
  app: sample-python-application
```

Therefore, the Service could discover the Pods.

### What happens if the selector is changed?

Suppose the Service selector is changed to:

```yaml
selector:
  app: sample-python-app
```

Now the selector does not match the Pod label.

As a result, the Service cannot discover those Pods.

The Service has no matching backend endpoints, so requests through it cannot reach the application.

### How to fix it?

Restore the correct selector:

```yaml
selector:
  app: sample-python-application
```

Apply the updated configuration:

```bash
kubectl apply -f service.yaml
```

The Service can then discover the matching Pods again.

**Important learning:** Even a small difference in labels and selectors can prevent a Service from finding its backend Pods.

## 10. Understanding Load Balancing in Kubernetes Services

Another important responsibility of Kubernetes Services is load balancing.

Imagine we have two application Pods:

```text
Pod 1: 172.17.0.5
Pod 2: 172.17.0.7
```

If multiple requests come to the Service, Kubernetes networking distributes connections across the available backend Pods.

This helps prevent all traffic from depending on a single Pod.

### Practical demonstration

The instructor used a tool called KubeShark to observe traffic inside Kubernetes.

Multiple requests were sent to the application through the Service.

The traffic capture showed requests reaching both backend Pods.

This demonstrated how the Service distributes traffic across multiple Pods.

### Important point

Kubernetes Service traffic distribution should not be understood as a guarantee that every Pod receives exactly the same number of requests.

The actual distribution depends on the networking implementation and connection behaviour.

## 11. Understanding KubeShark

KubeShark is a tool used to observe and analyze network traffic inside Kubernetes.

It helps DevOps Engineers understand how traffic flows between different Kubernetes components and applications.

### Why is KubeShark useful?

- Observe network traffic inside Kubernetes.
- Understand communication between Services and Pods.
- Investigate traffic flow.
- Analyze TCP and HTTP traffic.
- Troubleshoot networking problems.
- Understand how applications communicate inside a cluster.

### Installation reference

Official documentation:

https://docs.kubeshark.co/

The instructor demonstrated the installation and usage of KubeShark during the lesson.

### Starting a traffic capture

The command demonstrated was:

```bash
kubeshark tap -a
```

The `-a` option is used to capture traffic across all namespaces.

Without this option, the capture can be limited to the default namespace.

The instructor accessed the KubeShark interface through:

```text
http://localhost:8899
```

### What did I learn from the traffic capture?

The traffic capture helped explain how a request travels from the client to the Kubernetes node, through the Service and finally to one of the backend Pods.

It also helped demonstrate how multiple requests can reach different Pods through the same Service.

## 12. Important Kubernetes Service Commands

| Command | Purpose |
|---|---|
| `kubectl get svc` | Lists Services |
| `kubectl get svc -o wide` | Displays additional Service information |
| `kubectl describe svc <name>` | Shows Service details |
| `kubectl get pods` | Lists Pods |
| `kubectl get pods -o wide` | Displays Pod IP addresses and node information |
| `kubectl get deployments` | Lists Deployments |
| `kubectl apply -f service.yaml` | Creates or updates a Service |
| `kubectl edit svc <name>` | Edits an existing Service |
| `kubectl delete svc <name>` | Deletes a Service |
| `kubectl get endpoints <name>` | Displays Service endpoints |
| `kubectl get endpointslices` | Lists EndpointSlices |

## 13. What I Learned

After completing this lesson, I understood the practical working of Kubernetes Services in more detail.

The main concepts I learned were:

1. How to deploy an application using a Kubernetes Deployment.
2. Why directly accessing Pod IP addresses is not reliable.
3. How labels and selectors help Services discover Pods.
4. How NodePort exposes applications through Kubernetes nodes.
5. How LoadBalancer exposes applications through an external load balancer.
6. How Kubernetes Services distribute traffic across multiple Pods.
7. How KubeShark helps us understand traffic flow inside Kubernetes.

## 14. Final Understanding

Kubernetes Services are important because they provide a stable way to access applications even when Pods are replaced or their IP addresses change.

They help applications discover each other, distribute traffic and expose applications to users.

I also understood that different Service types are used for different access requirements:

- ClusterIP for internal cluster communication.
- NodePort for access through Kubernetes node IPs and ports.
- LoadBalancer for external access through a supported load balancer.

My main takeaway from this lesson is that understanding how traffic flows inside Kubernetes is just as important as knowing how to create Deployments and Services.

---

## 15. Learning References

- [Kubernetes Services Documentation](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Kubernetes Deployments Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Kubernetes Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)
- [KubeShark Documentation](https://docs.kubeshark.co/)

**Learning source:** Abhishek Veeramalla – Complete DevOps Course, Day 37: Deep Dive into Kubernetes Services.
