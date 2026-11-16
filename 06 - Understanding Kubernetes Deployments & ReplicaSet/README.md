# Understanding Kubernetes Deployments

## 1. Introduction

In Kubernetes, we can deploy an application directly using a Pod. But managing applications using standalone Pods is not recommended for real-world production environments.

Why?

Because a standalone Pod does not provide the management and recovery capabilities we need when an application fails or when more users access it.

Kubernetes provides a resource called **Deployment** to manage applications more effectively.

A Deployment helps us manage:

- Multiple replicas of an application
- Automatic replacement of failed or deleted Pods
- Scaling applications
- Rolling updates

The main goal is to maintain the desired state of our application.

## 2. Understanding Containers

A container is a lightweight, isolated environment used to run an application and its dependencies.

For example, when we run an NGINX application using Docker, Docker creates a container from the NGINX image.

We can configure a container using Docker commands.

For example:

```bash
docker run -d -p 8080:80 nginx
```

Here:

- `docker run` creates and starts a container.
- `-d` runs the container in detached mode.
- `-p 8080:80` maps host port 8080 to container port 80.
- `nginx` is the container image.

In Docker, we provide these configurations through command-line options.

Kubernetes provides a different approach. Instead of passing all the configurations through commands, we define them in YAML manifest files.

## 3. Understanding Pods

A Pod is the smallest deployable unit in Kubernetes.

It provides an environment to run one or more containers.

For example, suppose we want to deploy an NGINX application.

Kubernetes runs the NGINX container inside a Pod.

A Pod can contain:

- A single container
- Multiple containers that work together

### Why would a Pod contain multiple containers?

Sometimes an application depends on another container to perform additional tasks.

For example, in a service mesh architecture, we may have:

- Main application container
- Sidecar container

The sidecar container can handle supporting tasks such as proxying network traffic.

Containers inside the same Pod share the Pod's network namespace. This allows them to communicate using `localhost`.

They can also share storage volumes when those volumes are configured for them.

### Important point

A Pod YAML file defines the specifications required to run the container or containers inside that Pod.

However, a standalone Pod does not automatically get recreated if it is deleted.

This is one of the main reasons we use Deployments.

## 4. What is a Kubernetes Deployment?

A Deployment is a Kubernetes resource used to manage applications through Pods.

Instead of creating and managing individual Pods ourselves, we create a Deployment.

The Deployment manages a ReplicaSet, and the ReplicaSet manages the required number of Pods.

For example, suppose we want to run three copies of our NGINX application.

We can specify:

```yaml
spec:
  replicas: 3
```

Kubernetes then works to maintain three Pods.

If one Pod is deleted, Kubernetes creates another one to maintain the desired number.

### Why do we use Deployments?

Deployments help us with:

1. Managing multiple Pod replicas.
2. Replacing deleted or failed Pods.
3. Scaling applications.
4. Managing application updates.
5. Maintaining the desired state of applications.

This makes Deployments suitable for managing stateless applications in real-world Kubernetes environments.

## 5. What is a ReplicaSet?

A ReplicaSet is a Kubernetes resource responsible for maintaining the required number of Pod replicas.

It makes sure that the actual number of Pods matches the desired number.

For example:

Suppose our Deployment specifies three replicas.

```yaml
spec:
  replicas: 3
```

The ReplicaSet makes sure that three matching Pods are maintained.

If one Pod gets deleted, the ReplicaSet detects the difference and creates a replacement.

We do not normally create ReplicaSets manually when using Deployments. Kubernetes creates and manages them automatically.

### Main responsibility of ReplicaSet

Its main responsibility is to maintain the desired number of Pods.

## 6. Difference Between Container, Pod, Deployment and ReplicaSet

| Concept | Meaning |
|---|---|
| Container | Runs an application and its dependencies |
| Pod | Provides the environment to run one or more containers |
| ReplicaSet | Maintains the desired number of Pod replicas |
| Deployment | Manages ReplicaSets and provides application rollout and update management |

### Simple example

Imagine an online shopping application.

- Container: Runs the shopping application.
- Pod: Provides the environment for the application container.
- ReplicaSet: Maintains three copies of the application Pod.
- Deployment: Manages the ReplicaSet and controls the application's desired state and updates.

## 7. How Deployment, ReplicaSet and Pods Work Together

This is one of the most important concepts in Kubernetes.

When we create a Deployment, Kubernetes follows this structure:

**Deployment → ReplicaSet → Pods → Containers**

Let's understand each step.

1. We create a Deployment YAML file.
2. Kubernetes creates a Deployment resource.
3. The Deployment creates a ReplicaSet.
4. The ReplicaSet creates the required number of Pods.
5. Each Pod runs its defined containers.

We do not need to create the ReplicaSet and Pods manually.

The Deployment manages this process through Kubernetes controllers.

## 8. What is a Kubernetes Controller?

A Kubernetes controller is a control loop that continuously checks the actual state of resources against the desired state.

If the actual state does not match the desired state, the controller takes action to correct it.

### Understanding Desired State and Actual State

**Desired State:**

The configuration we specify in our YAML manifest.

Example:

```yaml
spec:
  replicas: 3
```

This means we want three Pods.

**Actual State:**

The current condition of the resources in the Kubernetes cluster.

For example, the cluster currently has two Pods.

The ReplicaSet controller detects the difference and creates another Pod.

The goal is to bring the actual state back to the desired state.

This process is called **Reconciliation**.

## 9. What is Self-Healing in Kubernetes?

Self-healing means Kubernetes can automatically take action when a managed application instance is deleted or fails.

For example:

1. We create a Deployment with three replicas.
2. Kubernetes creates three Pods.
3. Someone accidentally deletes one Pod.
4. The ReplicaSet notices that only two Pods remain.
5. It creates another Pod.
6. Kubernetes returns to the desired count of three.

The important thing to understand is that Kubernetes does not recover the deleted Pod itself. It creates a new Pod to replace it.

This automatic replacement is one of the major advantages of using Deployments.

**Real-world example:**

Imagine an e-commerce website running three application Pods.

If one Pod gets deleted, the ReplicaSet creates another one so that the application continues to have the configured number of replicas.

## 10. Scaling in Kubernetes

Scaling means increasing or decreasing the number of application replicas based on requirements.

For example:

During normal usage, an application may need only one Pod.

During a sale or high-traffic period, we may need three Pods.

We can change the replica count in the Deployment YAML.

From one replica:

```yaml
spec:
  replicas: 1
```

To three replicas:

```yaml
spec:
  replicas: 3
```

When the updated configuration is applied:

1. Kubernetes detects the change.
2. The ReplicaSet receives the updated desired count.
3. It creates two additional Pods.
4. The cluster works towards maintaining three Pods.

### Types of scaling

**Manual Scaling:**

We manually change the replica count in the Deployment configuration.

**Automatic Scaling:**

Kubernetes can automatically adjust the number of replicas based on metrics such as CPU or memory usage using a Horizontal Pod Autoscaler (HPA).

Changing the replica count manually is not the same as automatic scaling.

## 11. What is a Rolling Update?

A Rolling Update is a Deployment strategy used to gradually replace existing Pods with new Pods when an application is updated.

For example, suppose we are running three Pods with an older version of an application.

We want to deploy a newer version.

Instead of replacing all three Pods at once, Kubernetes gradually updates them according to the Deployment's rollout settings.

This helps maintain application availability during updates.

The default Deployment strategy is:

```yaml
strategy:
  type: RollingUpdate
```

Kubernetes also supports the `Recreate` strategy, which terminates the existing Pods before creating new ones.

### Important point

Rolling Updates aim to avoid application downtime, but actual availability depends on factors such as readiness checks, rollout settings and application health.

## 12. What is Zero-Downtime Deployment?

Zero-downtime deployment means updating or scaling an application without interrupting the service for users.

For example:

Suppose an application is running with three replicas.

When we update the application, Kubernetes can gradually create new Pods and terminate old Pods according to the rolling update strategy.

Similarly, when we scale from one replica to three replicas, Kubernetes creates additional Pods without deleting the existing Pod.

However, maintaining uninterrupted user traffic also requires proper application configuration and traffic management through resources such as Services and Ingress.

## 13. Why Should We Use Deployments Instead of Standalone Pods?

| Standalone Pod | Deployment |
|---|---|
| Runs one or more containers | Manages application Pods through ReplicaSets |
| Does not automatically recreate itself when deleted | Maintains the desired number of Pods |
| No built-in replica management | Supports replica management |
| No Deployment rollout management | Supports rolling updates |
| Not ideal for managing production application replicas | Commonly used for stateless production applications |

In real-world Kubernetes environments, we generally use Deployments to manage stateless applications rather than creating standalone Pods directly.

## 14. Important Kubernetes Commands to Know

| Command | Purpose |
|---|---|
| `kubectl get pods` | Lists Pods |
| `kubectl get deployments` | Lists Deployments |
| `kubectl get rs` | Lists ReplicaSets |
| `kubectl get all` | Lists several common resources in the current namespace |
| `kubectl get all -A` | Lists several common resources across namespaces |
| `kubectl get pods -w` | Watches Pod changes |
| `kubectl describe deployment nginx-deployment` | Displays detailed Deployment information |

Note: `kubectl get all` does not display every type of Kubernetes resource.

## 15. Important Interview Questions

**Q1. What is a Kubernetes Deployment?**

A Deployment is a Kubernetes resource used to manage application Pods through ReplicaSets. It supports replica management, self-healing through ReplicaSets and rolling updates.

**Q2. What is the difference between a Pod and a Deployment?**

A Pod is the smallest deployable unit in Kubernetes. A Deployment manages Pods through ReplicaSets and helps maintain the desired number of replicas.

**Q3. What is a ReplicaSet?**

A ReplicaSet maintains the desired number of Pod replicas. If a Pod is deleted, it creates another Pod to maintain the configured count.

**Q4. What happens when we delete a Pod managed by a Deployment?**

The ReplicaSet detects that the actual number of Pods is below the desired number and creates a replacement Pod.

**Q5. What is the difference between Deployment and ReplicaSet?**

A Deployment manages ReplicaSets and provides rollout and update management. A ReplicaSet maintains the desired number of Pods.

**Q6. What is self-healing in Kubernetes?**

Self-healing is Kubernetes' ability to automatically correct certain failures by bringing resources back to their desired state. For example, a ReplicaSet replaces a deleted Pod.

**Q7. What is scaling in Kubernetes?**

Scaling means increasing or decreasing the number of application replicas based on requirements.

**Q8. What is a Kubernetes controller?**

A controller is a control loop that continuously compares the desired state with the actual state and takes action to reconcile differences.

**Q9. What is a Rolling Update?**

A Rolling Update gradually replaces old application Pods with new ones without taking down all existing Pods at once.

**Q10. Why do we use Deployments instead of standalone Pods in production?**

Deployments provide replica management, automatic Pod replacement through ReplicaSets and controlled application updates. These capabilities make application management easier.

## 16. What I Learned

From this lesson, I understood:

- The difference between Containers, Pods, ReplicaSets and Deployments.
- Why standalone Pods are not suitable for managing production application replicas.
- How a Deployment creates and manages a ReplicaSet.
- How ReplicaSets maintain the desired number of Pods.
- How Kubernetes controllers maintain the desired state.
- How self-healing works when a Pod is deleted.
- How scaling increases or decreases application replicas.
- How Rolling Updates help update applications gradually.
- Why Deployments are commonly used in real-world Kubernetes environments.

## 17. Final Understanding

The most important concept from this lesson is:

**Deployment → ReplicaSet → Pods → Containers**

We define the desired state in a Deployment YAML file.

The Deployment manages the ReplicaSet, and the ReplicaSet maintains the required number of Pods.

If a Pod gets deleted, the ReplicaSet creates a replacement. If we increase the replica count, it creates additional Pods.

This is how Kubernetes helps us manage applications through self-healing, scaling and controlled updates.

The main goal is to make sure that the actual state of our application matches the desired state.
