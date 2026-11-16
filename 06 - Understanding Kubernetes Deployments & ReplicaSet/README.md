# Understanding Kubernetes Deployments & ReplicaSet

## 1. Introduction

In the previous practical, I learned how to deploy an NGINX application directly using a Kubernetes Pod.

I created a YAML file, deployed the Pod using kubectl, checked its status and accessed the application from inside the Kubernetes cluster.

But this raises an important question:

**If we can deploy applications using Pods, why do we need Deployments in Kubernetes?**

To understand this, we first need to understand the difference between a Container, a Pod and a Deployment.

## 2. Container vs Pod vs Deployment

These three concepts are closely related, but they have different responsibilities.

### 2.1 Container

A container is a running instance of an application packaged with its dependencies.

For example, when we use Docker, we can run an NGINX container using:

```bash
docker run -d \
  --name nginx \
  -p 80:80 \
  nginx
```

Here, we provide different options through the command line:

- Image name
- Container name
- Port mapping
- Volume configuration
- Network configuration

Docker uses these options to create and run the container.

### 2.2 Pod

Kubernetes follows a different approach.

Instead of providing all the container configuration through command-line options, we define the configuration in a YAML manifest.

A Pod is the smallest deployable unit in Kubernetes. It provides an environment for running one or more containers.

For example, we can define an NGINX Pod using a YAML file.

The Pod specification contains information such as:

- Container image
- Container name
- Ports
- Volumes
- Other container settings

**Can a Pod contain multiple containers?**

Yes. A Pod can contain one or more containers.

For example, an application may have:

- Main application container
- Sidecar container

Both containers can share the same network namespace and communicate using `localhost`. They can also share storage volumes when configured.

A common example is a service mesh, where a sidecar container works alongside the main application container.

### 2.3 Deployment

Now comes the important concept: Deployment.

A Deployment is a Kubernetes resource used to manage application Pods.

You might wonder why we need it when we can already create Pods directly.

The main reason is that a standalone Pod does not automatically get recreated if it is deleted or fails.

Kubernetes Deployments help us manage Pods and provide important capabilities such as:

- Maintaining the desired number of Pod replicas
- Self-healing through ReplicaSets
- Scaling the number of replicas
- Rolling updates

In real-world Kubernetes environments, we generally use Deployments instead of creating standalone Pods for stateless applications.

## 3. Why Do We Need Deployments?

Let's understand this with a real-world example.

Imagine we have deployed an online shopping application.

Initially, we run only one Pod.

Now imagine that:

- The Pod gets deleted accidentally.
- The application crashes.
- The number of users increases.
- We need multiple copies of the application to handle traffic.

If we created only a standalone Pod, Kubernetes would not automatically recreate it after deletion.

This creates a problem for application availability.

Kubernetes provides Deployments to solve this problem.

Deployments help maintain the desired number of application replicas and manage their lifecycle.

Two important capabilities we want to understand are:

### 3.1 Auto-healing

Auto-healing means Kubernetes can automatically replace Pods when they are deleted or become unavailable, provided they are managed by a controller such as a Deployment.

For example:

Imagine we have a Deployment with three replicas.

```text
Desired replicas: 3
Running replicas: 3
```

Now someone accidentally deletes one Pod.

```text
Before deletion:

Pod 1  → Running
Pod 2  → Running
Pod 3  → Running
```

After deleting Pod 2:

```text
Pod 1  → Running
Pod 2  → Deleted
Pod 3  → Running
```

The ReplicaSet notices that only two Pods are available, while the desired number is three.

It creates another Pod to maintain the desired count.

```text
After replacement:

Pod 1  → Running
Pod 2  → Deleted
Pod 3  → Running
Pod 4  → Running
```

Now the desired number of replicas is maintained again.

This is the self-healing behaviour we want from Kubernetes.

### 3.2 Scaling

Scaling means increasing or decreasing the number of application replicas based on our requirements.

For example, imagine our application initially runs with two replicas.

```text
replicas: 2
```

Later, if the application needs more capacity, we can increase the desired number to five.

```text
replicas: 5
```

Kubernetes creates additional Pods to reach the desired replica count.

This helps us run multiple copies of an application.

We can also reduce the replica count when fewer copies are required.

**Important:** Changing the replica count manually is called scaling the Deployment. Automatic scaling based on metrics such as CPU or memory requires an additional resource called Horizontal Pod Autoscaler (HPA).

## 4. Understanding ReplicaSet

Now that we understand why Deployments are required, let's learn about ReplicaSets.

A ReplicaSet is a Kubernetes resource that ensures a specified number of Pod replicas are running.

It continuously checks the actual number of Pods against the desired number.

For example:

```text
Desired replicas: 3
Actual replicas: 2
```

The ReplicaSet identifies that one Pod is missing and creates another one.

Its main responsibility is to maintain the desired number of replicas.

### How does Deployment work with ReplicaSet?

When we create a Deployment, Kubernetes automatically creates a ReplicaSet.

The ReplicaSet then creates and manages the Pods.

The relationship looks like this:

```text
Deployment
    |
    v
ReplicaSet
    |
    v
Pods
    |
    v
Containers
```

Let's understand the responsibilities:

| Resource | Responsibility |
|---|---|
| Deployment | Manages the application rollout and desired configuration |
| ReplicaSet | Maintains the desired number of Pod replicas |
| Pod | Provides the environment to run containers |
| Container | Runs the actual application |

**Important:** We do not need to create the ReplicaSet or Pods manually when using a Deployment. Kubernetes manages them for us.

## 5. Understanding Kubernetes Controllers

While learning about ReplicaSets, we come across another important term: Controller.

A Kubernetes Controller continuously observes the actual state of the cluster and works to make it match the desired state.

Let's understand this with an example.

Imagine we define a Deployment with:

```yaml
replicas: 3
```

This means our desired state is to have three Pods.

Now imagine only two Pods are running.

The actual state is different from the desired state.

The ReplicaSet controller notices this difference and creates another Pod.

Once the third Pod is running, the desired state and actual state match again.

### Desired State vs Actual State

| Desired State | Actual State |
|---|---|
| What we want | What is currently running |
| Defined in our configuration | Exists in the Kubernetes cluster |

The controller continuously works to maintain this relationship.

This is one of the important ideas behind Kubernetes.

## 6. Deployment vs ReplicaSet

These two resources are related, but they have different responsibilities.

| Deployment | ReplicaSet |
|---|---|
| Manages application rollouts | Maintains Pod replicas |
| Creates and manages ReplicaSets | Creates and manages Pods |
| Supports rolling updates and rollbacks | Ensures the desired replica count |
| Provides a higher-level way to manage applications | Works underneath Deployments |

A Deployment uses a ReplicaSet to maintain the required number of Pods.

We generally create Deployments directly instead of managing ReplicaSets ourselves.

## 7. Understanding Zero-Downtime Deployments

One important benefit of Deployments is that they support rolling updates.

Imagine we are running an application with three replicas.

Now we want to update the application to a newer version.

Instead of stopping all the existing Pods at once, Kubernetes can gradually create new Pods and terminate old ones according to the rollout strategy.

This helps keep the application available during an update.

The goal is to avoid interrupting users while the application is being updated.

Actual availability also depends on the rollout configuration, readiness checks and application health.

## 8. Important Interview Questions

### Q1. What is the difference between a Container, Pod and Deployment?

- Container: Runs the actual application.
- Pod: Smallest deployable unit in Kubernetes that contains one or more containers.
- Deployment: Manages Pods and provides capabilities such as scaling, self-healing through ReplicaSets and rolling updates.

### Q2. Why do we need Deployments if we already have Pods?

Standalone Pods do not automatically get recreated when deleted.

Deployments manage Pods through ReplicaSets and help maintain the desired number of replicas.

### Q3. What is a ReplicaSet?

A ReplicaSet is a Kubernetes resource that ensures the specified number of Pod replicas are running.

If a Pod is deleted, the ReplicaSet creates another Pod to maintain the desired count.

### Q4. What is the difference between Deployment and ReplicaSet?

A Deployment manages application rollouts and ReplicaSets.

A ReplicaSet maintains the desired number of Pod replicas.

### Q5. What is a Kubernetes Controller?

A Controller monitors the actual state of Kubernetes resources and works to make it match the desired state.

### Q6. What is Auto-healing in Kubernetes?

Auto-healing is the ability to automatically replace failed or deleted Pods through Kubernetes controllers when they are managed by resources such as Deployments.

### Q7. What is Scaling?

Scaling means increasing or decreasing the number of application replicas based on workload requirements.

### Q8. What is Zero-Downtime Deployment?

It is an application update strategy that aims to keep the application available while new versions are rolled out and old versions are replaced.

## 9. What I Learned

Through this theory session, I understood:

1. The difference between Containers, Pods and Deployments.
2. Why standalone Pods are not generally used to manage production applications.
3. How Deployments manage application Pods.
4. What ReplicaSets are and why they are important.
5. How Kubernetes controllers maintain the desired state.
6. How self-healing works when a Pod is deleted.
7. How replica scaling works.
8. How rolling updates help maintain application availability.

## 10. Final Understanding

The most important thing I learned is that Kubernetes is not just about running containers.

It also helps us manage applications and maintain their desired state.

When we create a Deployment, Kubernetes automatically creates a ReplicaSet, which manages the Pods.

The ReplicaSet ensures that the required number of Pods remains available, even if someone deletes a Pod.

This is why Deployments are preferred over standalone Pods when managing applications in real-world Kubernetes environments.
