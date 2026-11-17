# Kubernetes Ingress – Theory and Concepts

## 1. What is Kubernetes Ingress?

Kubernetes Ingress is used to manage external HTTP and HTTPS traffic coming into a Kubernetes cluster.

It helps us control which application should receive the incoming request.

For example, imagine we have three applications running inside Kubernetes:

- Frontend application
- Backend application
- Admin application

Instead of exposing each application separately, we can use Ingress to manage traffic for all three applications through a common entry point.

In simple words, **Ingress helps us route incoming traffic to the correct Kubernetes Service based on the domain name or URL path.**

## 2. Why Do We Need Ingress?

In our previous practical, we learned about Kubernetes Services.

Services provide:

1. Service Discovery
2. Load Balancing
3. Application Exposure

Then why do we need Ingress?

There are two main problems that Ingress helps to solve.

### Problem 1: Limited Load Balancing Features

Before Kubernetes, companies used virtual machines and physical servers to run their applications.

They also used load balancers such as NGINX, F5 and HAProxy.

These load balancers provided many advanced features, such as:

**1. Path-Based Routing**

Sending requests to different applications based on the URL path.

Example:

```text
example.com/app1 → Application 1
example.com/app2 → Application 2
```

**2. Host-Based Routing**

Sending requests to different applications based on domain names.

Example:

```text
app1.example.com → Application 1
app2.example.com → Application 2
```

**3. Sticky Sessions**

Making sure that requests from the same user go to the same backend server.

**4. Ratio-Based Load Balancing**

Distributing traffic based on a predefined ratio.

Example:

```text
30% traffic → Application 1
70% traffic → Application 2
```

**5. TLS / HTTPS**

Handling secure communication using HTTPS and TLS certificates.

**6. Whitelisting and Blacklisting**

Allowing or blocking requests from specific IP addresses.

When companies moved their applications from virtual machines to Kubernetes, they wanted these advanced load balancing features.

But Kubernetes Services mainly provided basic traffic distribution and did not provide all these advanced features by themselves.

This created the first problem.

### Problem 2: Cost of Multiple LoadBalancer Services

Kubernetes allows us to expose applications using a Service of type LoadBalancer.

But every time we create a LoadBalancer Service in a cloud environment, the cloud provider may create a separate external load balancer.

Imagine a company has 100 microservices.

If all 100 applications use separate LoadBalancer Services, the company may need 100 external load balancers.

This can increase infrastructure costs.

Instead of creating a separate load balancer for every application, companies wanted to use one common entry point to manage traffic for multiple applications.

**This is where Kubernetes Ingress comes into the picture.**

## 3. How Does Ingress Solve These Problems?

Ingress helps us manage incoming HTTP and HTTPS traffic through a common entry point.

Instead of exposing every application separately, we can use an Ingress Controller and configure rules to route traffic to different Services.

For example:

```text
             External Users
                    |
                    v
             Ingress Controller
                    |
          ---------------------
          |         |         |
          v         v         v
       Service 1 Service 2 Service 3
          |         |         |
          v         v         v
       App 1     App 2     App 3
```

Here, multiple applications can share one external entry point.

Ingress checks the incoming request and sends it to the correct Service.

This can also help reduce the number of external load balancers required.

## 4. What is an Ingress Resource?

An Ingress Resource is a Kubernetes object that contains rules for routing incoming HTTP and HTTPS traffic.

It tells Kubernetes where incoming requests should go.

For example:

```text
app1.example.com → Service 1
app2.example.com → Service 2

example.com/app3 → Service 3
```

We create an Ingress Resource using a YAML file.

But creating an Ingress Resource alone is not enough to make traffic routing work.

We also need something called an Ingress Controller.

## 5. What is an Ingress Controller?

An Ingress Controller is the component that reads the Ingress Resource and applies the routing rules.

In simple words:

- Ingress Resource defines what we want.
- Ingress Controller makes it work.

For example, if I create an Ingress Resource saying:

```text
app1.example.com → Service 1
app2.example.com → Service 2
```

The Ingress Controller reads these rules and configures the load balancer to route traffic correctly.

### Popular Ingress Controllers

Some commonly used Ingress Controllers are:

1. NGINX
2. HAProxy
3. F5
4. Traefik
5. Ambassador

Different organizations choose an Ingress Controller based on their requirements.

For example, if a company already uses NGINX, it may choose an NGINX-based Ingress Controller.

**Important:** We must install an Ingress Controller in our Kubernetes cluster before the Ingress routing rules can work.

## 6. Difference Between Ingress Resource and Ingress Controller

These two concepts are very important.

| Ingress Resource | Ingress Controller |
|---|---|
| It is a Kubernetes resource. | It is the component that implements the routing rules. |
| Defines routing rules. | Reads and applies routing rules. |
| Created using a YAML file. | Installed inside the Kubernetes cluster. |
| Tells where traffic should go. | Handles the actual traffic routing. |

**Simple example:**

Think of Ingress Resource as instructions and Ingress Controller as the person following those instructions.

## 7. How Does Ingress Work?

Let's understand the complete flow.

1. A user sends an HTTP or HTTPS request.
2. The request reaches the Ingress Controller.
3. The Ingress Controller checks the Ingress rules.
4. It identifies the correct Kubernetes Service.
5. The Service forwards the request to one of its backend Pods.
6. The application processes the request.
7. The response is returned to the user.

### Traffic Flow

```text
             External User
                   |
                   v
              HTTP / HTTPS
                   |
                   v
          Ingress Controller
                   |
                   v
             Ingress Rules
                   |
          -------------------
          |        |        |
          v        v        v
       Service 1 Service 2 Service 3
          |        |        |
          v        v        v
         Pods     Pods     Pods
```

## 8. Types of Routing in Ingress

There are different ways to route incoming traffic using Ingress.

### 8.1 Host-Based Routing

Host-based routing sends requests to different applications based on the domain name.

Example:

```text
app1.example.com → Service 1

app2.example.com → Service 2

app3.example.com → Service 3
```

Here, different applications use different domain names but can share the same external entry point.

**Real-world example:**

Imagine a company has three applications:

- shop.example.com
- payment.example.com
- admin.example.com

Each domain can send traffic to a different application.

### 8.2 Path-Based Routing

Path-based routing sends requests to different applications based on the URL path.

Example:

```text
example.com/app1 → Service 1

example.com/app2 → Service 2

example.com/app3 → Service 3
```

Here, all applications use the same domain name, but different URL paths.

**Real-world example:**

A company can host multiple applications using one domain.

For example:

- example.com/frontend
- example.com/backend
- example.com/admin

Ingress checks the URL path and sends the request to the correct Service.

### 8.3 TLS-Based Routing

Ingress can also manage secure HTTPS traffic using TLS.

TLS helps protect communication between the user and the application.

For example:

```text
https://app1.example.com
```

We can configure TLS certificates in Ingress to enable HTTPS access.

This helps secure the communication between users and applications.

## 9. What is TLS Termination?

TLS termination means handling and decrypting HTTPS traffic at the Ingress Controller or load balancer.

Let's understand it with an example.

```text
          External User
                |
                | HTTPS
                v
        Ingress Controller
                |
          TLS Termination
                |
                | HTTP
                v
          Kubernetes Service
                |
                v
             Backend Pod
```

Here:

1. The user sends an HTTPS request.
2. The Ingress Controller receives the request.
3. It handles the TLS certificate and decrypts the traffic.
4. It forwards the request to the backend Service.

The backend connection can also use HTTPS if required.

## 10. Difference Between Kubernetes Service and Ingress

This is an important interview question.

| Kubernetes Service | Kubernetes Ingress |
|---|---|
| Provides stable access to Pods. | Manages external HTTP and HTTPS traffic. |
| Provides Service Discovery. | Provides host-based and path-based routing. |
| Supports ClusterIP, NodePort and LoadBalancer. | Routes traffic to Kubernetes Services. |
| Provides basic traffic distribution. | Supports advanced routing through an Ingress Controller. |
| Does not require an Ingress Controller. | Requires an Ingress Controller to implement routing rules. |

**In simple words:**

Service helps applications communicate and provides stable access to Pods.

Ingress manages incoming external traffic and decides which Service should receive the request.

Ingress does not replace Services. Both work together.

## 11. Difference Between LoadBalancer Service and Ingress

| LoadBalancer Service | Ingress |
|---|---|
| Exposes an individual Service externally. | Can route traffic to multiple Services. |
| May create a separate cloud load balancer for each Service. | Multiple applications can share an Ingress entry point. |
| Provides external access to applications. | Provides host-based and path-based routing. |
| Can support different types of network traffic depending on the provider. | Mainly designed for HTTP and HTTPS. |
| Usually requires cloud provider integration. | Requires an Ingress Controller. |

### Example

Imagine a company has three applications:

- Frontend
- Backend
- Admin Panel

**Using LoadBalancer Services:**

The company may create three separate LoadBalancer Services.

This can result in three external load balancers.

**Using Ingress:**

The company can use one Ingress Controller and create routing rules for all three applications.

This can reduce the number of external load balancers required.

## 12. Why Do We Need an Ingress Controller?

Creating an Ingress Resource alone will not make traffic routing work.

Kubernetes needs an Ingress Controller to watch the Ingress Resources and apply their routing rules.

For example:

If I create an Ingress Resource with host-based routing but have not installed an Ingress Controller, the routing will not work as expected.

So, before using Ingress, we need to install an Ingress Controller.

## 13. Where is Ingress Used in Real-Time Projects?

Ingress is useful when companies run multiple applications inside Kubernetes.

Some common use cases are:

1. Hosting multiple applications through one external entry point.
2. Routing traffic based on domain names.
3. Routing traffic based on URL paths.
4. Managing HTTPS communication using TLS.
5. Reducing the need for separate external load balancers.
6. Managing external access to microservices.
7. Applying additional traffic and security rules supported by the Ingress Controller.

## 14. Important Interview Questions

**Q1. What is Kubernetes Ingress?**

Kubernetes Ingress is a resource used to manage external HTTP and HTTPS traffic and route it to different Services inside a Kubernetes cluster.

**Q2. Why do we need Ingress?**

We need Ingress for advanced traffic routing such as host-based routing, path-based routing and HTTPS configuration. It also allows multiple applications to share an external entry point.

**Q3. What is an Ingress Controller?**

An Ingress Controller is the component that watches Ingress Resources and implements their routing rules.

**Q4. What is the difference between Ingress and Ingress Controller?**

Ingress defines the routing rules, while Ingress Controller reads and applies those rules.

**Q5. Can Ingress work without an Ingress Controller?**

No. An Ingress Resource alone cannot implement traffic routing. We need an Ingress Controller.

**Q6. What are the two main problems Ingress helps solve?**

1. Kubernetes Services do not provide all advanced enterprise load balancing features by themselves.
2. Creating separate LoadBalancer Services for many applications can increase cloud infrastructure costs.

**Q7. What is host-based routing?**

It routes traffic to different Services based on the domain name.

**Q8. What is path-based routing?**

It routes traffic to different Services based on the URL path.

**Q9. What is TLS termination?**

It is the process of handling and decrypting HTTPS traffic at the Ingress Controller or load balancer.

**Q10. Does Ingress replace Kubernetes Services?**

No. Ingress works with Services and routes incoming HTTP and HTTPS traffic to the correct Service.

**Q11. Name some popular Ingress Controllers.**

NGINX, HAProxy, F5 and Traefik.

**Q12. Can one Ingress Resource route traffic to multiple Services?**

Yes. We can create multiple routing rules to send traffic to different Services.

## 15. What I Learned

Through this topic, I learned:

1. Why Kubernetes Ingress was introduced.
2. The limitations of using only Kubernetes Services.
3. How Ingress manages external application traffic.
4. What an Ingress Resource is.
5. What an Ingress Controller does.
6. The difference between Ingress Resource and Ingress Controller.
7. How host-based routing works.
8. How path-based routing works.
9. How TLS and HTTPS work with Ingress.
10. The difference between Kubernetes Service and Ingress.
11. The difference between LoadBalancer Service and Ingress.
12. Why we need to install an Ingress Controller.

## 16. References

1. [Kubernetes Official Documentation – Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
2. [Kubernetes Official Documentation – Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
3. [Kubernetes Official Documentation – Services](https://kubernetes.io/docs/concepts/services-networking/service/)
4. [Abhishek Veeramalla – Complete DevOps Course, Day 38](https://www.youtube.com/)

---

## Final Takeaway

Before learning Ingress, I understood how Kubernetes Services provide Service Discovery, Load Balancing and application exposure.

After learning Ingress, I understood how we can manage incoming traffic for multiple applications using a common entry point.

The most important thing I learned is:

**Ingress Resource defines the routing rules, while Ingress Controller implements those rules.**

Both are required to make Kubernetes Ingress work.
