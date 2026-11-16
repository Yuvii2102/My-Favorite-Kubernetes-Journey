# Kubernetes Architecture

## 1. Introduction

In this concept I learned why Kubernetes is needed and how it solves the problems of managing containers at scale.

In this class, I explored the internal architecture of Kubernetes and understood how its different components work together.

Before learning the architecture, let's recall the four major advantages of Kubernetes :

1. Cluster Architecture
2. Auto Healing
3. Auto Scaling
4. Enterprise-Level Capabilities

To understand how Kubernetes provides these capabilities, I need to understand its architecture.

## 2. Kubernetes Architecture Overview

Kubernetes architecture is divided into two major parts:

1. **Control Plane (Master Node)**
2. **Data Plane (Worker Nodes)**

The Control Plane manages the Kubernetes cluster, while the Data Plane is responsible for running application workloads.

### Control Plane

The Control Plane is the brain of Kubernetes.

It manages the cluster, receives requests, makes scheduling decisions and maintains the overall cluster state.

### Data Plane

The Data Plane consists of Worker Nodes where application workloads actually run.

It contains Pods and the components required to run and manage containers.

**Simple Understanding:**

- Control Plane = Brain of Kubernetes
- Data Plane = Runs the applications

## 3. Understanding Docker vs Kubernetes Architecture

Our teacher explained Kubernetes architecture by comparing it with Docker.

This comparison helps us understand why Kubernetes requires different components.

### 3.1 How Does Docker Run a Container?

Imagine you have an EC2 instance or virtual machine with Docker installed.

You execute the following command:

```bash
docker run nginx
```

But how does the container actually run?

A container requires a Container Runtime to execute it.

In the Docker architecture discussed in class, Docker Shim was introduced as the runtime-related component.

Without a Container Runtime, Docker cannot execute the container.

### 3.2 How Does Kubernetes Run a Pod?

In Kubernetes, the smallest deployable unit is called a **Pod**.

A Pod acts as a wrapper around one or more containers and provides additional capabilities.

Unlike Docker, where we directly run a container, Kubernetes uses a Control Plane and Worker Nodes to manage Pods.

For example, imagine a Kubernetes cluster containing:

- One Master Node.
- One Worker Node.

When a user requests a Pod, the request goes through the Control Plane.

The Control Plane decides where the Pod should run, and the Worker Node runs it using its components.

## 4. Data Plane – Worker Node Components

Our teacher explained that every Kubernetes Worker Node has three important components:

1. Kubelet
2. Kube-proxy
3. Container Runtime

Let's understand each component.

### 4.1 Kubelet

**What is Kubelet?**

Kubelet is an agent that runs on every Worker Node.

Its main responsibility is to ensure that the Pods assigned to that Worker Node are running properly.

**Responsibilities:**

- Communicates with the Control Plane.
- Receives Pod specifications.
- Ensures that the required containers are running.
- Monitors the status of Pods.
- Reports the status of Pods and Nodes to the Control Plane.

**Real-World Example:**

Imagine you have a Kubernetes cluster with three Worker Nodes.

The Control Plane assigns an application Pod to Worker Node 1.

Kubelet on Worker Node 1 receives the instructions and works with the Container Runtime to ensure that the Pod runs successfully.

If something goes wrong, Kubelet reports the status to the Control Plane.

**Remember:** Kubelet is responsible for managing Pods on its Worker Node.

### 4.2 Kube-proxy

**What is Kube-proxy?**

Kube-proxy is a networking component that runs on Worker Nodes.

It helps implement Kubernetes Service networking and routes traffic to the appropriate backend Pods.

**Responsibilities:**

- Maintains network rules for Services.
- Helps distribute traffic to backend Pods.
- Supports Service networking.
- Uses mechanisms such as iptables or IPVS, depending on configuration.

**Real-World Example:**

Imagine you have three replicas of your application running in Kubernetes.

Users send requests through a Kubernetes Service.

Kube-proxy helps ensure that traffic reaches the appropriate backend Pods.

**What are iptables?**

iptables is a Linux utility used to configure firewall and packet-filtering rules.

Kube-proxy can use iptables to implement Service networking.

**Remember:** Kube-proxy helps with networking and traffic routing inside the Kubernetes cluster.

### 4.3 Container Runtime

**What is a Container Runtime?**

A Container Runtime is responsible for running containers.

Just like Docker requires a runtime to execute containers, Kubernetes also requires a Container Runtime.

Examples:

- containerd
- CRI-O

Our teacher also discussed Docker Shim while comparing Docker and Kubernetes.

Kubernetes supports different container runtimes through the Container Runtime Interface (CRI).

**Real-World Example:**

Imagine Kubernetes schedules a Pod on Worker Node 1.

Kubelet communicates with the Container Runtime to create and run the containers inside that Pod.

**Remember:** Container Runtime is responsible for executing containers.

### Worker Node Components – Summary

| Component | Responsibility |
|---|---|
| Kubelet | Manages Pods on the Worker Node |
| Kube-proxy | Implements Service networking |
| Container Runtime | Runs containers |

These are the three main Worker Node components discussed in class.

---

## 5. Control Plane – Master Node Components

Now let's understand the Control Plane.

The Worker Node components are responsible for running applications.

But who decides:

- Where should a Pod run?
- Which Worker Node should receive a Pod?
- How should the cluster be managed?
- Where is cluster information stored?
- How should different controllers be managed?

The Control Plane handles these responsibilities.

Our teacher introduced five important components:

1. API Server
2. Scheduler
3. etcd
4. Controller Manager
5. Cloud Controller Manager

### 5.1 API Server

**What is API Server?**

The API Server is the central component of the Kubernetes Control Plane.

It exposes the Kubernetes API and acts as the main entry point for requests to the cluster.

Whenever a user wants to create or manage Kubernetes resources, the request goes through the API Server.

**Responsibilities:**

- Receives requests from users.
- Validates API requests.
- Acts as the communication point for Control Plane components.
- Provides access to Kubernetes resources.
- Helps manage cluster operations.

**Real-World Example:**

Imagine you want to create a Pod.

You execute:

```bash
kubectl apply -f pod.yaml
```

Your request reaches the API Server.

The API Server processes the request and makes the resource information available to other Control Plane components.

**Remember:** API Server is the entry point for communication with the Kubernetes cluster.

### 5.2 Scheduler

**What is Scheduler?**

The Scheduler is responsible for deciding which Worker Node should run a newly created Pod.

Imagine your Kubernetes cluster has three Worker Nodes:

- Worker Node 1
- Worker Node 2
- Worker Node 3

You create a new Pod.

Now Kubernetes needs to decide where that Pod should run.

This decision is handled by the Scheduler.

**Responsibilities:**

- Identifies newly created Pods that need scheduling.
- Checks available resources on Worker Nodes.
- Selects a suitable Worker Node.
- Assigns the Pod to the selected Node.

**Real-World Example:**

Imagine:

- Worker Node 1 has insufficient memory.
- Worker Node 2 has enough memory.
- Worker Node 3 is also available.

The Scheduler evaluates the available resources and selects a suitable Node.

**Remember:** Scheduler decides where a Pod should run.

### 5.3 etcd

**What is etcd?**

etcd is a distributed key-value store used by Kubernetes to store important cluster information.

Think of it as the database of Kubernetes.

It stores information about the cluster, including:

- Cluster configuration.
- Kubernetes resource information.
- Desired state of resources.
- Cluster metadata.

**Real-World Example:**

Imagine you have created five Pods in your Kubernetes cluster.

Kubernetes needs to maintain information about these resources.

etcd stores the relevant cluster state information.

If you need to recover a cluster, etcd's stored data is important for restoring its state.

**Remember:** etcd is the key-value store that maintains Kubernetes cluster data.

### 5.4 Controller Manager

**What is Controller Manager?**

The Controller Manager manages different controllers inside Kubernetes.

A controller continuously checks whether the actual state of the cluster matches the desired state.

If there is a difference, the controller works to bring the cluster back to the desired state.

**Responsibilities:**

- Runs Kubernetes controllers.
- Monitors cluster resources.
- Helps maintain the desired state.
- Manages different control loops.

**Real-World Example:**

Imagine you specified:

```yaml
replicas: 3
```

You want three Pods running.

But suddenly one Pod fails.

The ReplicaSet controller detects that the required number of replicas is not available and works to create another Pod.

The Controller Manager runs and manages the relevant controllers.

**Remember:** Controller Manager manages the controllers responsible for maintaining the desired state.

### 5.5 Cloud Controller Manager (CCM)

**What is Cloud Controller Manager?**

Cloud Controller Manager is a Kubernetes Control Plane component that integrates Kubernetes with cloud platforms.

It helps Kubernetes communicate with the underlying cloud provider.

Examples:

- AWS
- Microsoft Azure
- Google Cloud Platform

**Why do we need it?**

Imagine you are running Kubernetes on AWS using Amazon EKS.

You want to create a cloud Load Balancer through Kubernetes.

Kubernetes needs to communicate with AWS to request the required cloud infrastructure.

Cloud Controller Manager helps connect Kubernetes with cloud-provider-specific functionality.

**Responsibilities:**

- Integrates Kubernetes with cloud providers.
- Helps manage cloud Load Balancers.
- Supports cloud-specific networking.
- Helps manage cloud-related resources.

**Real-World Example:**

Imagine you create a Kubernetes Service of type LoadBalancer on a supported cloud environment.

The cloud integration can request the cloud provider to create an external Load Balancer.

**Important Point:**

Cloud Controller Manager is relevant when Kubernetes integrates with a cloud provider. It is not required in the same way for a purely on-premises cluster.

**Remember:** Cloud Controller Manager connects Kubernetes with cloud-provider-specific services.

---

## 6. Control Plane Components – Summary

| Component | Main Responsibility |
|---|---|
| API Server | Receives and processes API requests |
| Scheduler | Decides where Pods should run |
| etcd | Stores cluster information |
| Controller Manager | Manages Kubernetes controllers |
| Cloud Controller Manager | Integrates with cloud providers |

---

## 7. Complete Kubernetes Architecture

Now let's combine everything we learned.

Kubernetes architecture has two major parts.

### Control Plane

1. API Server
2. Scheduler
3. etcd
4. Controller Manager
5. Cloud Controller Manager

### Data Plane – Worker Nodes

1. Kubelet
2. Kube-proxy
3. Container Runtime

Every Worker Node has its own Kubelet, Kube-proxy and Container Runtime in the architecture explained in class.

**Important Difference:**

- Control Plane controls and manages the cluster.
- Data Plane executes and runs application workloads.

---

## 8. How Does Pod Creation Work in Kubernetes?

This is one of the most important concepts from today's class.

Let's understand how all these components work together when a user creates a Pod.

**Scenario:** You want to deploy an NGINX application in Kubernetes.

### Step-by-Step Process

1. The DevOps engineer creates a Pod YAML file.
2. The engineer executes `kubectl apply -f pod.yaml`.
3. The request reaches the API Server.
4. The API Server processes and validates the request.
5. The desired resource state is stored in etcd.
6. The Scheduler selects a suitable Worker Node.
7. The API Server records the scheduling result.
8. Kubelet on the selected Worker Node receives the Pod specification.
9. Kubelet communicates with the Container Runtime.
10. The Container Runtime downloads the required image if needed and runs the container.
11. The Pod starts running.
12. Kubelet reports the Pod status to the API Server.

### Simple Pod Creation Flow

```text
DevOps Engineer
       |
       v
kubectl apply -f pod.yaml
       |
       v
   API Server
       |
       v
      etcd
       |
       v
   Scheduler
       |
       v
  Worker Node
       |
       v
    Kubelet
       |
       v
Container Runtime
       |
       v
   Container
       |
       v
   Running Pod
```

This is the basic Pod creation flow discussed in class.

---

## 9. Difference Between Control Plane and Data Plane

| Control Plane | Data Plane |
|---|---|
| Manages the cluster | Runs application workloads |
| Makes scheduling decisions | Executes scheduled workloads |
| Maintains cluster state | Reports workload status |
| Contains five major components discussed in class | Contains three major components discussed in class |

**Easy Way to Remember:**

Control Plane = Brain

Data Plane = Execution

---

## 10. Important Commands

The following commands help us interact with and inspect a Kubernetes cluster.

### Create a Pod using a YAML file

```bash
kubectl apply -f pod.yaml
```

### Check running Pods

```bash
kubectl get pods
```

### Check Worker Nodes

```bash
kubectl get nodes
```

---

## 11. Important Interview Questions

**1. What are the two major parts of Kubernetes architecture?**

Control Plane and Data Plane.

**2. What are the components of the Control Plane?**

- API Server
- Scheduler
- etcd
- Controller Manager
- Cloud Controller Manager

**3. What are the components of a Worker Node?**

- Kubelet
- Kube-proxy
- Container Runtime

**4. What is API Server?**

API Server is the central component that exposes the Kubernetes API and receives requests to manage cluster resources.

**5. What is the responsibility of Scheduler?**

Scheduler selects a suitable Worker Node for newly created Pods.

**6. What is etcd?**

etcd is a distributed key-value store that stores Kubernetes cluster information.

**7. What is Kubelet?**

Kubelet is an agent running on every Worker Node that ensures assigned Pods and their containers are running properly.

**8. What is Kube-proxy?**

Kube-proxy helps implement Kubernetes Service networking and traffic routing.

**9. What is Container Runtime?**

Container Runtime is responsible for executing containers inside Pods.

**10. What is Controller Manager?**

Controller Manager manages Kubernetes controllers that help maintain the desired state of the cluster.

**11. What is Cloud Controller Manager?**

Cloud Controller Manager integrates Kubernetes with cloud-provider-specific services and infrastructure.

**12. What is the difference between Control Plane and Data Plane?**

Control Plane manages the Kubernetes cluster, whereas Data Plane runs the application workloads.

**13. How does Kubernetes create a Pod?**

The user sends a request through the API Server. The Scheduler selects a suitable Worker Node, and Kubelet communicates with the Container Runtime to start the container inside the Pod.

---

## 12. Additional Technical Notes

A few important technical details to remember:

1. Docker Shim was discussed as part of the Docker comparison. Kubernetes removed its built-in dockershim in version 1.24. Modern Kubernetes commonly uses containerd or CRI-O through the Container Runtime Interface (CRI).

2. Kube-proxy implements Service networking. Pod IP allocation is generally handled by the Container Network Interface (CNI) plugin.

3. The Scheduler selects a Node. Kubelet on that Node works with the Container Runtime to start the containers.

4. Kubernetes can run on a single-node cluster for learning. Production environments may use multiple Control Plane and Worker Nodes for availability.

---

## 13. What We Learned and Achieved Today

In Day 31, we understood the internal architecture of Kubernetes and the responsibilities of its major components.

### Concepts Covered

- Kubernetes Architecture.
- Control Plane and Data Plane.
- Docker vs Kubernetes Architecture.
- Kubelet.
- Kube-proxy.
- Container Runtime.
- API Server.
- Scheduler.
- etcd.
- Controller Manager.
- Cloud Controller Manager.
- Pod Creation Workflow.
- Communication between Kubernetes components.

### Final Understanding

Kubernetes is divided into two major parts: Control Plane and Data Plane.

The Control Plane manages the cluster, processes requests, makes scheduling decisions and maintains cluster information.

The Data Plane consists of Worker Nodes where Pods and containers actually run.

Every component has a specific responsibility, and all these components work together to manage containerized applications.

**The main goal of today's class was to understand how Kubernetes works internally and how its different components communicate when we create a Pod.**

---

## 14. Day 31 Assignment – Teacher's Instructions

Our teacher gave us an assignment to demonstrate our understanding of Kubernetes architecture.

### Tasks

- Prepare detailed notes about Kubernetes architecture.
- Explain all Control Plane components.
- Explain all Worker Node components.
- Draw a Kubernetes architecture diagram.
- Explain how a Pod gets created.
- Show how different components communicate with each other.
- Upload the documentation to GitHub.
- Share the GitHub documentation link on LinkedIn.

**Goal:** Demonstrate that we understand Kubernetes architecture and can explain the purpose of each component rather than simply memorizing component names.

---

## Day 31 Complete – Kubernetes Architecture

**Next Topic: Kubernetes Pods**

Learn • Build • Break • Fix • Grow
