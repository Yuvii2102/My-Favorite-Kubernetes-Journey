## Official References and Their Practical Uses

I used the following official documentation while setting up Kubernetes on my AWS EC2 instance. Each reference helped me understand a particular installation or configuration step.

### 1. kubectl Installation

**Official Documentation:**  
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

**Practical Use:**

I referred to this documentation to understand how to install `kubectl` on a Linux machine.

It provides installation instructions for different Linux distributions and processor architectures.

In my practical, I used the Linux AMD64 binary installation method.

Commands used:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

I verified the installation using:

```bash
kubectl version --client
```

### 2. Minikube Installation

**Official Documentation:**  
https://minikube.sigs.k8s.io/docs/start/

**Practical Use:**

I referred to this documentation to install Minikube on my Ubuntu EC2 instance.

It explains how to download and install Minikube based on the operating system and processor architecture.

I used the Linux AMD64 installation method.

Commands used:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

I verified the installation using:

```bash
minikube version
```

### 3. Docker Driver Configuration

**Official Documentation:**  
https://minikube.sigs.k8s.io/docs/drivers/docker/

**Practical Use:**

I referred to this documentation to understand how Minikube uses Docker as its driver.

Since I was using an AWS EC2 instance, I chose Docker to run the Minikube Kubernetes node.

Command used:

```bash
minikube start --driver=docker
```

This created my Kubernetes cluster using Docker.

### 4. Docker User Permissions

**Official Documentation:**  
https://docs.docker.com/engine/install/linux-postinstall/

**Practical Use:**

I referred to this documentation when I faced a Docker permission error while starting Minikube.

Initially, Minikube could not access the Docker socket because my current Linux user did not have the required Docker group permissions.

I fixed this by adding my user to the Docker group.

Commands used:

```bash
sudo systemctl enable docker

sudo usermod -aG docker $USER

newgrp docker
```

I verified Docker access using:

```bash
docker ps
```

After this, Minikube was able to use Docker successfully.

**Important:** Membership in the Docker group provides root-level privileges. Only add trusted users to this group.

### 5. kubectl Cheat Sheet

**Official Documentation:**  
https://kubernetes.io/docs/reference/kubectl/cheatsheet/

**Practical Use:**

I referred to this cheat sheet to understand and explore different kubectl commands.

It provides examples for managing Kubernetes resources without having to memorise every command.

During this practical, I used commands such as:

```bash
kubectl get nodes
kubectl get pods
kubectl get pods -o wide
kubectl describe pod nginx
kubectl logs nginx
kubectl create -f pod.yaml
kubectl delete pod nginx
```

This documentation is useful whenever I need to find the correct kubectl command while working with Kubernetes.

---

**Note:** These official references helped me install the required tools, configure Docker, create my Kubernetes cluster and manage my first NGINX Pod.
