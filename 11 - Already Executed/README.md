# My Kubernetes Concepts & Projects that I have already Executed

This file contains the Kubernetes and DevOps projects I have completed as part of my Cloud and DevOps learning journey.

I have documented my hands-on practice, commands, configurations, troubleshooting steps and what I learned from each project.

**GitHub Repository:** [My-Devops-Cloud-Journey](https://github.com/Yuvii2102/My-Devops-Cloud-Journey)

**Project Folder:** `DevOps-Labs-and-Projects`

---

# 1. Kubernetes Ingress

**Project:** Kubernetes Ingress

**What I worked on:**

In this practical, I learned how Kubernetes Ingress manages incoming HTTP and HTTPS traffic and routes it to different Kubernetes Services.

I worked with:

- Kubernetes Ingress Resource
- NGINX Ingress Controller
- Host-based routing
- Path-based routing
- Ingress rules
- Minikube Ingress addon
- Routing traffic to Kubernetes Services

**What I learned:**

I understood the difference between an Ingress Resource and an Ingress Controller and how they work together to route traffic to applications.

**GitHub:** [View Kubernetes Ingress Project](https://github.com/Yuvii2102/My-Devops-Cloud-Journey/tree/main/DevOps-Labs-and-Projects/16-Kubernetes-Ingress)

---

# 2. Kubernetes Ingress Using Flask

**Project:** Kubernetes Ingress with Flask Application

**What I worked on:**

In this practical, I created a simple Flask application, containerized it using Docker and deployed it inside Kubernetes.

The complete workflow was:

```text
Flask Application
       ↓
Docker Image
       ↓
Docker Hub
       ↓
Kubernetes Deployment
       ↓
Pod
       ↓
ClusterIP Service
       ↓
Ingress Resource
       ↓
NGINX Ingress Controller
       ↓
Host-Based Routing
```

**Tools and Technologies:**

- Python
- Flask
- Docker
- Docker Hub
- Kubernetes
- Minikube
- NGINX Ingress Controller

**What I learned:**

I learned how to deploy a Flask application in Kubernetes and expose it using an Ingress Resource and host-based routing.

**GitHub:** [View Flask Ingress Project](https://github.com/Yuvii2102/My-Devops-Cloud-Journey/tree/main/DevOps-Labs-and-Projects/17-kubernetes-Ingress-2-Using-Flask)

---

# 3. Kubernetes RBAC – Practical

**Project:** Kubernetes Role-Based Access Control

**What I worked on:**

In this practical, I implemented Kubernetes RBAC using a Minikube cluster.

I worked with:

- Namespaces
- ServiceAccounts
- Roles
- RoleBindings
- ClusterRoles
- ClusterRoleBindings
- `kubectl auth can-i`
- Namespace-level permissions
- Cluster-level permissions

I created a ServiceAccount named `foo` inside the `test` namespace and tested its permissions before and after assigning RBAC permissions.

**What I learned:**

I understood how Kubernetes controls who can perform which actions on which resources.

I also learned that creating a Role alone does not grant permissions. We need a RoleBinding to connect the identity with the Role.

**GitHub:** [View Kubernetes RBAC Practical](https://github.com/Yuvii2102/My-Devops-Cloud-Journey/tree/main/DevOps-Labs-and-Projects/18-Kubernetes-RBAC)

---

# 4. Understanding Kubernetes RBAC – Part 2

**Project:** Kubernetes RBAC – Complete Concept Explanation

**What I worked on:**

In this project, I documented RBAC concepts in more detail using simple examples.

I used a company employee example to understand how Kubernetes permissions work.

The main concepts covered were:

- WHO – User or ServiceAccount
- WHAT – Role or ClusterRole
- WHERE – Namespace or Cluster
- RoleBinding
- ClusterRoleBinding
- Permissions and access control

**What I learned:**

I understood how Kubernetes RBAC connects identities with permissions and how we can control access to cluster resources.

**GitHub:** [View RBAC Understanding – Part 2](https://github.com/Yuvii2102/My-Devops-Cloud-Journey/tree/main/DevOps-Labs-and-Projects/19-Understand-RBAC-2)

---

# 5. Understanding Kubernetes RBAC – Additional Practical

**Project:** Kubernetes RBAC – Practical Understanding

**What I worked on:**

This project documents another RBAC practical where I created a namespace, ServiceAccount, Role and RoleBinding.

I tested permissions using Kubernetes commands and understood how permissions are assigned to identities.

**What I learned:**

I learned the importance of RoleBindings and how Kubernetes decides whether an identity is allowed to perform a particular action.

**GitHub:** [View RBAC Practical](https://github.com/Yuvii2102/My-Devops-Cloud-Journey/tree/main/DevOps-Labs-and-Projects/19-Understand-RBAC)

---

# 6. Kubernetes Custom Resources – Istio

**Project:** Understanding Kubernetes Custom Resource Definitions using Istio

**What I worked on:**

In this practical, I explored how external Kubernetes projects extend the Kubernetes API using Custom Resource Definitions (CRDs).

I followed the Istio installation process using Helm.

The practical covered:

- Kubernetes Custom Resource Definitions (CRDs)
- Custom Resources (CRs)
- Istio
- Helm
- Istio Base installation
- Istio control plane installation
- `istiod`
- Istio-specific Kubernetes resources

**What I learned:**

I understood how CRDs allow external projects such as Istio to introduce their own resource types into Kubernetes.

I also learned the difference between a CRD, a Custom Resource and a controller.

**Tools and Technologies:**

- Kubernetes
- Minikube
- Istio
- Helm

**GitHub:** [View Istio Custom Resources Project](https://github.com/Yuvii2102/My-Devops-Cloud-Journey/tree/main/DevOps-Labs-and-Projects/20-Custom-Resources-Istio)

---

# 7. Kubernetes ConfigMaps and Secrets

**Project:** Kubernetes ConfigMaps and Secrets – Practical

**What I worked on:**

In this practical, I learned how Kubernetes manages application configuration and sensitive information.

I worked with:

- ConfigMaps
- Secrets
- YAML configuration files
- Environment variables
- Volume mounts
- ConfigMap updates
- Secret references
- Kubernetes Deployments

I also used a Python application running inside Kubernetes to understand how configuration can be provided to containers.

**What I learned:**

I understood why application configuration should be kept separate from application code.

I also learned when to use ConfigMaps and when to use Secrets.

**GitHub:** [View ConfigMaps and Secrets Practical](https://github.com/Yuvii2102/My-Devops-Cloud-Journey/tree/main/DevOps-Labs-and-Projects/21-K8-ConfigMaps-Secrets)

---

# 8. Understanding ConfigMaps and Secrets – Part 2

**Project:** Kubernetes ConfigMaps and Secrets – Complete Understanding

**What I worked on:**

I documented the complete practical session covering configuration management in Kubernetes.

The topics included:

- Why external configuration is needed
- ConfigMaps
- Secrets
- Environment variables
- Volume mounts
- Updating ConfigMaps
- Secret creation
- Base64 encoding and decoding
- RBAC
- Least privilege
- Secret homework exercise

**What I learned:**

I understood how Kubernetes provides configuration to applications through environment variables and mounted files.

I also learned the importance of protecting sensitive application information.

**GitHub:** [View ConfigMaps and Secrets Understanding](https://github.com/Yuvii2102/My-Devops-Cloud-Journey/tree/main/DevOps-Labs-and-Projects/21-K8-ConfigMaps-Secrets-2-Understanding)

---

# 9. Kubernetes Secrets – Exercise

**Project:** Kubernetes Secrets Homework

**What I worked on:**

This practical was an exercise based on the previous ConfigMaps practical.

I repeated the configuration process using Kubernetes Secrets instead of ConfigMaps.

The exercise covered:

- Creating a Secret
- Using `secretKeyRef`
- Providing Secret values as environment variables
- Using Secrets as volume mounts
- Accessing Secret information inside a Pod

**What I learned:**

I understood how to provide sensitive configuration values to Kubernetes applications using Secrets.

**GitHub:** [View Kubernetes Secrets Exercise](https://github.com/Yuvii2102/My-Devops-Cloud-Journey/tree/main/DevOps-Labs-and-Projects/22-K8-Secrets-Exercise)

---

# 10. Kubernetes Monitoring Using Prometheus and Grafana

**Project:** Kubernetes Monitoring using Prometheus and Grafana

**What I worked on:**

In this practical, I implemented monitoring for my Kubernetes cluster using Prometheus and Grafana.

I installed Prometheus using Helm and explored how Kubernetes metrics are collected and visualized.

The practical covered:

- Helm repository configuration
- Prometheus installation
- Prometheus components
- kube-state-metrics
- Node Exporter
- Prometheus Services
- NodePort
- Prometheus UI
- PromQL queries
- Grafana installation
- Prometheus as a Grafana data source
- Grafana dashboards

**Tools and Technologies:**

- Kubernetes
- Minikube
- Helm
- Prometheus
- Grafana
- kube-state-metrics
- Node Exporter
- PromQL

**What I learned:**

I understood how Prometheus collects Kubernetes metrics and how Grafana helps visualize those metrics using dashboards.

I also learned how to query monitoring data using PromQL.

**GitHub:** [View Kubernetes Monitoring Project](https://github.com/Yuvii2102/My-Devops-Cloud-Journey/tree/main/DevOps-Labs-and-Projects/23-K8-Monitoring-using-Prometheus-and-Grafana)

**Practical Screenshots:** [View Monitoring Project Images](https://github.com/Yuvii2102/My-Devops-Cloud-Journey/tree/main/DevOps-Labs-and-Projects/23-Practicals-images-of-Monitoring/images)

---

# Overall Learning

Through these projects, I gained hands-on experience with different Kubernetes concepts.

| Area | Concepts and Tools |
|---|---|
| Traffic Management | Ingress, NGINX, Host-Based Routing, Path-Based Routing |
| Application Deployment | Flask, Docker, Docker Hub, Deployments, Services |
| Security | RBAC, ServiceAccounts, Roles, RoleBindings, Secrets |
| Kubernetes Extensions | CRDs, Custom Resources, Istio |
| Configuration Management | ConfigMaps, Secrets, Environment Variables, Volume Mounts |
| Monitoring | Prometheus, Grafana, PromQL, kube-state-metrics |
| Package Management | Helm |
| Kubernetes Environment | Minikube, AWS EC2, Ubuntu |

## Final Note

These projects are part of my hands-on Cloud and DevOps learning journey. I have documented the practical steps, commands, configurations and things I learned in my GitHub repository.

I will continue adding more projects and practicals as I learn new DevOps and Cloud technologies.
