# Installing Kubernetes Locally Using Minikube

## 1. Introduction

In this class, I learned how to install and run a Kubernetes cluster on my local machine using **Minikube**.

When learning Kubernetes, creating a full production-level cluster on cloud platforms can be expensive and requires more resources.

For learning and development purposes, we need a lightweight Kubernetes environment where we can practise different concepts without spending money on a full cluster.

This is where Minikube comes into the picture.

### What is Minikube?

Minikube is a tool that allows us to run a Kubernetes cluster locally on our laptop or virtual machine.

It is mainly used for learning, development and testing Kubernetes applications.

**Why do we use Minikube?**

- To create a local Kubernetes cluster.
- To practise Kubernetes concepts.
- To deploy and manage applications.
- To test Kubernetes configurations.
- To learn without creating an expensive cloud-based cluster.

### Other Local Kubernetes Options

Our teacher also mentioned a few alternatives:

- **Minikube:** A popular tool for running Kubernetes locally.
- **K3s:** A lightweight Kubernetes distribution.

In this class, we focused on Minikube because it is easy to get started with.

---

## 2. Prerequisites for Installing Minikube

Before installing Minikube, we need to make sure our machine has the required resources.

According to the class, the minimum requirements are:

| Requirement | Minimum |
|---|---|
| CPU | 2 CPUs |
| RAM | 2 GB free |
| Storage | 20 GB free |
| Internet | Required |

We also need a suitable driver or hypervisor depending on the installation setup.

### What is a Hypervisor?

A Hypervisor is software that allows us to create and run virtual machines on a physical machine.

**Examples:**
- VirtualBox
- VMware

**Real-World Example:**

Imagine we have a laptop running Windows.

We install VirtualBox and create an Ubuntu virtual machine inside it.

VirtualBox helps us create and run that virtual machine.

Minikube can use a supported driver to run its Kubernetes environment.

---

## 3. Two Important Tools We Need

To work with our local Kubernetes cluster, we need two important tools.

### 3.1 Minikube

Minikube is used to create and manage a local Kubernetes cluster.

It provides a simple way to start, stop and configure our local cluster.

### 3.2 kubectl

kubectl is the Kubernetes command-line tool.

It allows us to communicate with and manage our Kubernetes cluster.

Using kubectl, we can:

- Create Kubernetes resources.
- Check running Pods.
- View cluster Nodes.
- Deploy applications.
- Manage Kubernetes workloads.

**Easy Way to Remember:**

- Minikube helps us run a local Kubernetes cluster.
- kubectl helps us interact with that cluster.

---

## 4. Step 1 – Install Minikube

First, we need to visit the official Minikube documentation.

**Official Website:** https://minikube.sigs.k8s.io/docs/start/

Follow these steps:

1. Open the Minikube official documentation.
2. Select your operating system.
3. Follow the installation instructions.
4. Download the Minikube binary or use the installation script.
5. Make sure Minikube is available in your system PATH.

Once Minikube is installed, we can start our local Kubernetes cluster.

### Start Minikube

```bash
minikube start
```

This command starts a local Kubernetes cluster using the configured driver.

**What happens when we execute this command?**

Minikube starts the Kubernetes environment on our machine.

We can also pass additional options such as the driver and resource configurations depending on our requirements.

---

## 5. Step 2 – Install kubectl

After installing Minikube, we need kubectl to interact with our Kubernetes cluster.

Visit the official Kubernetes documentation:

**Official Website:** https://kubernetes.io/docs/tasks/tools/

Follow these steps:

1. Open the Kubernetes documentation.
2. Select your operating system.
3. Download the appropriate kubectl binary.
4. Follow the installation instructions.
5. Make sure kubectl is available from your terminal.

Once installed, kubectl can communicate with the Kubernetes cluster configured in our environment.

### Check Kubernetes Nodes

```bash
kubectl get nodes
```

This command displays the Nodes available in our Kubernetes cluster.

---

## 6. Managing the Minikube Cluster

Minikube provides different commands to manage our local Kubernetes cluster.

Here are some important commands:

| Command | Purpose |
|---|---|
| `minikube start` | Starts the cluster |
| `minikube stop` | Stops the cluster |
| `minikube pause` | Pauses the cluster |
| `minikube unpause` | Resumes a paused cluster |
| `minikube status` | Checks the cluster status |
| `minikube` | Displays available commands and options |

### Real-World Example

Imagine I have finished practising Kubernetes for the day.

I don't need to keep my local cluster running all the time.

I can stop the cluster using:

```bash
minikube stop
```

When I want to continue learning, I can start it again:

```bash
minikube start
```

This helps me manage my local environment whenever required.

---

## 7. Creating Multiple Minikube Clusters

Another interesting feature of Minikube is that it supports multiple clusters.

We can create different clusters for different purposes.

**Example:**

Imagine I am working on an application and need separate environments.

| Cluster | Purpose |
|---|---|
| Development Cluster | To develop and test new changes |
| Testing Cluster | To test application behaviour |

We can also configure different resources such as memory depending on our requirements.

This helps us practise working with different Kubernetes environments.

---

## 8. Minikube Add-ons

Minikube supports several add-ons that help extend the functionality of our local Kubernetes cluster.

### What are Add-ons?

Add-ons are additional components that we can enable in Minikube to use extra Kubernetes features.

Our teacher introduced a few examples:

**1. Ingress Controller**

An Ingress Controller manages external HTTP and HTTPS traffic and routes requests to the appropriate Kubernetes applications.

**2. Operator Lifecycle Manager (OLM)**

OLM helps manage the installation and lifecycle of Kubernetes Operators.

Minikube provides different add-ons that we can explore based on our learning requirements.

---

## 9. Complete Installation Flow

The overall process of setting up our local Kubernetes environment looks like this:

```text
        Local Machine / VM
                |
                v
       Check System Requirements
                |
                v
       Install Suitable Driver
                |
                v
         Install Minikube
                |
                v
          minikube start
                |
                v
     Local Kubernetes Cluster
                |
                v
          Install kubectl
                |
                v
        kubectl get nodes
                |
                v
       Interact with Cluster
                |
                v
       Deploy Applications
```

---

## 10. Important Commands Summary

| Command | Description |
|---|---|
| `minikube start` | Starts the local Kubernetes cluster |
| `minikube stop` | Stops the cluster |
| `minikube pause` | Pauses the cluster |
| `minikube unpause` | Resumes the cluster |
| `minikube status` | Displays the cluster status |
| `minikube addons list` | Lists available add-ons |
| `minikube addons enable ingress` | Enables the Ingress add-on |
| `kubectl get nodes` | Displays Kubernetes Nodes |

---

## 11. Important Interview Questions

**1. What is Minikube?**

Minikube is a tool used to run a local Kubernetes cluster for learning, development and testing.

**2. Why do we use Minikube?**

We use Minikube to practise Kubernetes without creating an expensive production-level cluster.

**3. What is kubectl?**

kubectl is a command-line tool used to communicate with and manage Kubernetes clusters.

**4. What is a Hypervisor?**

A Hypervisor is software that allows us to create and manage virtual machines on a physical machine.

**5. How do you start Minikube?**

```bash
minikube start
```

**6. How do you stop Minikube?**

```bash
minikube stop
```

**7. How do you check the Nodes in a Kubernetes cluster?**

```bash
kubectl get nodes
```

**8. What are Minikube Add-ons?**

Add-ons are additional components that extend the functionality of a Minikube cluster.

**9. Can we create multiple clusters using Minikube?**

Yes. Minikube supports multiple clusters, which can be used for different development and testing environments.

---

## 12. What I Learned

In this class, I learned how to set up a local Kubernetes environment using Minikube.

Here are the main concepts I covered:

- Why we need Minikube.
- What Minikube is and how it works.
- Difference between Minikube and kubectl.
- System requirements for installing Minikube.
- What a Hypervisor is.
- Installing Minikube.
- Installing kubectl.
- Starting and stopping a local cluster.
- Managing multiple clusters.
- Understanding Minikube Add-ons.

### Final Understanding

Minikube gives us a simple way to run Kubernetes locally without creating a full production-level cluster.

With Minikube, we can practise Kubernetes commands, create Pods, deploy applications and explore different Kubernetes features.

**My main takeaway is that before working with Kubernetes in production environments, I can use Minikube to learn and experiment in my own local environment.**

---

**Next Step:** Start creating and deploying our first Kubernetes Pod using Minikube.

**Learn • Build • Break • Fix • Grow**
