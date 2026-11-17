# Project: Kubernetes Services – Service Discovery, Load Balancing and Application Exposure

## 1. Goal of the Practical

The goal of this practical was to understand how Kubernetes Services work and how they help applications communicate inside and outside a Kubernetes cluster.

I used a Python Django sample application and deployed it inside my Minikube cluster.

Through this practical, I demonstrated three important capabilities of Kubernetes Services:

1. **Service Discovery:** Finding application Pods using labels and selectors.
2. **Load Balancing:** Distributing incoming requests between multiple Pods.
3. **Application Exposure:** Exposing applications using NodePort and understanding LoadBalancer.

I also used Kubeshark to monitor network traffic and understand how requests travel between Kubernetes components.

## 2. Tools and Technologies Used

| Tool | Purpose |
|---|---|
| AWS EC2 | Linux environment |
| Ubuntu | Operating system |
| Minikube | Kubernetes cluster |
| kubectl | Managing Kubernetes resources |
| Docker | Building application images |
| Python Django | Sample application |
| Kubernetes Deployment | Managing application replicas |
| Kubernetes Service | Service Discovery and traffic routing |
| Kubeshark | Monitoring Kubernetes network traffic |
| Git and GitHub | Getting application source code |

## 3. Architecture

```text
                 Client / EC2 Terminal
                         |
                    HTTP Request
                         |
                         v
                NodePort Service
                    Port 30007
                         |
                         v
                 Kubernetes Service
                     Port 80
                         |
               ---------------------
               |                   |
               v                   v
          Python Pod 1        Python Pod 2
           Port 8000           Port 8000
               |                   |
               ---------------------

                 Kubeshark
          Network Traffic Monitoring
```

---

# Phase 1: Prepare the Environment

## Step 1: Connect to the EC2 Instance

I connected to my AWS EC2 instance using SSH through MobaXterm.

Check the current directory:

```bash
pwd
```

## Step 2: Check Minikube Status

```bash
minikube status
```

The cluster should be running.

If it is stopped:

```bash
minikube start
```

## Step 3: Verify Kubernetes Nodes

```bash
kubectl get nodes -o wide
```

The Minikube node should show `Ready`.

## Step 4: Check Existing Resources

```bash
kubectl get all
```

I checked the existing resources and removed only old application resources that were no longer required.

```bash
kubectl delete deployment <old-deployment-name>
kubectl delete service <old-service-name>
```

I did not delete the default Kubernetes Service or my Kubeshark resources.

---

# Phase 2: Download the Python Django Application

## Step 5: Clone the GitHub Repository

I used Abhishek Veeramalla's Docker Zero to Hero repository.

GitHub Repository:

https://github.com/iam-veeramalla/Docker-Zero-to-Hero

Clone the repository:

```bash
git clone https://github.com/iam-veeramalla/Docker-Zero-to-Hero.git
```

Move into the Python application directory:

```bash
cd Docker-Zero-to-Hero/examples/python-web-app
```

Check the files:

```bash
ls
```

The directory contains the Python application source code, Dockerfile and requirements file.

## Step 6: Understand the Dockerfile

```bash
cat Dockerfile
```

I checked the Dockerfile to understand how the application image is built and which port the application uses.

The Django application runs on port `8000` and its application endpoint is `/demo/`.

## Step 7: Build the Docker Image

```bash
docker build -t python-sample-app:v1 .
```

Check the image:

```bash
docker images
```

Load the image into Minikube:

```bash
minikube image load python-sample-app:v1
```

Verify:

```bash
minikube image ls
```

**Why did I do this?**

Kubernetes Pods need access to the application image. Loading the image into Minikube avoids depending on a remote image registry for this practical.

---

# Phase 3: Create Kubernetes Deployment

## Step 8: Create deployment.yaml

I created a Deployment to run two replicas of my Python Django application.

```bash
vim deployment.yaml
```

Add the following configuration:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: sample-python-app
  labels:
    app: sample-python-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: sample-python-app

  template:
    metadata:
      labels:
        app: sample-python-app

    spec:
      containers:
        - name: python-app
          image: python-sample-app:v1
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8000
```

Save the file:

```text
Esc
:wq
Enter
```

### Understanding the Configuration

| Configuration | Explanation |
|---|---|
| kind: Deployment | Creates a Kubernetes Deployment |
| replicas: 2 | Maintains two application Pods |
| selector | Identifies Pods using labels |
| template | Defines the Pod configuration |
| image | Specifies the Docker image |
| containerPort: 8000 | Application listening port |

**Important:** The label `app: sample-python-app` is important because I will use exactly the same label in the Service selector.

## Step 9: Apply the Deployment

```bash
kubectl apply -f deployment.yaml
```

Expected output:

```text
deployment.apps/sample-python-app created
```

Check the Deployment:

```bash
kubectl get deployments
```

Check the Pods:

```bash
kubectl get pods
```

Get additional information, including Pod IP addresses:

```bash
kubectl get pods -o wide
```

Both Pods should show:

- READY: 1/1
- STATUS: Running

## Step 10: Understand Dynamic Pod IP Addresses

I checked the IP addresses of the two Pods:

```bash
kubectl get pods -o wide
```

Then I deleted one Pod to understand how Kubernetes handles Pod failures.

First, copy one of the Pod names:

```bash
kubectl delete pod <pod-name>
```

Check again:

```bash
kubectl get pods -o wide
```

I observed that Kubernetes automatically created a replacement Pod.

The new Pod could receive a different IP address.

**What I learned:**

- Pod IP addresses can change.
- Deployments maintain the desired number of replicas.
- Applications should not depend on fixed Pod IP addresses.
- Kubernetes Services provide a stable way to access application Pods.

---

# Phase 4: Create a NodePort Service

## Step 11: Create service.yaml

I created a Service to expose my Python application.

```bash
vim service.yaml
```

Add the following configuration:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: python-django-app-service

spec:
  type: NodePort

  selector:
    app: sample-python-app

  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
      nodePort: 30007
```

### Understanding the Service Configuration

| Field | Explanation |
|---|---|
| kind: Service | Creates a Kubernetes Service |
| type: NodePort | Exposes the Service through a node port |
| selector | Finds Pods using their labels |
| port: 80 | Service port |
| targetPort: 8000 | Application port inside the Pod |
| nodePort: 30007 | Port exposed on the Kubernetes node |

### Traffic Flow

```text
Client Request
      |
      v
NodePort: 30007
      |
      v
Service Port: 80
      |
      v
Pod TargetPort: 8000
```

## Step 12: Create the Service

```bash
kubectl apply -f service.yaml
```

Expected output:

```text
service/python-django-app-service created
```

Check:

```bash
kubectl get services
```

Or:

```bash
kubectl get svc
```

Expected output should look similar to:

```text
NAME                        TYPE       CLUSTER-IP      PORT(S)
python-django-app-service   NodePort   10.106.x.x      80:30007/TCP
```

The ClusterIP and other values may differ.

## Step 13: Verify Service Endpoints

```bash
kubectl get endpoints python-django-app-service
```

Expected result: Two backend Pod IP addresses with port 8000.

For the newer Kubernetes API, also check:

```bash
kubectl get endpointslices \
  -l kubernetes.io/service-name=python-django-app-service
```

This confirms that the Service has discovered the application Pods.

---

# Phase 5: Access the Application Using NodePort

## Step 14: Get the Minikube IP

```bash
minikube ip
```

In my earlier execution, the Minikube IP was:

```text
192.168.49.2
```

The IP may be different when the cluster is recreated.

## Step 15: Test NodePort Access

The standard NodePort URL is:

```text
http://<MINIKUBE-IP>:30007/demo/
```

For example:

```bash
curl -L http://192.168.49.2:30007/demo/
```

If successful, the Django application should return its response.

### Issue Faced: NodePort Connection Failed

My first attempt returned:

```text
curl: (7) Failed to connect to 192.168.49.2 port 30007
```

**Reason:**

Minikube was running with an isolated Docker network, so its internal node IP was not necessarily reachable directly from the EC2 host.

**Solution 1: Use Minikube Service Helper**

```bash
minikube service python-django-app-service --url
```

This can provide a local tunnel URL on Linux. Keep the command running and use another terminal to test the returned URL:

```bash
curl -L <RETURNED-URL>/demo/
```

**Solution 2: Use Port Forwarding**

In one terminal:

```bash
kubectl port-forward service/python-django-app-service 8080:80
```

Keep it running.

Open another terminal:

```bash
curl -L http://127.0.0.1:8080/demo/
```

This provides a reliable way to test the Service through port forwarding.

**Important:** Port forwarding tests the Service through a local tunnel. It does not prove direct NodePort connectivity.

---

# Phase 6: Understand LoadBalancer Service

## Step 16: Change NodePort to LoadBalancer

I wanted to understand how Kubernetes exposes applications to external users using a LoadBalancer Service.

Open:

```bash
vim service.yaml
```

Change:

```yaml
type: NodePort
```

To:

```yaml
type: LoadBalancer
```

Apply:

```bash
kubectl apply -f service.yaml
```

Check:

```bash
kubectl get svc
```

On my Minikube cluster, the External IP can remain:

```text
<pending>
```

**Why?**

Minikube does not automatically create an AWS, Azure or GCP load balancer.

In a supported cloud environment, a LoadBalancer Service requests an external load balancer through the configured cloud integration.

## Step 17: Restore NodePort

For the remaining practical, I restored NodePort.

Open:

```bash
vim service.yaml
```

Restore:

```yaml
type: NodePort
```

Apply:

```bash
kubectl apply -f service.yaml
```

Verify:

```bash
kubectl get svc
```

Make sure the Service shows:

```text
NodePort   80:30007/TCP
```

---

# Phase 7: Demonstrate Service Discovery

## Step 18: Verify Existing Labels

Check Pod labels:

```bash
kubectl get pods --show-labels
```

My application Pods should have:

```text
app=sample-python-app
```

Check the Service selector:

```bash
kubectl describe service python-django-app-service
```

Look for:

```text
Selector: app=sample-python-app
```

Both values should match.

Check endpoints:

```bash
kubectl get endpoints python-django-app-service
```

Both application Pod IP addresses should appear.

## Step 19: Intentionally Break Service Discovery

Now I intentionally changed the Service selector to understand what happens when labels and selectors do not match.

Open:

```bash
vim service.yaml
```

Change:

```yaml
selector:
  app: sample-python-app
```

To:

```yaml
selector:
  app: wrong-python-app
```

Save the file.

Apply the changes:

```bash
kubectl apply -f service.yaml
```

## Step 20: Verify the Changed Selector

```bash
kubectl describe service python-django-app-service
```

Expected:

```text
Selector: app=wrong-python-app
```

Now check the endpoints:

```bash
kubectl get endpoints python-django-app-service
```

Expected:

```text
NAME                        ENDPOINTS
python-django-app-service   <none>
```

This happens because none of my application Pods have the label `app=wrong-python-app`.

## Step 21: Test the Application

If using the local port-forward method, access it from another terminal:

```bash
curl -v http://127.0.0.1:8080/demo/
```

Expected: The request should fail because the Service has no backend endpoints.

**What I learned:**

When the Service selector does not match the Pod labels, Kubernetes cannot discover the application Pods through that Service.

### Issue Faced: Application Still Responded After Changing the Selector

Initially, I changed the selector to `wrong-python-app` but still received the application response.

I learned not to judge Service Discovery only by the curl response.

I verified the following:

```bash
kubectl get pods --show-labels
```

```bash
kubectl describe service python-django-app-service
```

```bash
kubectl get endpoints python-django-app-service
```

```bash
kubectl get svc -A
```

I also made sure I was testing the correct Service, namespace and URL.

If the endpoints are empty, the Service has no selected backend Pods. A successful response would indicate that I was reaching another working route or Service, or that I was not testing the updated configuration.

## Step 22: Fix Service Discovery

Now I restored the original selector.

Open:

```bash
vim service.yaml
```

Change it back:

```yaml
selector:
  app: sample-python-app
```

Apply:

```bash
kubectl apply -f service.yaml
```

Check:

```bash
kubectl get endpoints python-django-app-service
```

The two Pod IP addresses should appear again.

Test:

```bash
curl -L http://127.0.0.1:8080/demo/
```

Or use the working Minikube Service URL.

The application should now respond successfully.

---

# Phase 8: Demonstrate Load Balancing

## Step 23: Verify the Two Application Pods

```bash
kubectl get pods -o wide
```

I checked that both Pods were running.

The two Pods act as backend application instances.

## Step 24: Generate Multiple HTTP Requests

I sent multiple requests to the Python application to generate traffic.

Using the working local port-forward URL:

```bash
for i in {1..10}; do
    curl -s -L http://127.0.0.1:8080/demo/ > /dev/null
    echo "Request $i sent"
done
```

This generates ten HTTP requests.

The Kubernetes Service forwards connections to its available backend Pods.

With multiple Pods, the Service provides traffic distribution across the backends.

**Note:** Kubernetes does not guarantee an exact 50-50 distribution of requests. Connection reuse and networking behaviour can affect the distribution.

---

# Phase 9: Install and Configure Kubeshark

## Step 25: Install Kubeshark

Kubeshark is a network traffic monitoring tool for Kubernetes.

I used it to understand how requests travel between Kubernetes resources.

Official documentation:

https://docs.kubeshark.co/

Follow the official installation instructions for Linux.

Verify the CLI:

```bash
kubeshark --help
```

## Step 26: Start Traffic Capture

To capture traffic across all namespaces, I used:

```bash
kubeshark tap -A
```

Keep the capture process running.

The `-A` option captures traffic across all namespaces.

## Step 27: Open the Kubeshark Dashboard

I accessed the dashboard using:

```text
http://localhost:8899
```

If accessing from my Windows laptop, I used SSH port forwarding:

```bash
ssh -L 8899:localhost:8899 -i your-key.pem ubuntu@<EC2-PUBLIC-IP>
```

Keep the SSH session open and visit the dashboard in the local browser.

## Step 28: Generate Python Application Traffic

Keep Kubeshark running.

Open another terminal and execute:

```bash
for i in {1..10}; do
    curl -s -L http://127.0.0.1:8080/demo/ > /dev/null
    echo "Request $i sent"
    sleep 1
done
```

## Step 29: Inspect Traffic in Kubeshark

Open the Kubeshark dashboard.

Navigate to:

1. Real-time.
2. API Stream.
3. Find requests to `/demo/`.
4. Open an individual request.
5. Inspect Request and Response information.

I observed successful HTTP requests returning status code `200 OK`.

## Step 30: Understand the Captured Traffic

| Information | Meaning |
|---|---|
| GET | HTTP request method |
| `/demo/` | Requested application endpoint |
| HTTP 200 | Successful HTTP response |
| Source | Origin of the captured request |
| Destination | Target of the captured request |
| Port 8000 | Python application port |

## Step 31: Understand Traffic Distribution

I used Kubeshark to inspect how requests reached my application Pods.

The expected traffic flow was:

```text
Client
  |
  v
Kubernetes Service
  |
  |------------------|
  |                  |
  v                  v
Pod 1              Pod 2
Python Django      Python Django
  |                  |
  --------------------
```

**What I learned:**

Kubeshark helped me visualize Kubernetes network communication instead of relying only on terminal commands.

It made it easier to understand how Services route requests to backend Pods.

---

# Phase 10: Final Verification

## Step 32: Check Deployment

```bash
kubectl get deployments
```

## Step 33: Check Pods

```bash
kubectl get pods -o wide
```

## Step 34: Check Services

```bash
kubectl get svc
```

## Step 35: Check Endpoints

```bash
kubectl get endpoints python-django-app-service
```

## Step 36: Describe the Service

```bash
kubectl describe service python-django-app-service
```

## Step 37: Describe the Deployment

```bash
kubectl describe deployment sample-python-app
```

These commands helped me verify the Deployment, Pods, Service configuration and backend endpoints.

---

# Phase 11: Cleanup

After completing the practical, I removed the application resources.

First, stop any running port-forward process using `Ctrl + C`.

Delete the Service:

```bash
kubectl delete -f service.yaml
```

Delete the Deployment:

```bash
kubectl delete -f deployment.yaml
```

Verify:

```bash
kubectl get all
```

The application Deployment and Service should no longer appear.

I did not remove the Minikube cluster or Kubeshark because I may need them for other practicals.

---

# 12. Errors Faced and Solutions

| Error / Issue | Reason | Solution |
|---|---|---|
| NodePort connection failed | Minikube IP was not reachable directly from EC2 host | Used `minikube service --url` and port forwarding |
| Selector changed but application still responded | Actual Service configuration or request route needed verification | Checked labels, selectors, endpoints and URL |
| Kubeshark showed too much system traffic | Capturing traffic across multiple namespaces | Identified Python application's `/demo/` requests |
| Kubeshark traffic was not visible immediately | Capture or proxy connection needed attention | Verified capture process and Kubeshark connection |

---

# 13. What I Learned and Achieved

Through this practical, I learned:

1. How to create a Kubernetes Deployment with multiple replicas.
2. How Kubernetes automatically recreates deleted Pods.
3. Why Pod IP addresses should not be hardcoded.
4. How Kubernetes Services provide stable access to application Pods.
5. How to configure a NodePort Service.
6. The difference between ClusterIP, NodePort and LoadBalancer.
7. How labels and selectors are used for Service Discovery.
8. What happens when Service selectors do not match Pod labels.
9. How Kubernetes Services distribute traffic between backend Pods.
10. How to use Kubeshark to monitor Kubernetes network traffic.

**Final Outcome:**

I successfully deployed a Python Django application on Minikube, created a Kubernetes Service, tested Service Discovery, explored LoadBalancer behaviour and used Kubeshark to inspect application traffic.

---

# 14. Important Commands Reference

## Environment and Cluster

```bash
minikube status
minikube start
minikube ip
kubectl get nodes -o wide
kubectl get all
```

## Docker Image

```bash
docker build -t python-sample-app:v1 .
docker images
minikube image load python-sample-app:v1
```

## Deployment

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get pods
kubectl get pods -o wide
kubectl get pods --show-labels
kubectl delete pod <pod-name>
```

## Services and Endpoints

```bash
kubectl apply -f service.yaml
kubectl get svc
kubectl describe svc python-django-app-service
kubectl get endpoints python-django-app-service
kubectl get endpointslices
```

## Application Access

```bash
minikube service python-django-app-service --url
kubectl port-forward service/python-django-app-service 8080:80
curl -L http://127.0.0.1:8080/demo/
```

## Kubeshark

```bash
kubeshark --help
kubeshark tap -A
```

## Cleanup

```bash
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
kubectl get all
```

---

# 15. References and Learning Resources

I referred to the following resources while completing this practical.

| Resource | Purpose |
|---|---|
| [Abhishek Veeramalla – Docker Zero to Hero](https://github.com/iam-veeramalla/Docker-Zero-to-Hero) | Python sample application |
| [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) | Deployment configuration |
| [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/) | ClusterIP, NodePort and LoadBalancer |
| [Debugging Kubernetes Services](https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/) | Service troubleshooting |
| [Kubernetes EndpointSlices](https://kubernetes.io/docs/concepts/services-networking/endpoint-slices/) | Understanding Service endpoints |
| [Minikube Documentation](https://minikube.sigs.k8s.io/docs/) | Cluster management and networking |
| [Kubeshark Documentation](https://docs.kubeshark.co/) | Network traffic monitoring |

---

# Final Takeaway

This practical helped me understand that Kubernetes Services are not just used to expose applications. They also provide a stable way to discover Pods and distribute traffic between multiple application replicas.

Using Kubeshark made the practical more interesting because I could actually observe network traffic instead of only reading about how Kubernetes works.
