# Deploying My First Application in Kubernetes Using Minikube

## 1. Project Overview

In this practical, I set up a Kubernetes cluster on an AWS EC2 instance using Minikube. After creating the cluster, I deployed my first application using a Kubernetes Pod.

I used NGINX as the sample application and learned how to create a Pod using a YAML file, check its status, access the application from inside the cluster, view logs and delete the Pod.

The main goal was to understand how Kubernetes works practically and how we can manage applications using `kubectl`.

### Technologies and Tools Used

| Technology | Purpose |
|---|---|
| AWS EC2 | Linux environment for running our practical |
| Ubuntu | Operating system on EC2 |
| Docker | Container runtime used by Minikube |
| Minikube | Creates a local Kubernetes cluster |
| Kubernetes | Container orchestration platform |
| kubectl | Command-line tool to communicate with Kubernetes |
| Vim | Text editor used to create YAML files |
| NGINX | Sample web server application |

---

## 2. Prerequisites

Before starting, I made sure I had:

- An AWS EC2 instance running Ubuntu.
- SSH access to the EC2 instance.
- An internet connection.
- Docker installed and running.
- Sufficient CPU, RAM and disk space.

I used an Ubuntu EC2 instance with an x86_64 architecture.

Minikube recommends at least 2 CPUs, 2 GB of free memory and 20 GB of free disk space. These are minimum requirements, so additional resources may be needed depending on the workload.

---

## 3. Installing kubectl

### What is kubectl?

`kubectl` is the command-line tool used to communicate with a Kubernetes cluster.

We use it to create, view, update and delete Kubernetes resources such as Pods, Deployments and Services.

Some examples:

```bash
kubectl get nodes
kubectl get pods
kubectl create -f pod.yaml
kubectl delete pod nginx
```

### Step 1: Update package information

I first updated the Ubuntu package information.

```bash
sudo apt update
```

This refreshes the package information available to the system.

### Step 2: Download kubectl

I downloaded the latest stable kubectl binary for Linux x86_64.

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

### Step 3: Install kubectl

```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

This installs kubectl in `/usr/local/bin` so that I can execute it from the terminal.

### Step 4: Verify the installation

```bash
kubectl version --client
```

This displays the installed kubectl client version.

**Official documentation:** https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

---

## 4. Installing Minikube

### What is Minikube?

Minikube is a tool that allows us to create and run a Kubernetes cluster on a single machine.

Instead of creating a full production Kubernetes cluster, we can use Minikube to practise Kubernetes concepts in a learning environment.

### Step 1: Download Minikube

I downloaded the Minikube binary for Linux x86_64.

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```

### Step 2: Install Minikube

```bash
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

This installs Minikube in `/usr/local/bin`.

### Step 3: Verify the installation

```bash
minikube version
```

My installed version:

```text
minikube version: v1.39.0
```

**Official documentation:** https://minikube.sigs.k8s.io/docs/start/

---

## 5. Configuring Docker for Minikube

### Why do we need Docker?

Minikube needs a driver to create and run its Kubernetes node.

In this practical, I used Docker as the driver.

Docker was already installed on my EC2 instance, so I checked its service and configured user permissions.

### Step 1: Check Docker version

```bash
docker --version
```

### Step 2: Check Docker service status

```bash
sudo systemctl status docker
```

I verified that the Docker service was active and running.

### Step 3: Enable Docker at startup

```bash
sudo systemctl enable docker
```

This enables Docker to start automatically when the EC2 instance boots.

### Step 4: Add the current user to the Docker group

```bash
sudo usermod -aG docker $USER
```

This adds the current Linux user to the Docker group so that Docker commands can be executed without using `sudo`.

### Step 5: Apply the group changes

Initially, Minikube showed a Docker permission error because my current SSH session had not picked up the new group membership.

I fixed it using:

```bash
newgrp docker
```

This starts a new shell with the updated group membership.

### Step 6: Verify Docker access

```bash
docker ps
```

The command executed successfully without a permission error.

**Important:** Membership in the Docker group grants root-level privileges on the Linux machine. Only add trusted users to this group.

**Official documentation:** https://docs.docker.com/engine/install/linux-postinstall/

---

## 6. Starting the Kubernetes Cluster

Now that Docker and Minikube were installed, I created my Kubernetes cluster.

### Step 1: Start Minikube using Docker

```bash
minikube start --driver=docker
```

### What does this command do?

- `minikube start` starts a Kubernetes cluster.
- `--driver=docker` tells Minikube to use Docker to run its Kubernetes node.

During startup, Minikube performed the following operations:

1. Checked the Docker driver.
2. Pulled the required Minikube base image.
3. Downloaded Kubernetes components.
4. Created the Minikube Docker container.
5. Prepared Kubernetes.
6. Configured the Container Network Interface (CNI).
7. Verified Kubernetes components.
8. Enabled the storage provisioner and default storage class.

After successful completion, I received:

```text
Done! kubectl is now configured to use "minikube" cluster
```

This means kubectl was configured to communicate with the Minikube cluster.

### Step 2: Check Minikube status

```bash
minikube status
```

My output showed:

```text
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

This confirmed that the Minikube cluster was running.

---

## 7. Verifying the Kubernetes Cluster

After starting Minikube, I checked whether the Kubernetes node was ready.

### Step 1: Get the nodes

```bash
kubectl get nodes
```

My output:

```text
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   ...   v1.37.0
```

### Understanding the output

| Field | Meaning |
|---|---|
| NAME | Name of the Kubernetes node |
| STATUS | Shows whether the node is ready |
| ROLES | Role assigned to the node |
| AGE | How long the node has been running |
| VERSION | Kubernetes version |

The `Ready` status confirmed that my Kubernetes node was healthy and ready to accept workloads.

### Step 2: Get detailed node information

```bash
kubectl get nodes -o wide
```

My output included:

```text
NAME       STATUS   ROLES           VERSION   INTERNAL-IP
minikube   Ready    control-plane   v1.37.0   192.168.49.2
```

It also showed:

- OS Image: Debian GNU/Linux 12
- Container Runtime: containerd 2.3.4
- Kernel Version: 7.0.0-1016-aws

The EC2 host was running Ubuntu 26.04, while the Minikube node reported Debian GNU/Linux 12. This is because Minikube was running its Kubernetes node inside a Docker container.

**Important:** This is a single-node Kubernetes learning cluster. The Minikube node acts as both the control plane and the worker node.

---

## 8. Creating Our First Kubernetes Pod

### What is a Pod?

A Pod is the smallest deployable unit in Kubernetes.

It provides an environment in which one or more containers can run.

In this practical, I created a Pod that runs an NGINX container.

### Step 1: Create a project directory

```bash
mkdir kubernetes-practice
cd kubernetes-practice
```

I created a separate directory to organise my Kubernetes YAML files.

### Step 2: Create the Pod YAML file using Vim

```bash
vim pod.yaml
```

Inside Vim:

1. Press `i` to enter Insert Mode.
2. Paste the following YAML configuration.

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx

spec:
  containers:
    - name: nginx
      image: nginx:1.14.2
      ports:
        - containerPort: 80
```

3. Press `Esc` to return to Normal Mode.
4. Type `:wq` and press `Enter` to save and exit.

### Step 3: Verify the YAML file

```bash
cat pod.yaml
```

This displays the contents of the YAML file.

### Understanding the YAML configuration

| Field | Purpose |
|---|---|
| `apiVersion: v1` | Specifies the Kubernetes API version |
| `kind: Pod` | Tells Kubernetes to create a Pod |
| `metadata` | Contains information about the Pod |
| `name: nginx` | Name of the Pod |
| `spec` | Defines the desired Pod configuration |
| `containers` | Lists the containers inside the Pod |
| `name: nginx` | Name of the container |
| `image: nginx:1.14.2` | NGINX image used by the container |
| `containerPort: 80` | Port on which the container application listens |

### Understanding the Docker equivalent

Since I had already worked with Docker, I compared the Pod configuration with its approximate Docker equivalent.

```bash
docker run -d \
  --name nginx \
  -p 80:80 \
  nginx:1.14.2
```

In Docker, we use `docker run` to start a container.

In Kubernetes, we define the desired configuration in a YAML file and use kubectl to create the Pod.

**Important:** `containerPort: 80` documents the container's port. It does not automatically expose the application outside the Kubernetes cluster. We need a Kubernetes Service for that.

---

## 9. Deploying the NGINX Pod

Now that the YAML file was ready, I used kubectl to create the Pod.

### Step 1: Create the Pod

```bash
kubectl create -f pod.yaml
```

My output:

```text
pod/nginx created
```

### Understanding the command

- `kubectl` communicates with Kubernetes.
- `create` creates a Kubernetes resource.
- `-f` specifies the configuration file.
- `pod.yaml` is the YAML file containing the Pod definition.

Kubernetes accepted the configuration and created the NGINX Pod.

### Step 2: Check the Pod status

```bash
kubectl get pods
```

My output:

```text
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          ...
```

### Understanding the output

| Field | Meaning |
|---|---|
| NAME | Name of the Pod |
| READY | Number of ready containers out of the total containers |
| STATUS | Current Pod status |
| RESTARTS | Number of container restarts |
| AGE | How long the Pod has been running |

My Pod showed `1/1` under READY, which means the one container inside the Pod was ready.

The `Running` status confirmed that the NGINX container was running.

---

## 10. Finding the Pod IP Address

Every Pod receives its own IP address inside the Kubernetes cluster.

To view the Pod IP, I used:

```bash
kubectl get pods -o wide
```

My NGINX Pod received this IP address:

```text
10.244.0.3
```

This is an internal Pod IP used for communication inside the cluster.

The `-o wide` option displays additional information such as the Pod IP and the node where the Pod is running.

---

## 11. Accessing NGINX from Inside Minikube

Now I wanted to verify whether the NGINX application was actually responding.

I accessed the Minikube node and sent an HTTP request to the Pod IP.

### Step 1: Connect to the Minikube node

```bash
minikube ssh
```

This opens a shell inside the Minikube node.

### Step 2: Access the NGINX application

```bash
curl http://10.244.0.3
```

My terminal returned the NGINX welcome page HTML.

Important parts of the response:

```html
<title>Welcome to nginx!</title>

<h1>Welcome to nginx!</h1>

<p>Thank you for using nginx.</p>
```

This confirmed that the NGINX web server was running and responding to HTTP requests inside the Kubernetes cluster.

### Step 3: Exit the Minikube node

```bash
exit
```

This returned me to my original AWS EC2 terminal.

**What I learned:** We can use `minikube ssh` to enter the Minikube node and test communication with applications running inside the cluster.

This test does not expose the application to the public internet.

---

## 12. Troubleshooting Using kubectl

Kubernetes provides useful commands to inspect resources and troubleshoot application issues.

I practised two important commands.

### Step 1: Describe the Pod

```bash
kubectl describe pod nginx
```

This displays detailed information about the Pod.

Some important details I checked:

- Pod name and namespace
- Node where the Pod is running
- Pod IP address
- Container image
- Container state
- Ready status
- Restart count
- Events

### Understanding the Events

My output showed these events:

| Event | Meaning |
|---|---|
| Scheduled | Kubernetes assigned the Pod to the Minikube node |
| Pulling | Kubernetes started downloading the NGINX image |
| Pulled | The image was successfully downloaded |
| Created | The container was created |
| Started | The container was started |

These events helped me understand how Kubernetes schedules and starts a Pod.

### Step 2: Check Pod logs

```bash
kubectl logs nginx
```

My output:

```text
10.244.0.1 - - [16/Nov/2026:12:26:41 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.88.1" "-"
```

### Understanding the log

| Field | Meaning |
|---|---|
| `10.244.0.1` | Client IP address |
| `GET /` | Request to the root webpage |
| `HTTP/1.1` | HTTP protocol version |
| `200` | Successful HTTP response |
| `612` | Response size in bytes |
| `curl/7.88.1` | Client that made the request |

The HTTP status code `200` confirmed that NGINX successfully processed the request.

This also helped me understand how application logs can be used to troubleshoot issues.

---

## 13. Deleting the Pod

After completing the practical, I learned how to delete a Kubernetes Pod.

### Step 1: Delete the NGINX Pod

```bash
kubectl delete pod nginx
```

Expected output:

```text
pod "nginx" deleted
```

### Step 2: Verify the deletion

```bash
kubectl get pods
```

Expected output:

```text
No resources found in default namespace.
```

This confirmed that the standalone NGINX Pod had been deleted.

### Important concept: Why doesn't Kubernetes recreate the Pod?

In this practical, I created a standalone Pod directly using:

```bash
kubectl create -f pod.yaml
```

A standalone Pod does not have a Deployment managing it.

Therefore, when I deleted the Pod, Kubernetes did not automatically create another one.

In real-world applications, we generally use Kubernetes Deployments to manage Pods.

Deployments provide capabilities such as:

- Maintaining the desired number of replicas.
- Replacing failed or deleted Pods.
- Scaling applications.
- Performing rolling updates.

We will explore Deployments in the next practical.

---

## 14. Important Commands Cheat Sheet

| Command | Purpose |
|---|---|
| `kubectl version --client` | Check kubectl version |
| `minikube version` | Check Minikube version |
| `minikube start --driver=docker` | Start Kubernetes using Docker |
| `minikube status` | Check Minikube status |
| `minikube ssh` | Connect to the Minikube node |
| `kubectl get nodes` | List Kubernetes nodes |
| `kubectl get nodes -o wide` | Display detailed node information |
| `kubectl create -f pod.yaml` | Create a Pod using YAML |
| `kubectl get pods` | List Pods |
| `kubectl get pods -o wide` | Display detailed Pod information |
| `kubectl describe pod nginx` | Get detailed Pod information |
| `kubectl logs nginx` | View container logs |
| `kubectl delete pod nginx` | Delete the Pod |
| `exit` | Exit the current shell |

For more kubectl commands, I can refer to the official Kubernetes cheat sheet:

https://kubernetes.io/docs/reference/kubectl/cheatsheet/

---

## 15. What I Learned and Achieved

Through this practical, I learned how to set up and use a Kubernetes learning environment on AWS EC2.

Here are the main things I learned:

1. Installed kubectl and Minikube on Ubuntu.
2. Configured Docker as the Minikube driver.
3. Created and started a single-node Kubernetes cluster.
4. Verified Kubernetes nodes using kubectl.
5. Understood how Pods work in Kubernetes.
6. Created a Pod configuration using YAML and Vim.
7. Deployed an NGINX application using kubectl.
8. Checked Pod status and internal IP addresses.
9. Accessed the NGINX application from inside the Minikube node.
10. Used describe and logs commands to inspect and troubleshoot Pods.
11. Deleted a standalone Pod and understood why it was not automatically recreated.

## 16. Final Understanding

This practical helped me understand how Kubernetes works beyond just theory.

I started with an AWS EC2 instance, installed the required tools, created a Kubernetes cluster using Minikube and deployed my first NGINX application.

I also learned how to inspect Pods, check application logs and understand the basic Pod lifecycle.

One important thing I understood is that creating a standalone Pod is useful for learning, but in real-world applications, we generally use Deployments to manage Pods and provide self-healing and scaling capabilities.

This was my first hands-on Kubernetes application deployment, and it gave me a better understanding of how to work with Kubernetes using YAML files and kubectl.
