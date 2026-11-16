# Introduction to the World of Kubernetes

## 1. Introduction to Kubernetes

In this class, we started learning Kubernetes as part of our DevOps journey.

Before understanding Kubernetes, we need to understand why it is important and what problems it solves.

Today, many companies are moving towards **Microservices Architecture**. Applications are divided into smaller services and deployed using containers.

We have already learned about Docker and containers. But when applications grow and thousands of containers need to be managed, handling everything manually becomes difficult.

This is where Kubernetes comes into the picture.

### What is Kubernetes?

Kubernetes is an **open-source Container Orchestration Platform** used to manage containerized applications.

It helps us:

- Manage containers across multiple machines.
- Automatically recover failed applications.
- Scale applications based on traffic.
- Manage application workloads.
- Handle container deployments efficiently.

### Docker vs Kubernetes

| Docker | Kubernetes |
|---|---|
| Container platform | Container orchestration platform |
| Builds and runs containers | Manages containerized applications |
| Mainly manages containers on a host | Manages workloads across a cluster |
| Provides container lifecycle management | Provides orchestration, scaling and self-healing |

**Simple Example:**

Imagine you have developed an application and created a Docker container.

If you have only one or two containers, managing them using Docker is easy.

But imagine managing 1,000 containers across multiple servers.

You need a platform that can manage all these containers automatically.

That platform is Kubernetes.

---

## 2. Prerequisite for Learning Kubernetes

Before starting Kubernetes, we should understand Docker and container fundamentals.

Our teacher explained that Docker is the main prerequisite for learning Kubernetes.

We should understand:

1. What are containers?
2. Difference between containers and virtual machines.
3. Container lifecycle.
4. Container networking.
5. Namespace isolation.
6. Why containers are lightweight.
7. Container security.
8. Multi-stage Docker builds.
9. Distroless images.

We should not focus only on Docker commands. Understanding how containers work internally is important.

---

## 3. Why Do We Need Kubernetes?

Our teacher explained four major problems that we face when managing containers using Docker alone.

### Problem 1: Single Host Problem

First, understand what a host is.

A host is a machine on which Docker is installed and containers are running. It can be an EC2 instance or a physical server.

**Example:**

Imagine we have:

- One EC2 instance.
- Docker installed on that instance.
- 100 containers running on the same machine.

Now imagine one container starts consuming too much CPU and memory.

Because all containers share the resources of the same host, other containers can also get affected.

A container might fail to start or get terminated if the host runs out of resources.

**Problem:**

Everything depends on one machine. If that machine fails, all the containers running on it can become unavailable.

**How Kubernetes solves this:**

Kubernetes uses a Cluster Architecture.

Instead of running everything on one machine, Kubernetes manages multiple machines called Nodes.

If one Node has a problem, Kubernetes can schedule affected workloads on another healthy Node if sufficient resources are available.

This reduces dependency on a single machine.

### Problem 2: Auto Healing

Containers are ephemeral in nature.

**Ephemeral means something that is temporary or short-lived.**

Containers can stop or crash because of different reasons:

- Application errors.
- Memory problems.
- Resource limitations.
- Container failures.

**Example:**

Imagine you have deployed an application inside a Docker container.

Suddenly, the container crashes.

Now your application becomes unavailable.

In basic Docker management, someone may need to restart or recreate the container unless additional restart mechanisms have been configured.

Imagine managing 10,000 containers. A DevOps engineer cannot continuously monitor every container manually.

**What is Auto Healing?**

Auto Healing is the ability to detect failed workloads and automatically recover them without requiring manual intervention.

**How Kubernetes solves this:**

Kubernetes continuously monitors the desired state of applications.

If a Pod fails, Kubernetes works to recreate it and maintain the required number of replicas.

**Example:**

You want three replicas of your application running.

If one Pod fails, Kubernetes creates another Pod to maintain the desired count of three.

This is called Auto Healing.

### Problem 3: Auto Scaling

Auto Scaling means automatically increasing or decreasing application capacity based on demand.

**Example:**

Imagine an online shopping application normally receives 10,000 users.

During a festival sale like Christmas or Dasara, the number of users suddenly increases to 100,000.

Now one container might not be sufficient to handle the increased traffic.

We need additional containers to handle the load.

There are two ways to increase the number of containers.

**1. Manual Scaling**

A DevOps engineer manually increases the number of application replicas.

Example:

- Before increased traffic: 1 replica.
- After increased traffic: 10 replicas.

**2. Automatic Scaling**

The platform automatically increases or decreases the number of replicas based on configured conditions.

Kubernetes supports automatic scaling through a feature called HPA.

#### What is HPA?

HPA stands for **Horizontal Pod Autoscaler**.

It automatically increases or decreases the number of Pod replicas based on metrics such as CPU and memory utilization.

**Example:**

Imagine your application has one Pod.

You configure HPA with a CPU utilization target of 80%.

When the workload increases and CPU utilization goes above the configured target, HPA can increase the number of Pods.

When the workload decreases, it can reduce the number of Pods.

This is called Horizontal Scaling.

### Load Balancing

When the number of application containers increases, incoming traffic must be distributed across them.

This is where Load Balancing becomes important.

**Example: Netflix**

Imagine Netflix receives requests from millions of users.

All users should not depend on one container.

A Load Balancer distributes incoming requests across multiple application instances.

The user simply accesses netflix.com without worrying about which container is handling the request.

Kubernetes provides Services for basic traffic distribution across Pods.

Additional tools can provide more advanced load balancing capabilities.

### Problem 4: Enterprise-Level Requirements

Running containers in a real company requires more than simply starting containers.

Enterprise applications require several additional capabilities.

Examples:

- Load Balancing.
- Firewall and security policies.
- Auto Scaling.
- Auto Healing.
- API Gateways.
- IP Whitelisting.
- IP Blacklisting.

Docker alone does not provide all these capabilities as a complete container orchestration solution.

**How Kubernetes helps:**

Kubernetes provides a platform that supports these capabilities through its built-in features and integrations with additional tools.

For example:

- Services help with networking and traffic distribution.
- Ingress Controllers manage external HTTP/HTTPS traffic.
- Network Policies control communication between Pods.
- HPA handles automatic scaling.

Our teacher also explained that Kubernetes does not provide every enterprise capability by default. We use additional tools and integrations to extend its functionality.

---

## 4. Four Problems and Kubernetes Solutions

| Problems | Kubernetes Solutions |
|---|---|
| Single Host Problem | Cluster Architecture |
| Auto Scaling | Horizontal Pod Autoscaler (HPA) |
| Auto Healing | Automatic recovery of failed Pods |
| Enterprise Requirements | Built-in features and external integrations |

These four problems explain why Kubernetes is important when managing applications at scale.

---

## 5. Kubernetes Cluster Architecture

Now that we understand why Kubernetes is needed, let's understand its basic architecture.

### What is a Cluster?

A Cluster is a group of machines working together.

Kubernetes uses a cluster architecture to manage application workloads across multiple machines.

A Kubernetes cluster generally contains:

1. Control Plane (Master Node)
2. Worker Nodes

### A. Control Plane

The Control Plane is the brain of the Kubernetes cluster.

It manages and controls the cluster.

Its responsibilities include:

- Managing the cluster.
- Making scheduling decisions.
- Monitoring the desired state.
- Managing application workloads.

Our teacher mentioned that Kubernetes has an API Server, which is one of the important components of its architecture.

The detailed architecture and components will be explained in the upcoming classes.

### B. Worker Nodes

Worker Nodes are the machines where application workloads run.

They contain Pods that run containers.

**Example:**

Imagine a Kubernetes cluster with:

- 1 Control Plane.
- 3 Worker Nodes.
- Multiple application Pods.

The Control Plane manages the cluster, while the Worker Nodes run the application workloads.

### Single Node vs Multi-Node Cluster

Kubernetes can run on a single machine for development and learning purposes.

However, production environments generally use multiple Nodes to improve availability and distribute workloads.

---

## 6. ReplicaSet

Our teacher introduced ReplicaSet while explaining Auto Scaling and Auto Healing.

### What is a ReplicaSet?

A ReplicaSet is a Kubernetes resource that ensures a specified number of Pod replicas are running.

**Example:**

Imagine you want three copies of your application running.

You specify:

```yaml
replicas: 3
```

Kubernetes tries to maintain three running Pods.

If one Pod fails, the ReplicaSet works to create another Pod.

### Replication Controller vs ReplicaSet

Our teacher briefly mentioned:

- Replication Controller is the older Kubernetes resource.
- ReplicaSet is its newer successor.

Deployments commonly manage ReplicaSets to maintain application replicas and manage updates.

---

## 7. YAML in Kubernetes

Our teacher explained that Kubernetes uses YAML files to define resources and configurations.

### What is YAML?

YAML stands for YAML Ain't Markup Language.

It is a human-readable data serialization format.

In Kubernetes, we use YAML files to describe what we want Kubernetes to create and manage.

For example, we can define:

- Deployments.
- Pods.
- Services.
- ReplicaSets.

Example:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: my-app

spec:
  replicas: 3

  selector:
    matchLabels:
      app: my-app

  template:
    metadata:
      labels:
        app: my-app

    spec:
      containers:
        - name: my-container
          image: nginx
          ports:
            - containerPort: 80
```

### Important YAML Fields

| Field | Purpose |
|---|---|
| apiVersion | Specifies the API version |
| kind | Specifies the Kubernetes resource type |
| metadata | Contains resource information |
| spec | Defines the desired configuration |
| replicas | Specifies the desired number of Pods |
| containers | Defines the containers |
| image | Specifies the container image |

For example, `replicas: 3` tells Kubernetes that we want three replicas of our application.

---

## 8. Kubernetes Auto Healing – How It Works

Let's understand the basic working of Auto Healing.

1. A DevOps engineer deploys an application in Kubernetes.
2. Kubernetes maintains the desired number of replicas.
3. One of the application Pods fails.
4. Kubernetes detects that the actual state does not match the desired state.
5. Kubernetes works to create a replacement Pod.
6. The application returns to the desired state.

This process happens automatically through Kubernetes controllers.

The API Server is an important part of this process. We will understand its role in the upcoming architecture class.

---

## 9. Ingress Controllers

Our teacher briefly introduced Ingress Controllers while discussing enterprise-level requirements.

Imagine you have deployed multiple applications in Kubernetes.

You want users to access them using different domain names.

For example:

- frontend.example.com
- api.example.com
- admin.example.com

You need a way to manage incoming HTTP and HTTPS traffic and direct it to the correct application.

This is where Ingress comes into the picture.

### What is an Ingress Controller?

An Ingress Controller manages external HTTP/HTTPS traffic according to configured Ingress rules.

Example:

- frontend.example.com → Frontend Service.
- api.example.com → Backend Service.
- admin.example.com → Admin Service.

Our teacher mentioned NGINX as an example of an Ingress Controller.

---

## 10. Kubernetes and CNCF

Our teacher also discussed the history and community behind Kubernetes.

### What is Google Borg?

Before Kubernetes, Google used an internal container management system called Borg.

Kubernetes was developed at Google based on experience gained from systems like Borg.

Kubernetes was later released as an open-source project.

### What is CNCF?

CNCF stands for **Cloud Native Computing Foundation**.

It supports the cloud-native ecosystem through open-source projects and community collaboration.

Kubernetes is one of the important projects associated with CNCF.

Other cloud-native projects include:

- Prometheus.
- Helm.
- Fluentd.
- containerd.

### Custom Resource Definitions (CRDs)

Our teacher briefly introduced Custom Resource Definitions.

CRDs allow us to extend the Kubernetes API with custom resource types.

Additional controllers can then manage these resources and provide functionality beyond Kubernetes' built-in capabilities.

**Example:**

Kubernetes provides basic Service networking.

For more advanced traffic management, we can integrate additional tools and controllers.

Our teacher mentioned Ingress Controllers as an example of extending Kubernetes functionality.

---

## 11. Topics We Will Learn in Upcoming Classes

At the end of the first class, our teacher explained the upcoming Kubernetes topics.

The learning sequence includes:

1. Kubernetes Architecture.
2. Pods.
3. Deployments.
4. Services.
5. Ingress Controllers.
6. Admission Controllers.

We will understand these concepts step by step through theory and practical implementation.

---

## 12. Important Interview Questions

**1. What is Kubernetes?**

Kubernetes is an open-source container orchestration platform used to deploy, manage and scale containerized applications.

**2. What is the difference between Docker and Kubernetes?**

Docker is a container platform used to build and run containers, whereas Kubernetes is a container orchestration platform used to manage containerized applications across a cluster.

**3. What is a Kubernetes Cluster?**

A Kubernetes Cluster is a group of machines working together to run and manage containerized applications.

**4. What is Auto Healing?**

Auto Healing is the ability to automatically recover failed application workloads and maintain the desired state.

**5. What is Auto Scaling?**

Auto Scaling automatically increases or decreases application capacity based on demand.

**6. What is HPA?**

HPA stands for Horizontal Pod Autoscaler. It automatically adjusts the number of Pod replicas based on configured metrics.

**7. What is a ReplicaSet?**

A ReplicaSet ensures that a specified number of Pod replicas are running.

**8. What is a Worker Node?**

A Worker Node is a machine in a Kubernetes cluster where application Pods run.

**9. What is YAML?**

YAML is a human-readable data serialization format used to define Kubernetes resources and configurations.

**10. What is an Ingress Controller?**

An Ingress Controller manages external HTTP/HTTPS traffic and routes requests to the appropriate Kubernetes Services.

---

## 13. What We Learned and Achieved Today

In this first Kubernetes class, we understood the importance of Kubernetes in the DevOps world.

We learned:

- What Kubernetes is.
- Difference between Docker and Kubernetes.
- Why Kubernetes is required.
- Problems with managing containers at scale.
- Single Host Problem.
- Auto Healing.
- Auto Scaling.
- Horizontal Pod Autoscaler.
- Load Balancing.
- Enterprise-level requirements.
- Kubernetes Cluster Architecture.
- Control Plane and Worker Nodes.
- ReplicaSet.
- YAML basics.
- Ingress Controllers.
- CNCF and the Kubernetes ecosystem.

### Final Understanding

Docker helps us build and run containers.

But when applications grow and we need to manage hundreds or thousands of containers across multiple machines, manual management becomes difficult.

Kubernetes solves this problem by providing container orchestration.

It helps us manage application workloads, recover failed Pods, scale applications and manage containers across a cluster.

**The main goal of this first class was to understand WHY we need Kubernetes before learning HOW Kubernetes works.**

