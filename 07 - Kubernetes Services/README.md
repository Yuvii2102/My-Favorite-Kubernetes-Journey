# Understanding Kubernetes Services

## 1. Introduction

In the previous lesson, I learned about Kubernetes Deployments, ReplicaSets and Pods.

I understood how a Deployment creates a ReplicaSet, how a ReplicaSet manages Pods and how Kubernetes automatically replaces deleted Pods.

But there is one important problem.

How do other applications or users communicate with these Pods?

This is where **Kubernetes Services** come into the picture.

A Kubernetes Service provides a stable way to access a group of Pods. It helps applications communicate with each other without depending on individual Pod IP addresses.

In this lesson, I learned why Services are important, how they work and what different types of Services are available in Kubernetes.

## 2. Why Do We Need Kubernetes Services?

Let's take an example from our previous practical.

We created an NGINX Deployment with three replicas.

This means Kubernetes created three NGINX Pods.

Each Pod receives its own IP address.

For example:

| Pod | IP Address |
|---|---|
| Pod 1 | 172.16.3.4 |
| Pod 2 | 172.16.3.5 |
| Pod 3 | 172.16.3.6 |

These are example IP addresses used for understanding.

Now imagine another application wants to communicate with our NGINX application.

Without a Service, we might share the individual Pod IP addresses with the other application.

But there is a problem.

### What happens when a Pod gets deleted?

Suppose Pod 1 gets deleted because of some failure.

We already know that Kubernetes provides self-healing through Deployments and ReplicaSets.

The ReplicaSet automatically creates a replacement Pod.

However, the new Pod may receive a different IP address.

For example:

- Old Pod IP: `172.16.3.4`
- New Pod IP: `172.16.3.8`

Now the other application is still trying to access the old IP address.

Even though Kubernetes has successfully created a replacement Pod, the other application may not be able to communicate with it using the old IP.

This creates a communication problem.

### How does Kubernetes solve this?

Instead of sharing individual Pod IP addresses, we create a Kubernetes Service.

Other applications communicate with the Service rather than directly accessing individual Pods.

The Service provides a stable access point and keeps track of the matching Pods.

This solves the problem of changing Pod IP addresses.

## 3. What is a Kubernetes Service?

A Kubernetes Service is a resource that provides a stable way to access a group of Pods.

It acts as an entry point for communication with the application.

Instead of accessing individual Pod IP addresses, we access the application through its Service.

A Kubernetes Service provides three important benefits:

1. Load Balancing
2. Service Discovery
3. Exposing Applications

Let's understand each one.

## 4. Load Balancing

Load balancing means distributing incoming requests across multiple application instances.

Imagine we have three Pods running the same NGINX application.

If multiple users access the application, we don't want all requests to depend on just one Pod.

A Service provides a stable entry point and distributes traffic across the matching Pods.

For example:

- User 1 sends requests to the Service.
- User 2 sends requests to the Service.
- User 3 sends requests to the Service.

The Service forwards traffic to the available matching Pods.

This helps distribute the workload instead of depending on a single Pod.

### Real-world example

Imagine an e-commerce website during a big sale.

Thousands of users are trying to access the website at the same time.

If the application has multiple replicas, a Service can distribute traffic across those Pods.

This helps the application handle incoming requests more effectively.

**What I learned:** Kubernetes Services provide a stable entry point and help distribute traffic across matching Pods.

## 5. Service Discovery

Service Discovery is another important feature of Kubernetes Services.

As we discussed earlier, Pod IP addresses can change whenever Pods are recreated.

So how does a Service identify the correct Pods without depending on their IP addresses?

Kubernetes uses **Labels and Selectors**.

### What are Labels?

Labels are key-value pairs attached to Kubernetes resources.

They help us identify and organise resources.

For example:

```yaml
metadata:
  labels:
    app: nginx
```

Here:

- `app` is the label key.
- `nginx` is the label value.

This label identifies our NGINX Pod.

### What are Selectors?

Selectors are used to identify resources based on their labels.

For example:

```yaml
selector:
  app: nginx
```

This tells the Service to find Pods that have the label `app: nginx`.

### How does Service Discovery work?

Let's understand with an example.

1. We create a Deployment with three replicas.
2. The Deployment creates a ReplicaSet.
3. The ReplicaSet creates three Pods.
4. All three Pods receive the same label: `app: nginx`.
5. We create a Service with the selector `app: nginx`.
6. The Service identifies the Pods that match this selector.

Now imagine one Pod gets deleted.

The ReplicaSet creates a replacement Pod using the same Pod template.

The replacement Pod receives the same label, even if its IP address is different.

The Service can discover the new Pod because it matches the selector.

This is how Kubernetes Services maintain communication with Pods without manually tracking their IP addresses.

### Important difference

- **Label:** A tag attached to a Kubernetes resource.
- **Selector:** Used to find resources with matching labels.

**What I learned:** Service Discovery works through labels and selectors, not by manually maintaining a list of Pod IP addresses.

## 6. Exposing Applications

Another important purpose of Kubernetes Services is exposing applications.

When we create a Deployment, Kubernetes creates Pods inside the cluster.

These Pods can communicate within the Kubernetes networking environment.

But what if someone outside the cluster wants to access our application?

For example, imagine we have deployed an e-commerce application.

We cannot expect every customer to connect to our Kubernetes cluster and access a Pod directly.

We need a way to expose the application.

Kubernetes Services provide different types of access depending on our requirements.

The three main Service types covered in this lesson are:

1. ClusterIP
2. NodePort
3. LoadBalancer

## 7. Types of Kubernetes Services

### 7.1 ClusterIP

ClusterIP is the default Service type in Kubernetes.

It provides access to an application from within the Kubernetes cluster.

Other applications running inside the cluster can communicate with the Service.

However, it is not directly accessible from outside the cluster.

**Example:**

Imagine an e-commerce application with:

- Frontend
- Backend
- Database

The backend needs to communicate with the database.

We can expose the database using a ClusterIP Service so that the backend can access it internally.

**When should I use ClusterIP?**

When an application needs to be accessed only by other applications inside the Kubernetes cluster.

### 7.2 NodePort

NodePort exposes an application on a specific port on each Kubernetes node.

It allows users to access the application using:

```text
<Node-IP>:<NodePort>
```

For example:

```text
192.168.1.10:30080
```

Here:

- `192.168.1.10` is an example Kubernetes node IP.
- `30080` is an example NodePort.

Anyone who can reach the node IP and the assigned port, and is allowed by the network and firewall rules, can access the application.

**When should I use NodePort?**

When I want to expose an application through the Kubernetes node IP address and a specific port.

### 7.3 LoadBalancer

LoadBalancer exposes a Service through an external load balancer provided by a supported cloud provider integration.

For example, when using a supported AWS Kubernetes environment, a LoadBalancer Service can request an AWS load balancer.

The load balancer provides an external address through which users can access the application.

**Example:**

Imagine we are working on a public e-commerce website.

We want customers from anywhere on the internet to access our application.

A LoadBalancer Service can provide an external entry point for the application.

**When should I use LoadBalancer?**

When I want to expose an application externally through a cloud provider's load balancing service.

**Important:** A LoadBalancer Service does not automatically create a working public load balancer in every Kubernetes environment. It depends on the cloud provider integration. In Minikube, additional configuration may be required.

## 8. Difference Between ClusterIP, NodePort and LoadBalancer

| Feature | ClusterIP | NodePort | LoadBalancer |
|---|---|---|---|
| Access | Inside the cluster | Through node IP and port | Through external load balancer |
| External access | No direct external access | Yes, if node is reachable | Yes, when provisioned |
| Main purpose | Internal communication | Expose through node ports | External application access |
| Common use | Backend and database communication | Development and testing | Public-facing applications |

### Simple real-world examples

| Requirement | Service Type |
|---|---|
| Backend communicates with database | ClusterIP |
| Access an application through a Kubernetes node | NodePort |
| Make an application publicly accessible through a cloud load balancer | LoadBalancer |

## 9. How Kubernetes Services Work with Deployments

Now let's connect everything we have learned.

When we create a Deployment, Kubernetes creates and manages Pods through a ReplicaSet.

We can then create a Service on top of those Pods.

The overall structure looks like this:

**Deployment → ReplicaSet → Pods → Containers**

The Service uses labels and selectors to identify the matching Pods.

It provides a stable access point and forwards traffic to the selected Pods.

### Example

Suppose we create a Deployment with three NGINX replicas.

All three Pods have this label:

```yaml
labels:
  app: nginx
```

Now we create a Service with this selector:

```yaml
selector:
  app: nginx
```

The Service identifies all three Pods and forwards traffic to the matching endpoints.

If one Pod gets deleted and a replacement is created, the Service can discover the replacement through its matching label.

This is how Services work together with Deployments and ReplicaSets.

## 10. How does Kubernetes handle Service traffic?

Kubernetes uses components such as `kube-proxy` to implement Service networking in many cluster configurations.

`kube-proxy` watches Service and EndpointSlice information and configures the necessary networking rules to direct traffic to the available endpoints.

The exact networking implementation can vary depending on the cluster and its networking solution.

For this lesson, the important thing to understand is that a Service provides a stable access point and Kubernetes maintains the information needed to route traffic to matching Pods.

## 11. Important Terms to Remember

| Term | Meaning |
|---|---|
| Service | Provides a stable way to access a group of Pods |
| Load Balancing | Distributes incoming traffic across matching Pods |
| Service Discovery | Helps applications find services and their matching Pods |
| Label | A key-value tag attached to a resource |
| Selector | Finds resources using matching labels |
| ClusterIP | Default Service type for internal cluster access |
| NodePort | Exposes a Service through a port on each node |
| LoadBalancer | Exposes a Service through an external load balancer |
| kube-proxy | Implements Service networking in many Kubernetes configurations |

## 12. Important Interview Questions

**Q1. What is a Kubernetes Service?**

A Kubernetes Service provides a stable way to access a group of Pods. It helps with load balancing, service discovery and exposing applications.

**Q2. Why do we need Services in Kubernetes?**

Pod IP addresses can change when Pods are recreated. Services provide a stable access point so that applications do not need to track individual Pod IP addresses.

**Q3. What are the three main types of Kubernetes Services?**

ClusterIP, NodePort and LoadBalancer.

**Q4. What is ClusterIP?**

ClusterIP is the default Service type. It allows applications inside the Kubernetes cluster to communicate with the Service.

**Q5. What is NodePort?**

NodePort exposes a Service on a specific port on each Kubernetes node. Users can access it through the node IP address and assigned port.

**Q6. What is LoadBalancer?**

LoadBalancer exposes a Service through an external load balancer provided by a supported cloud provider integration.

**Q7. What is Service Discovery?**

Service Discovery helps applications find the correct Pods without manually tracking changing Pod IP addresses.

**Q8. What are Labels and Selectors?**

Labels are key-value tags attached to Kubernetes resources. Selectors identify resources using matching labels.

**Q9. How does a Service identify Pods?**

A Service uses selectors to find Pods with matching labels. Kubernetes maintains the corresponding endpoint information.

**Q10. What happens when a Pod behind a Service gets deleted?**

The ReplicaSet creates a replacement Pod if it manages the deleted Pod. The new Pod receives the labels from the Pod template, and the Service can discover it through its selector.

**Q11. What is the difference between NodePort and LoadBalancer?**

NodePort exposes an application through a port on Kubernetes nodes. LoadBalancer exposes an application through an external load balancer, usually provided by a cloud platform.

## 13. What I Learned

From this lesson, I understood:

- Why Kubernetes Services are important.
- Why directly depending on Pod IP addresses is not a good approach.
- How Services provide a stable access point for applications.
- How Services help distribute traffic across matching Pods.
- How Service Discovery works.
- How Labels and Selectors help identify Pods.
- The difference between ClusterIP, NodePort and LoadBalancer.
- How Services help expose applications inside and outside the Kubernetes cluster.
- How Services work together with Deployments, ReplicaSets and Pods.

## 14. Final Understanding

The main reason we use Kubernetes Services is to make application communication easier and more reliable.

Pods can be deleted and recreated, and their IP addresses can change. We should not make other applications depend directly on those changing IP addresses.

Instead, we create a Service that provides a stable access point and identifies the correct Pods using labels and selectors.

Services also help distribute traffic and expose applications based on our requirements.

The three main Service types I learned are:

- **ClusterIP:** Internal cluster communication.
- **NodePort:** Access through Kubernetes node IP and port.
- **LoadBalancer:** External access through a cloud load balancer.

This lesson helped me understand how applications communicate with each other in Kubernetes and how we make our deployed applications accessible.
