# Understanding Kubernetes Pods and kubectl

## 1. Introduction

In this class, I started learning how to deploy my first application in Kubernetes.

Before deploying an application, we need to understand a few important Kubernetes concepts.

In Docker, we build and run containers. But in Kubernetes, we don't directly deploy containers. Instead, we deploy them inside a resource called a **Pod**.

To understand how Kubernetes works, we will first learn about:

- What is a Pod?
- Why does Kubernetes use Pods instead of directly running containers?
- How YAML files are used in Kubernetes.
- Why can a Pod contain multiple containers?
- What is kubectl?
- What is the difference between a Pod and a Deployment?
- How do we troubleshoot Kubernetes applications?

Let's understand each concept step by step.

---

## 2. What is a Pod in Kubernetes?

A Pod is the smallest deployable unit in Kubernetes.

It is a wrapper around one or more containers.

In simple words, Kubernetes doesn't directly manage individual containers as its basic deployment unit. It manages Pods, and containers run inside those Pods.

### Docker vs Kubernetes

Let's understand this with a simple comparison.

**In Docker:**

We use the `docker run` command to start a container.

While running a container, we can provide different configurations such as:

- Container image.
- Container name.
- Port mapping.
- Volume mounts.
- Network settings.

**In Kubernetes:**

Instead of passing all these configurations through a command, we define them inside a YAML file.

Kubernetes uses that configuration to create and manage a Pod.

### Simple Example

Imagine we have an NGINX application.

In Docker, we run the NGINX container directly.

In Kubernetes, we create a Pod and run the NGINX container inside it.

The basic idea is:

```text
          Docker
             |
             v
        Container
             |
             v
       Application


        Kubernetes
             |
             v
            Pod
             |
             v
         Container
             |
             v
        Application
```

**Important:** A Pod is not the same as a container. It provides an additional layer that Kubernetes uses to manage containers.

---

## 3. What is a Pod YAML File?

In Docker, we generally use commands to provide container configurations.

For example, we use different options to specify the image, name, ports and volumes.

Kubernetes follows a different approach.

It uses YAML files to define the desired configuration of resources.

A Pod YAML file describes how we want our Pod and its containers to run.

### Why do we use YAML?

Kubernetes uses YAML to provide a declarative way of managing applications.

**What does declarative mean?**

It means we describe the desired state of what we want, and Kubernetes works to make the actual state match it.

For example, instead of manually providing every instruction through a long command, we define our application configuration in a YAML file.

This makes it easier to:

- Understand application configurations.
- Maintain deployment files.
- Share configurations with team members.
- Store Kubernetes files in GitHub.
- Track configuration changes.
- Reuse configurations across environments.

### Important YAML Fields

| Field | Purpose |
|---|---|
| `apiVersion` | Specifies the Kubernetes API version |
| `kind` | Specifies the resource type |
| `metadata` | Contains resource information such as name |
| `spec` | Defines the desired configuration |
| `containers` | Defines the containers inside the Pod |
| `image` | Specifies the container image |
| `ports` | Defines the container ports |

We will use these fields while creating our first Pod.

**My Understanding:**

YAML files make Kubernetes configurations easier to read, maintain and manage instead of writing everything through command-line arguments.

---

## 4. Can a Pod Contain Multiple Containers?

Yes!

A Pod can contain:

1. A single container.
2. Multiple containers.

Most of the time, we use one main application container inside a Pod.

However, there are situations where multiple containers need to work together.

### Why do we need multiple containers?

Imagine we have an application running inside Container A.

This application needs another container to support it by handling tasks such as logging, configuration or other supporting operations.

Instead of running both containers in separate Pods, we can place them inside the same Pod when they need to work closely together.

### Example

Imagine we have:

- Container A – Main Application.
- Container B – Supporting Container.

Both containers run inside the same Pod.

<escape>
</escape>

```text
          Kubernetes Pod
         ________________
        |                |
        |  Container A   |
        | Main Application|
        |________________|
        |                |
        |  Container B   |
        | Supporting Task|
        |________________|
        |________________|
```

### Advantages of Multiple Containers in a Pod

When multiple containers are inside the same Pod, they can share certain resources.

**1. Shared Networking**

Containers inside the same Pod share the same network namespace.

This means they can communicate with each other using `localhost`.

For example:

If Container A runs an application on port 3000, Container B can communicate with it using:

```text
localhost:3000
```

**2. Shared Storage**

Containers in the same Pod can share storage volumes.

This allows one container to write files that another container can access.

### What are Sidecar Containers?

A Sidecar Container is a supporting container that runs alongside the main application container inside the same Pod.

It helps the main application perform additional tasks.

Examples include:

- Collecting application logs.
- Handling monitoring tasks.
- Supporting network communication.

### What are Init Containers?

Init Containers are special containers that run before the main application containers start.

They are commonly used for tasks such as:

- Preparing the application environment.
- Checking required dependencies.
- Downloading configuration files.

We will explore these advanced concepts later.

**Important:** Multiple containers should be placed in the same Pod when they need to share resources and work closely together. Otherwise, separate Pods are generally more suitable.

---

## 5. Pod IP Address

Another important concept is networking.

In Kubernetes, each Pod normally receives its own IP address through the cluster networking system.

The application running inside the container can be accessed through the Pod's IP address, subject to the cluster's networking and access configuration.

### Example

Imagine we have three application Pods:

```text
       Kubernetes Cluster
               |
       ___________________
       |        |        |
       v        v        v
     Pod 1    Pod 2    Pod 3
       |        |        |
     IP 1     IP 2     IP 3
```

Each Pod has its own IP address.

If a Pod contains multiple containers, those containers share the Pod's network namespace and IP address.

### Important Difference

- Pod IP belongs to the Pod.
- Containers inside the same Pod share the Pod's network.
- Containers do not receive separate Pod IP addresses.

Pod networking is generally provided through a CNI (Container Network Interface) plugin.

---

## 6. What is kubectl?

In Docker, we use the Docker CLI to interact with Docker.

Similarly, Kubernetes provides a command-line tool called **kubectl**.

kubectl is the command-line tool used to communicate with and manage Kubernetes clusters.

It allows us to perform different operations such as:

- Creating Pods.
- Viewing Pods.
- Managing Deployments.
- Checking Nodes.
- Viewing application logs.
- Troubleshooting resources.
- Deleting Kubernetes resources.

### Docker CLI vs kubectl

| Docker CLI | kubectl |
|---|---|
| Interacts with Docker | Interacts with Kubernetes |
| Manages containers | Manages Kubernetes resources |
| `docker ps` | `kubectl get pods` |
| `docker run` | Creates resources using Kubernetes configurations |
| `docker logs` | `kubectl logs` |

### Important kubectl Commands

These are some commands we will use while working with Kubernetes:

| Command | Purpose |
|---|---|
| `kubectl get nodes` | Displays cluster Nodes |
| `kubectl get pods` | Displays Pods |
| `kubectl get deployments` | Displays Deployments |
| `kubectl describe pod` | Displays detailed Pod information |
| `kubectl logs` | Displays application logs |
| `kubectl delete pod` | Deletes a Pod |

We will practise these commands during the practical implementation.

---

## 7. Why Do We Need Deployments?

Now comes an important question.

If we can create Pods and run applications inside them, why do we need Deployments?

Remember why we started learning Kubernetes.

We wanted features such as:

- Auto Healing.
- Auto Scaling.
- Managing application replicas.
- Enterprise-level application management.

Creating a standalone Pod does not provide the full application management capabilities we need in production.

This is where **Deployments** come into the picture.

### What is a Deployment?

A Deployment is a Kubernetes resource used to manage application Pods.

It manages ReplicaSets, which help maintain the required number of Pod replicas.

### How does it work?

Imagine we want three copies of our application running.

A Deployment helps maintain the desired number of replicas.

If one Pod fails, the ReplicaSet works to create a replacement Pod.

This helps Kubernetes maintain the desired application state.

### Pod vs Deployment

| Pod | Deployment |
|---|---|
| Smallest deployable unit | Manages application Pods |
| Runs one or more containers | Manages ReplicaSets and Pods |
| Can be created directly | Creates and manages Pods |
| Does not independently provide Deployment-based rollout management | Supports application updates and rollbacks |
| Suitable for understanding basic Kubernetes workloads | Commonly used for stateless applications in production |

**My Understanding:**

We learn Pods first because they are the foundation of Kubernetes application deployment.

But in real-world environments, we generally use Deployments to manage application workloads instead of creating standalone Pods.

We will explore Deployments in detail in the upcoming classes.

---

## 8. How Do We Troubleshoot Kubernetes Pods?

While working with applications, we may face different problems.

For example:

- A Pod is not starting.
- An application is crashing.
- A container is not running properly.
- The application is showing errors.

Kubernetes provides commands that help us investigate these problems.

### 1. kubectl describe

The `describe` command provides detailed information about a Kubernetes resource.

For example:

```bash
kubectl describe pod nginx
```

It helps us understand:

- Pod status.
- Container information.
- Node information.
- Events related to the Pod.
- Errors and scheduling problems.

### 2. kubectl logs

The `logs` command helps us view the logs generated by an application running inside a container.

Example:

```bash
kubectl logs nginx
```

It is useful when an application is showing errors or not behaving as expected.

### Difference Between describe and logs

| Command | Purpose |
|---|---|
| `kubectl describe pod` | Shows detailed information and events about a Pod |
| `kubectl logs` | Shows application logs from a container |

**Real-World Example:**

Imagine your application is not working properly.

First, you can use `kubectl describe pod` to check its status and events.

Then, you can use `kubectl logs` to investigate application errors.

These are two important commands for Kubernetes troubleshooting.

---

## 9. Local Kubernetes Cluster Options

For learning and development, we don't always need a full Kubernetes cluster on AWS.

We can use local Kubernetes solutions.

Our teacher mentioned the following options:

| Tool | Purpose |
|---|---|
| Minikube | Runs a local Kubernetes cluster |
| K3s | Lightweight Kubernetes distribution |
| Kind | Runs Kubernetes clusters using Docker containers |
| MicroK8s | Provides a lightweight Kubernetes distribution |

For our learning and upcoming practicals, we are using Minikube.

We can explore other tools as we gain more experience.

---

## 10. What I Learned

In this theory session, I understood the basic concepts required before deploying my first application in Kubernetes.

### Concepts Covered

- What is a Pod?
- Why Kubernetes uses Pods instead of directly deploying containers.
- Difference between Docker containers and Kubernetes Pods.
- What is a Pod YAML file?
- Why Kubernetes uses declarative configurations.
- How multiple containers work inside a Pod.
- Shared networking and storage.
- Sidecar Containers.
- Init Containers.
- Pod IP addresses.
- What is kubectl?
- Difference between Pods and Deployments.
- Why Deployments are important.
- Basic Kubernetes troubleshooting concepts.
- Local Kubernetes cluster options.

### Final Understanding

Docker allows us to build and run containers.

Kubernetes takes container management further by introducing Pods as its smallest deployable unit.

A Pod provides an environment for one or more containers to run together. YAML files define how these resources should be configured, and kubectl helps us communicate with the Kubernetes cluster.

We start with Pods to understand how applications run in Kubernetes. Later, we use Deployments to manage those Pods and get features such as scaling, self-healing and controlled application updates.

**My main takeaway is that understanding Pods is the foundation for learning how applications are deployed and managed in Kubernetes.**

---

**Next Step: Practical Implementation**

In the next part, I will install and configure the required tools, create a local Kubernetes cluster using Minikube and deploy my first application as a Pod.

**Learn • Build • Break • Fix • Grow**
