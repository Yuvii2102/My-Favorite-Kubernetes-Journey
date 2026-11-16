# Kubernetes Interview Questions – Part 1

## 1. Introduction

In this document, I have covered 10 important Kubernetes interview questions based on the concepts I have learned so far.

The goal is to revise the fundamentals of Kubernetes and understand how to explain these concepts confidently during technical interviews.

---

## Question 1: What is the difference between Docker and Kubernetes?

**Docker** is a container platform used to build, package and run applications inside containers.

**Kubernetes** is a container orchestration platform used to manage containers across multiple nodes in a cluster.

### Key Differences

| Docker | Kubernetes |
|---|---|
| Used to build and run containers | Used to manage and orchestrate containerized applications |
| Can run containers on a single machine | Manages applications across multiple nodes |
| Provides container management | Provides auto-healing, scaling and service discovery |
| Does not provide Kubernetes-style orchestration by itself | Provides container orchestration capabilities |

### Why do we need Kubernetes?

Containers are temporary in nature. They can stop or fail due to different reasons.

For example, imagine I am running an application inside a Docker container. If that container stops, my application becomes unavailable until I restart it or replace it.

Kubernetes helps solve this problem through features such as:

- **Self-healing:** Recreates failed Pods when managed by a controller such as a Deployment.
- **Auto-scaling:** Can increase or decrease the number of Pods based on configured conditions.
- **Load balancing:** Distributes traffic across application Pods through Services.
- **Cluster management:** Manages applications across multiple nodes.

### Real-world example

Imagine an e-commerce application running on three Kubernetes worker nodes.

If one worker node fails, Kubernetes can schedule replacement Pods on healthy nodes, provided the cluster has enough available resources.

This helps maintain application availability.

**In simple words:** Docker helps us run containers, while Kubernetes helps us manage containerized applications at scale.

---

## Question 2: What are the main components of Kubernetes architecture?

Kubernetes architecture can be divided into two major parts:

1. Control Plane
2. Data Plane (Worker Nodes)

### A. Control Plane Components

The Control Plane manages and controls the Kubernetes cluster.

**1. API Server**

- Acts as the entry point to the Kubernetes cluster.
- Handles requests from users and other Kubernetes components.
- Communicates with the other components of Kubernetes.

**2. Scheduler**

- Responsible for scheduling Pods onto suitable worker nodes.
- Checks available resources and other scheduling requirements before selecting a node.

**3. etcd**

- A distributed key-value store.
- Stores Kubernetes cluster data and configuration.
- Maintains the cluster's desired state and other important information.

**4. Controller Manager**

- Runs different Kubernetes controllers.
- Continuously monitors the actual state of resources.
- Works to bring the actual state closer to the desired state.

For example, if a Deployment requires three Pods but only two are running, the controllers help ensure that the missing Pod is created.

**5. Cloud Controller Manager**

- Connects Kubernetes with cloud provider services.
- Helps manage cloud-specific resources.
- For example, on AWS EKS, it can work with AWS integrations to provision resources such as load balancers for supported Service configurations.

### B. Data Plane Components

The Data Plane consists of worker nodes where our application Pods run.

**1. Kubelet**

- Runs on every worker node.
- Communicates with the API Server.
- Ensures that the Pods and containers assigned to its node are running as expected.

**2. Kube-proxy**

- Handles Service networking on worker nodes in clusters that use kube-proxy.
- Configures network rules to route traffic to the appropriate Pods.
- Commonly uses iptables or IPVS, depending on its configuration.

**3. Container Runtime**

- Responsible for running containers.
- Kubernetes needs a container runtime to start and manage containers.
- Examples include containerd and CRI-O.

### Quick Revision

| Component | Main Responsibility |
|---|---|
| API Server | Handles Kubernetes API requests |
| Scheduler | Schedules Pods onto nodes |
| etcd | Stores cluster data |
| Controller Manager | Runs controllers to maintain desired state |
| Cloud Controller Manager | Integrates Kubernetes with cloud providers |
| Kubelet | Manages Pods on worker nodes |
| Kube-proxy | Handles Service networking |
| Container Runtime | Runs containers |

---

## Question 3: What is the difference between Docker Swarm and Kubernetes?

Docker Swarm and Kubernetes are both container orchestration platforms.

However, they differ in features, flexibility and ecosystem support.

| Docker Swarm | Kubernetes |
|---|---|
| Easy to set up and use | Has a more detailed setup and learning process |
| Suitable for simpler orchestration requirements | Suitable for complex and large-scale environments |
| Provides basic orchestration features | Provides advanced orchestration capabilities |
| Has a smaller ecosystem | Has a large ecosystem and community |
| Offers comparatively limited extensibility | Supports extensions through Custom Resource Definitions and controllers |

### Why is Kubernetes preferred?

Kubernetes provides advanced capabilities such as:

- Automated scaling
- Self-healing
- Advanced networking
- Rolling updates
- Service discovery
- Integration with monitoring and deployment tools
- Custom Resource Definitions (CRDs)

### Real-world example

For a small application with simple deployment requirements, Docker Swarm may be sufficient.

For a large e-commerce application with multiple microservices, multiple teams and complex deployment requirements, Kubernetes provides more flexibility and a larger ecosystem.

**In simple words:** Docker Swarm is simpler to use, while Kubernetes provides a more extensive platform for managing complex containerized applications.

---

## Question 4: What is the difference between a Docker Container and a Kubernetes Pod?

A Docker container is an isolated environment where an application runs.

A Kubernetes Pod is the smallest deployable unit in Kubernetes. It can contain one or multiple containers.

### Key Differences

| Docker Container | Kubernetes Pod |
|---|---|
| Runs an application | Runs one or multiple containers |
| Managed by a container runtime or container platform | Managed through Kubernetes |
| Has its own isolated environment | Containers in the same Pod share a network namespace |
| Containers communicate through networking mechanisms | Containers in the same Pod can communicate using localhost |

### Important Pod concepts

A Pod can contain multiple containers.

When multiple containers run inside the same Pod:

- They share the same network namespace.
- They can communicate using `localhost`.
- They can share storage volumes when those volumes are configured.

### Real-world example

Imagine I have an application container and a logging sidecar container.

Both containers can run inside the same Pod.

The application generates logs, and the sidecar container collects those logs.

They can communicate using localhost because they share the Pod's network namespace.

**In simple words:** A container is the environment where an application runs, while a Pod is the Kubernetes unit that manages one or multiple containers.

---

## Question 5: What is a Namespace in Kubernetes?

A Namespace is a way to logically organize and separate resources inside a Kubernetes cluster.

In a real-world organization, multiple teams may use the same Kubernetes cluster for different projects.

Instead of creating a separate cluster for every project, we can create different namespaces.

### Real-world example

Imagine an organization has two projects:

- Project A – E-commerce application
- Project B – Banking application

We can create two namespaces:

```bash
kubectl create namespace project-a
kubectl create namespace project-b
```

Now, Project A can deploy its applications inside `project-a`, and Project B can deploy its applications inside `project-b`.

### Why do we use Namespaces?

- Logical separation of resources.
- Better resource organization.
- Resource quotas and limits can be configured.
- Access can be controlled using Role-Based Access Control (RBAC).
- Helps multiple teams work within the same cluster.

### Important point

Namespaces provide logical separation, but they are not complete security boundaries by themselves.

For stronger isolation, we can use RBAC, Network Policies and other security configurations.

**In simple words:** Namespaces help multiple teams and projects share one Kubernetes cluster while keeping their resources logically organized.

---

## Question 6: What is the role of Kube-proxy?

Kube-proxy is a networking component that runs on worker nodes in Kubernetes clusters that use it.

Its main responsibility is to implement Kubernetes Service networking by configuring network rules that forward traffic to the appropriate backend Pods.

### How does Kube-proxy work?

Imagine I create a Kubernetes Service of type NodePort.

For example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

Now users who can reach the node can access the application through:

```text
http://<NODE-IP>:30080
```

But how does the request reach the correct Pod?

Kube-proxy configures the necessary network rules so that traffic reaching the NodePort can be forwarded to the Service's backend Pods.

### Important points

- Kube-proxy handles Service networking.
- It configures network rules on worker nodes.
- It can use iptables or IPVS modes.
- It helps route Service traffic to backend Pods.

**In simple words:** Kube-proxy helps route traffic from a Kubernetes Service to the correct application Pods.

---

## Question 7: What are the different types of Services in Kubernetes?

Kubernetes Services provide three important capabilities:

1. Service discovery
2. Load balancing
3. Exposing applications

There are three main Service types covered in this lesson.

### 1. ClusterIP

- Default Service type in Kubernetes.
- Provides a stable internal IP address.
- Used for communication inside the Kubernetes cluster.
- Not directly accessible from outside the cluster.

**Example:** A backend application communicating with a database inside Kubernetes.

### 2. NodePort

- Exposes an application through a port on the Kubernetes nodes.
- Uses the node IP address and assigned NodePort.
- Can be accessed by users who can reach the node over the network.

**Example:**

```text
http://<NODE-IP>:30080
```

### 3. LoadBalancer

- Exposes an application through an external load balancer.
- Requires support from the cloud provider or another suitable load balancer implementation.
- On supported cloud platforms, Kubernetes can request a load balancer and an external IP address.

**Example:** Creating a LoadBalancer Service on AWS EKS can trigger the relevant cloud integration to provision a load balancer.

### Comparison

| Service Type | Accessibility | Main Purpose |
|---|---|---|
| ClusterIP | Inside the cluster | Internal communication |
| NodePort | Through node IP and port | Expose applications through nodes |
| LoadBalancer | Through an external load balancer | External application access |

**In simple words:**

- ClusterIP – Internal access.
- NodePort – Access through a node IP and port.
- LoadBalancer – External access through a load balancer.

---

## Question 8: What is the difference between NodePort and LoadBalancer?

Both NodePort and LoadBalancer are Kubernetes Service types used to expose applications.

However, they work differently.

### NodePort

- Exposes the application through a port on the Kubernetes nodes.
- Users need network access to the node IP address.
- The default NodePort range is 30000–32767.

Example:

```text
http://<NODE-IP>:30080
```

### LoadBalancer

- Requests an external load balancer through a supported cloud provider or implementation.
- Provides an external endpoint for accessing the application.
- Commonly used when applications need to be accessible from outside the organization's network.

### Real-world example

Imagine my Kubernetes cluster is running on AWS EKS.

If I create a NodePort Service, users who can reach the worker node IP and NodePort can access the application.

If I create a LoadBalancer Service, the AWS integration can provision a load balancer that provides an external endpoint.

### Comparison

| NodePort | LoadBalancer |
|---|---|
| Uses node IP and port | Uses an external load balancer |
| Requires network access to the node | Uses the load balancer endpoint |
| Does not automatically provision a cloud load balancer | Can provision one through cloud integration |
| Useful for direct node-level access | Useful for external application access |

**In simple words:** NodePort exposes an application through the node IP and port, while LoadBalancer provides an external load balancer endpoint.

---

## Question 9: What is the role of Kubelet?

Kubelet is an important Kubernetes component that runs on every worker node.

Its main responsibility is to manage the Pods assigned to that node and ensure that their containers are running as expected.

### How does Kubelet work?

Imagine I have a Deployment with three replicas.

```yaml
spec:
  replicas: 3
```

Kubernetes schedules the Pods onto suitable worker nodes.

Kubelet on those nodes communicates with the API Server and works with the container runtime to ensure the assigned Pods are running.

If a container inside a Pod fails, Kubelet can restart the container according to the Pod's restart policy.

If an entire Pod is deleted, its replacement is handled by the relevant controller, such as a ReplicaSet, rather than Kubelet directly recreating that Pod.

### Important responsibilities

- Monitors Pods assigned to the node.
- Communicates with the API Server.
- Works with the container runtime.
- Ensures containers are running according to the Pod specification.
- Reports Pod and node status.

**In simple words:** Kubelet manages the lifecycle of Pods assigned to a worker node and ensures that their containers run as expected.

---

## Question 10: What are your day-to-day activities as a DevOps Engineer working with Kubernetes?

This is an important interview question because interviewers want to understand what a DevOps Engineer actually does in a real-world environment.

As a DevOps Engineer, I would be responsible for managing Kubernetes clusters, deploying applications and ensuring that applications are running properly.

### My day-to-day activities would include:

**1. Managing Kubernetes Clusters**

- Monitoring cluster health.
- Checking the status of control plane and worker nodes.
- Performing cluster maintenance.
- Managing cluster upgrades.

**2. Application Deployment**

- Deploying applications using Kubernetes YAML files.
- Managing Deployments, Pods and Services.
- Updating application versions.
- Managing rolling updates and rollbacks.

**3. Monitoring and Troubleshooting**

- Checking Pod and Deployment status.
- Troubleshooting failed or pending Pods.
- Checking application logs.
- Investigating networking and Service-related issues.
- Monitoring resource utilization.

**4. Scaling Applications**

- Increasing or decreasing the number of Pod replicas.
- Configuring autoscaling where required.
- Ensuring applications have enough resources.

**5. Security and Maintenance**

- Managing access using RBAC.
- Applying security updates.
- Checking node health.
- Managing resource requests and limits.
- Ensuring the cluster follows security best practices.

**6. Supporting Development Teams**

- Helping developers troubleshoot Kubernetes issues.
- Resolving deployment problems.
- Helping teams understand Pods, Services and networking.
- Working on issues and requests through tools such as Jira.

### Interview Answer

"As a DevOps Engineer, my day-to-day activities involve managing Kubernetes clusters and ensuring that applications are deployed and running properly. I work on application deployments, scaling, monitoring and troubleshooting. I also manage cluster maintenance, upgrades and security. Apart from that, I help development teams resolve issues related to Pods, Deployments, Services and networking."

---

## 2. Final Revision – All 10 Questions

| No. | Interview Question | Key Concept |
|---|---|---|
| 1 | Difference between Docker and Kubernetes | Container platform vs orchestration |
| 2 | Main components of Kubernetes architecture | Control Plane and Data Plane |
| 3 | Docker Swarm vs Kubernetes | Container orchestration |
| 4 | Docker Container vs Kubernetes Pod | Container and Pod |
| 5 | What is a Namespace? | Logical isolation |
| 6 | Role of Kube-proxy | Service networking |
| 7 | Types of Kubernetes Services | ClusterIP, NodePort, LoadBalancer |
| 8 | NodePort vs LoadBalancer | Application exposure |
| 9 | Role of Kubelet | Pod management |
| 10 | Day-to-day activities of a DevOps Engineer | Real-world responsibilities |

---

## 3. What I Learned

After going through these interview questions, I revised the important Kubernetes concepts I have learned so far.

I now have a better understanding of:

- Docker and Kubernetes differences.
- Kubernetes architecture and its components.
- Container orchestration platforms.
- Containers and Pods.
- Namespaces and resource isolation.
- Kubelet and Kube-proxy responsibilities.
- Kubernetes Service types.
- How applications are exposed inside and outside the cluster.
- Real-world responsibilities of a DevOps Engineer.

## 4. My Goal

My goal is not just to learn Kubernetes commands but also to understand why we use each component and how everything works together in a real-world environment.

I will keep revising these questions and practice explaining the answers in my own words so that I can confidently answer them during interviews.

---

**Learning source:** Abhishek Veeramalla – Complete DevOps Course, Kubernetes Interview Questions Part 1.
