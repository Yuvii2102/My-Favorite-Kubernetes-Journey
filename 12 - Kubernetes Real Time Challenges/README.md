# 3 Real-Time Challenges Faced in Kubernetes by DevOps Engineers

## 1. Introduction

In this topic, I learned about three common challenges that DevOps Engineers face while managing Kubernetes clusters in production environments.

These are important because interviewers may ask about real-time Kubernetes problems and how we solve them.

The three challenges covered are:

1. Resource Sharing between Multiple Teams
2. Handling OOMKilled Issues
3. Kubernetes Cluster Upgrades

Let's understand each challenge with a real-world example.

---

# 2. Challenge 1 – Resource Sharing Between Multiple Teams

## What is the problem?

In a production environment, multiple development teams may use the same Kubernetes cluster.

For example, imagine an e-commerce application with different teams:

- Web Team – Handles frontend applications
- Payments Team – Handles payment services
- Transactions Team – Handles transaction services
- Delivery Team – Handles shipment services

Instead of creating a separate Kubernetes cluster for every team, all teams may share one cluster.

We can create separate Namespaces for each team.

For example:

```text
Kubernetes Cluster
        |
        | 
   +----+----+----------------+
   |         |                |
   v         v                v
Web       Payments        Delivery
Namespace Namespace       Namespace
```

But creating separate Namespaces alone does not control how much CPU and memory each team can consume.

## Understanding the problem with an example

Assume our Kubernetes cluster has:

- 5 Worker Nodes
- 20 CPU per node
- 20 GB RAM per node

Total cluster resources:

```text
Total CPU = 100 CPU
Total RAM = 100 GB
```

Now imagine the Payments Team deploys multiple microservices.

Normally, these services should consume around 6 GB RAM.

But because of a memory leak, they start consuming 32 GB RAM.

As a result, other applications may not get enough resources.

Some Pods may fail because of insufficient memory.

This can affect applications running in other Namespaces too.

## Solution 1 – ResourceQuota

To solve this problem, I can use **ResourceQuota**.

A ResourceQuota is a Kubernetes resource that sets an overall resource limit for a particular Namespace.

For example:

```text
Payments Namespace

Maximum RAM = 15 GB
Maximum CPU = 15 CPU
```

Even if the Namespace contains 5, 10 or 20 microservices, their total resource usage is restricted by the configured quota.

The development teams should perform performance benchmarking to identify how much CPU and memory their applications actually require.

Based on that information, the DevOps Engineer can configure ResourceQuota for each Namespace.

### Example ResourceQuota YAML

```yaml
apiVersion: v1
kind: ResourceQuota

metadata:
  name: payments-quota
  namespace: payments

spec:
  hard:
    requests.cpu: "15"
    requests.memory: 15Gi
    limits.cpu: "15"
    limits.memory: 15Gi
```

To apply it:

```bash
kubectl apply -f resource-quota.yaml
```

To check it:

```bash
kubectl get resourcequota -n payments
```

To see more details:

```bash
kubectl describe resourcequota payments-quota -n payments
```

**Important:** ResourceQuota controls the total resource requests and limits allowed in a Namespace. It does not automatically assign resources to individual Pods.

## Solution 2 – Resource Requests and Limits

Even after setting ResourceQuota, another problem can happen.

Imagine the Payments Namespace has 15 GB RAM available.

It contains five microservices.

One microservice starts leaking memory and consumes most of the Namespace's available resources.

This can affect the other Pods inside the same Namespace.

To prevent this, I can configure resource requests and limits for individual containers.

### Resource Requests

A resource request specifies the amount of CPU or memory that a container needs for scheduling.

### Resource Limits

A resource limit specifies the maximum CPU or memory that a container is allowed to use.

For example:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: payment-service

spec:
  replicas: 1

  selector:
    matchLabels:
      app: payment-service

  template:
    metadata:
      labels:
        app: payment-service

    spec:
      containers:
        - name: payment-service
          image: nginx

          resources:
            requests:
              memory: "1Gi"
              cpu: "500m"

            limits:
              memory: "2Gi"
              cpu: "1"
```

Here:

| Resource | Request | Limit |
|---|---:|---:|
| Memory | 1 GiB | 2 GiB |
| CPU | 500m | 1 CPU |

This means the container requests 1 GiB memory and 500 millicores of CPU for scheduling.

It can use up to 2 GiB memory and 1 CPU.

**Note:** If a container exceeds its memory limit, it may be terminated with an OOMKilled status. CPU limits are enforced differently: the container is throttled rather than killed.

## Understanding Blast Radius

I also learned an important industry term called **Blast Radius**.

Blast radius means the extent of the impact caused by a problem.

Let's understand how ResourceQuota and Resource Limits reduce the blast radius.

```text
Without ResourceQuota
        |
        v
Problem can affect the entire cluster
        |
        v
With ResourceQuota
        |
        v
Impact is restricted by Namespace quota
        |
        v
With Pod Resource Limits
        |
        v
Memory usage of an individual container is restricted
```

The important idea is:

- ResourceQuota controls resource consumption at the Namespace level.
- Resource requests and limits control resource allocation and consumption at the container level.

Together, they help prevent one application from consuming all available cluster resources.

---

# 3. Challenge 2 – Handling OOMKilled Issues

## What is OOMKilled?

OOMKilled means **Out Of Memory Killed**.

It happens when a container exceeds its memory limit or when the system cannot provide the memory it needs.

For example, imagine a Java microservice running inside a Kubernetes Pod.

The development team has performed performance benchmarking and identified that the application needs 8 GB RAM.

We configure the required memory limit.

But the application still starts consuming more memory than expected.

Eventually, the container may be terminated because it exceeds its memory limit.

When we check the Pod, we may see:

```text
CrashLoopBackOff
```

But remember:

**CrashLoopBackOff is a status. OOMKilled is the reason the container was terminated.**

## How do I troubleshoot this?

First, I check the Pods:

```bash
kubectl get pods
```

If a Pod is restarting, I describe it:

```bash
kubectl describe pod <pod-name>
```

I look for information such as:

```text
Reason: OOMKilled
```

I can also check the previous container logs:

```bash
kubectl logs <pod-name> --previous
```

The `--previous` option helps me inspect logs from the previous container instance if it has restarted.

## What should I do as a DevOps Engineer?

According to the approach explained in the lesson, I should first make sure that:

1. The Namespace has an appropriate ResourceQuota.
2. The Pod or container has suitable resource requests and limits.
3. The resource values are based on performance benchmarking.

If the application still gets OOMKilled despite having the required resources, I need to work with the development team to investigate the application.

## Thread Dump and Heap Dump

For example, if the application is written in Java, I can help collect diagnostic information such as:

- Thread Dump
- Heap Dump

### What is a Thread Dump?

A Thread Dump provides information about the threads running inside a Java application at a particular point in time.

It helps developers investigate thread-related problems.

### What is a Heap Dump?

A Heap Dump contains information about objects and memory usage in the Java heap.

Developers can analyze it to investigate memory leaks and excessive memory consumption.

The teacher explained that Java thread dumps can be collected using commands such as:

```bash
kill -3 <java-process-id>
```

A thread dump can also be collected using:

```bash
jstack <java-process-id>
```

For a Heap Dump, Java provides:

```bash
jmap -dump:format=b,file=heap.hprof <java-process-id>
```

These commands require the relevant Java diagnostic tools and appropriate permissions inside the container.

After collecting the required diagnostic files, I can share them with the development team.

The developers can analyze the application, identify the root cause, fix the memory issue and provide a new application version.

Then I can deploy the updated version to Kubernetes.

## Complete OOMKilled Troubleshooting Flow

```text
Application Pod
       |
       v
Pod gets OOMKilled
       |
       v
Check Pod Status
       |
       v
kubectl describe pod
       |
       v
Identify OOMKilled
       |
       v
Check Resource Requests and Limits
       |
       v
Collect Thread Dump / Heap Dump
       |
       v
Share with Development Team
       |
       v
Developers Analyze the Issue
       |
       v
New Application Version
       |
       v
Deploy Updated Application
```

**Important:** The exact diagnostic commands depend on the programming language and runtime. Thread dumps and heap dumps are examples for Java applications, not universal commands for every application.

---

# 4. Challenge 3 – Kubernetes Cluster Upgrades

## What is the problem?

Kubernetes releases new versions regularly.

For example, imagine our cluster is running Kubernetes version 1.29 and we need to upgrade it to version 1.30.

Upgrading a production Kubernetes cluster is challenging because we must make sure that applications continue running during the upgrade.

We should not simply upgrade all the nodes at once.

We need a proper upgrade plan.

## How do I prepare for an upgrade?

As explained in the lesson, I should prepare a detailed upgrade manual before starting.

The manual should include:

1. Backup of important Kubernetes resources.
2. Reading Kubernetes release notes.
3. Checking deprecated and removed features.
4. Checking compatibility requirements.
5. Planning the control plane upgrade.
6. Planning the worker node upgrade.
7. Testing the upgrade process.

## Step 1 – Take Backups

Before performing an upgrade, I should take the required backups.

For example, in a kubeadm cluster, etcd backup is especially important because etcd stores Kubernetes cluster state.

I should also identify any other application data and configuration that needs protection.

## Step 2 – Read Release Notes

This is a very important step.

Before upgrading Kubernetes, I should read the release notes for the target version.

Why?

Because a new release may include:

- Breaking changes
- Deprecated APIs
- Removed features
- Changes in default behavior
- Version compatibility requirements

If I skip this step, existing applications or configurations may stop working after the upgrade.

## Step 3 – Upgrade Control Plane Components

The control plane is responsible for managing the Kubernetes cluster.

Some important control plane components are:

| Component | Responsibility |
|---|---|
| kube-apiserver | Exposes the Kubernetes API |
| kube-scheduler | Schedules Pods onto nodes |
| kube-controller-manager | Runs controller processes |
| etcd | Stores Kubernetes cluster data |

In a kubeadm-based cluster, the upgrade process must follow the supported version and component order.

The exact commands depend on the Kubernetes version and installation method.

## Step 4 – Upgrade Worker Nodes

After the control plane upgrade, I need to upgrade the worker nodes.

The lesson explained the rolling upgrade approach.

Imagine our cluster has three worker nodes:

```text
Worker Node 1
Worker Node 2
Worker Node 3
```

Instead of upgrading all three nodes at once, I upgrade them one by one.

### What is Cordon?

Cordon makes a node unschedulable.

This means Kubernetes will not schedule new Pods onto that node.

Command:

```bash
kubectl cordon <node-name>
```

### What is Drain?

Drain safely evicts eligible Pods from a node so they can be scheduled on other available nodes.

Command:

```bash
kubectl drain <node-name> --ignore-daemonsets
```

The exact drain options depend on the workloads running on the node. For example, local storage and PodDisruptionBudgets may require additional planning.

## Rolling Upgrade Process

```text
Worker Node 1
      |
      v
Cordon Node 1
      |
      v
Drain Node 1
      |
      v
Upgrade Node 1
      |
      v
Bring Node 1 Back
      |
      v
Uncordon Node 1
      |
      v
Worker Node 2
      |
      v
Repeat the Process
      |
      v
Worker Node 3
      |
      v
Repeat the Process
```

After upgrading a node, I can make it schedulable again:

```bash
kubectl uncordon <node-name>
```

Then I move to the next worker node.

This approach helps reduce application downtime because I upgrade the nodes one at a time.

**Important:** This is a high-level rolling upgrade explanation. Real upgrade commands and ordering depend on whether the cluster uses kubeadm, Amazon EKS or another Kubernetes platform. The control plane and worker node versions must follow the platform's supported version-skew policy.

---

# 5. Quick Comparison of the Three Challenges

| Challenge | Problem | Solution |
|---|---|---|
| Resource Sharing | One team consumes too many cluster resources | ResourceQuota and container resource requests/limits |
| OOMKilled | Container is terminated because of memory exhaustion | Check resource configuration and investigate application memory usage |
| Cluster Upgrades | Upgrading Kubernetes without disrupting workloads | Backups, release notes and a planned rolling upgrade |

---

# 6. Important Kubernetes Commands

These are some useful commands from the concepts covered in this lesson.

### ResourceQuota

```bash
kubectl get resourcequota -n <namespace>
```

```bash
kubectl describe resourcequota <quota-name> -n <namespace>
```

### Pod Resource Troubleshooting

```bash
kubectl get pods
```

```bash
kubectl describe pod <pod-name>
```

```bash
kubectl logs <pod-name> --previous
```

### Node Maintenance

```bash
kubectl get nodes
```

```bash
kubectl cordon <node-name>
```

```bash
kubectl drain <node-name> --ignore-daemonsets
```

```bash
kubectl uncordon <node-name>
```

---

# 7. Interview Questions

### 1. How do you share Kubernetes cluster resources between multiple development teams?

I create separate Namespaces for different teams and configure ResourceQuota to control the total CPU and memory resources available to each Namespace. I also configure resource requests and limits for individual containers.

### 2. What is ResourceQuota?

ResourceQuota is a Kubernetes resource used to limit the total resource requests and limits that can be used within a Namespace.

### 3. What is the difference between ResourceQuota and Resource Limits?

ResourceQuota controls resource usage at the Namespace level, whereas container resource limits control the maximum CPU or memory a particular container can consume.

### 4. What is OOMKilled?

OOMKilled means Out Of Memory Killed. It happens when a container is terminated because it exceeds its memory limit or the system cannot provide the required memory.

### 5. Is CrashLoopBackOff an error?

CrashLoopBackOff is a Pod status indicating that a container repeatedly crashes and Kubernetes is delaying its restart attempts. We need to investigate the actual reason, such as OOMKilled or an application error.

### 6. How do you troubleshoot OOMKilled?

I check the Pod using `kubectl describe pod`, inspect its resource configuration and review the container logs. If required, I work with the development team to collect application diagnostic information such as Java thread dumps and heap dumps.

### 7. What is a Thread Dump?

A Thread Dump provides information about the threads running inside a Java application at a particular moment.

### 8. What is a Heap Dump?

A Heap Dump contains information about objects in the Java heap and helps developers investigate memory-related problems.

### 9. Why should we read Kubernetes release notes before an upgrade?

Release notes explain breaking changes, deprecated APIs, removed features and other changes that may affect our existing cluster and applications.

### 10. What is the difference between Cordon and Drain?

Cordon prevents new Pods from being scheduled on a node. Drain evicts eligible existing Pods from the node so they can run elsewhere.

---

# 8. What I Learned

In this lesson, I learned about three important real-world Kubernetes challenges.

1. **Resource Sharing:** I understood how multiple development teams can share one Kubernetes cluster and how ResourceQuota helps control Namespace-level resource consumption.

2. **OOMKilled Troubleshooting:** I learned how to identify memory-related container failures and how DevOps Engineers can work with developers using diagnostic information such as thread dumps and heap dumps.

3. **Kubernetes Upgrades:** I understood why backups, release notes and a planned rolling upgrade process are important when upgrading production clusters.

The most important thing I learned is that managing Kubernetes in production is not just about deploying applications. It is also about managing resources, troubleshooting failures and maintaining the cluster safely.

---

# 9. References

- [Kubernetes Resource Quotas – Official Documentation](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
- [Kubernetes Container Resources – Official Documentation](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [Kubernetes Pod Lifecycle – Official Documentation](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [Kubernetes Cluster Upgrade – Official Documentation](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
- [Kubernetes Nodes – Official Documentation](https://kubernetes.io/docs/concepts/architecture/nodes/)

**Source for this learning note:** Abhishek Veeramalla's lesson on three real-time Kubernetes challenges faced by DevOps Engineers. The notes follow the concepts and examples explained in that lesson.
