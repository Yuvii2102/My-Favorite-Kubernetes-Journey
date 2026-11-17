# Helm in Kubernetes – Complete Notes and Practicals

## 1. Introduction to Helm

Helm is a **package manager for Kubernetes**.

Just like `apt` is used to install and manage packages in Ubuntu, Helm is used to install and manage applications and controllers in Kubernetes.

For example, in Ubuntu, if I want to install Python, I can use:

```bash
sudo apt update
sudo apt install python3
```

Similarly, if I want to install applications such as Prometheus, Grafana, Argo CD or NGINX in Kubernetes, I can use Helm instead of manually creating multiple Kubernetes YAML files.

Helm helps us to:

- Install applications in Kubernetes.
- Upgrade application versions.
- Uninstall applications.
- Customize application configurations.
- Package our own applications as Helm charts.
- Share Helm charts with other teams.

## 2. Why Do We Need Helm?

Before Helm, we could deploy applications in Kubernetes using `kubectl apply` or `kubectl create`.

However, installing a complete application can require multiple Kubernetes resources.

For example, installing Prometheus may require:

- Namespace
- ServiceAccount
- ConfigMap
- Deployment or StatefulSet
- Services
- Other supporting resources

We would have to create and maintain multiple YAML files for these resources.

Imagine that I am working as a DevOps engineer and my organization uses 20 different applications and controllers.

Without Helm, I may have to:

1. Maintain separate YAML files for every application.
2. Manage different application versions.
3. Update configuration files whenever requirements change.
4. Maintain installation scripts.
5. Make sure all required resources are deployed correctly.

This becomes difficult when multiple teams use different versions and configurations.

**Helm solves this problem** by packaging all the required Kubernetes resources into a single package called a Helm chart.

Instead of manually managing many YAML files, I can use Helm commands to install, upgrade and uninstall applications.

## 3. Important Helm Components

There are three important concepts that I learned:

1. Helm Repository
2. Helm Chart
3. Helm Release

### 3.1 Helm Repository

A Helm repository is a centralized location where Helm charts are stored.

For example, the Bitnami repository contains charts for many popular applications.

We first add a repository and then search for the charts available inside it.

### 3.2 Helm Chart

A Helm chart is a package that contains all the required Kubernetes configuration files and templates needed to deploy an application.

For example:

- Prometheus has a Helm chart.
- Grafana has a Helm chart.
- NGINX has a Helm chart.
- Argo CD has a Helm chart.

We can also create our own Helm charts for the microservices developed in our organization.

### 3.3 Helm Release

A release is an installed instance of a Helm chart.

For example, if I install an NGINX chart with the release name `nginx-v1`, then `nginx-v1` is the name of that release.

We can install the same chart multiple times using different release names and configurations.

**Simple example:**

| Component | Example |
|---|---|
| Repository | Bitnami |
| Chart | NGINX |
| Release | nginx-v1 |

## 4. Helm Architecture

Helm 3 does not require any additional server-side component to be installed inside the Kubernetes cluster.

Helm runs from our local machine or terminal and communicates with the Kubernetes cluster using the Kubernetes configuration file (`kubeconfig`).

It works similarly to `kubectl`.

If I have multiple Kubernetes clusters configured, Helm uses the current Kubernetes context to determine which cluster to communicate with.

Check the current context:

```bash
kubectl config current-context
```

List available contexts:

```bash
kubectl config get-contexts
```

Change the current context:

```bash
kubectl config use-context <context-name>
```

Check whether Helm is installed:

```bash
helm version
```

## 5. Installing Helm

Helm can be installed on different operating systems.

For Ubuntu, follow the official Helm installation documentation:

https://helm.sh/docs/intro/install/

The teacher explained that Helm should be installed on the machine from which we manage our Kubernetes cluster.

We do not need to install Helm inside every Kubernetes cluster.

After installation, verify it:

```bash
helm version
```

## 6. Working with Helm Repositories

Before installing an application, we need to add the repository that contains its Helm chart.

In this practical, we use the Bitnami repository.

### Step 1: Add the Bitnami repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Here:

- `helm repo add` adds a new Helm repository.
- `bitnami` is the name we give to the repository.
- The URL points to the repository.

### Step 2: List available repositories

```bash
helm repo list
```

### Step 3: Update repository information

```bash
helm repo update
```

This refreshes the information about the charts available in the repositories.

### Step 4: Search for charts

To search for charts available in the Bitnami repository:

```bash
helm search repo bitnami
```

To search for NGINX:

```bash
helm search repo bitnami/nginx
```

To search for Prometheus:

```bash
helm search repo bitnami/prometheus
```

The search command helps us find whether a particular chart is available in the repository.

## 7. Installing Applications Using Helm

Now that the repository is added, we can install applications using Helm.

### 7.1 Install NGINX

```bash
helm install nginx-v1 bitnami/nginx
```

Here:

- `helm install` installs a chart.
- `nginx-v1` is the release name.
- `bitnami/nginx` is the repository and chart name.

Check the deployed resources:

```bash
kubectl get pods
```

Check Helm releases:

```bash
helm list
```

### 7.2 Install Prometheus

First, search for the chart:

```bash
helm search repo bitnami/prometheus
```

Install it:

```bash
helm install prometheus bitnami/prometheus
```

Check the pods:

```bash
kubectl get pods
```

Check the releases:

```bash
helm list
```

Depending on the chart configuration, the installation may create multiple Kubernetes resources.

## 8. Installing Charts from Other Repositories

Not every chart is available in the Bitnami repository.

For example, if I want to install an application or controller that is maintained by another organization, I need to add the repository provided by that organization.

The teacher demonstrated this using the AWS Load Balancer Controller.

The general process is:

1. Open the official documentation for the application.
2. Find its Helm installation instructions.
3. Add the repository mentioned in the documentation.
4. Update the repository.
5. Search for the required chart.
6. Install it using Helm.

The commands depend on the application and its official documentation.

**Important:** Some charts have prerequisites and require additional configuration values. Running `helm install` without those values may not be enough.

## 9. Uninstalling Applications Using Helm

We can uninstall an application using its release name.

First, list the installed releases:

```bash
helm list
```

Uninstall NGINX:

```bash
helm uninstall nginx-v1
```

Uninstall Prometheus:

```bash
helm uninstall prometheus
```

Verify:

```bash
helm list
kubectl get pods
```

We need to provide the release name, not the chart name.

## 10. Creating Our Own Helm Charts

This is an important part of the session.

As a DevOps engineer, I may need to package my organization's microservices as Helm charts so that other teams can deploy them easily.

The teacher explained this using an e-commerce application.

Imagine I am working in an e-commerce company called **Best Commerce**.

The company has two microservices:

1. Payments
2. Shipping

My task is to create a Helm chart for each microservice.

For this demonstration, the teacher used the BusyBox Docker image instead of a complete application source code.

The purpose was to understand how Helm charts work.

### Step 1: Create the project directory

```bash
mkdir -p best-commerce
cd best-commerce
```

### Step 2: Create the Payments chart

```bash
helm create payments
```

### Step 3: Create the Shipping chart

```bash
helm create shipping
```

Check the directory structure:

```bash
ls
```

We should now have:

```text
best-commerce/
├── payments/
└── shipping/
```

Helm automatically creates the required chart structure when we use `helm create`.

## 11. Understanding the Helm Chart Structure

Inside each chart, Helm creates some important files and directories.

Example:

```text
payments/
├── Chart.yaml
├── values.yaml
├── charts/
└── templates/
```

### 11.1 Chart.yaml

This file contains metadata about the Helm chart.

It can contain:

- Chart name
- Chart version
- Application version
- Description
- Chart type

Example:

```yaml
apiVersion: v2
name: payments
description: Helm chart for payments service
type: application
version: 0.1.0
appVersion: "1.0.0"
```

### 11.2 values.yaml

This file contains the default configuration values used by the templates.

For example, we can define:

```yaml
replicaCount: 2

image:
  repository: busybox
  tag: latest
  pullPolicy: IfNotPresent

appMessage: "I am a payment service"
```

Instead of hardcoding these values in every Kubernetes YAML file, we can define them in `values.yaml`.

We can change these values depending on the environment.

For example:

- UAT environment may need 2 replicas.
- Production environment may need 3 replicas.

### 11.3 templates Directory

The `templates` directory contains Kubernetes YAML templates.

For example:

```text
templates/
├── deployment.yaml
├── service.yaml
└── serviceaccount.yaml
```

We can place the Kubernetes resources required by our application inside this directory.

Helm processes these templates and replaces the variables with values from `values.yaml`.

### 11.4 charts Directory

This directory is used for chart dependencies.

For example, if our application depends on another chart, we can manage that dependency here.

For our basic demonstration, we do not need to modify this directory.

## 12. Understanding Templates and Values

This is one of the most important concepts in Helm.

**Templates** contain the Kubernetes resource definitions.

**Values** provide the configuration that is inserted into those templates.

For example, consider this Deployment template:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deployment
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: payments
  template:
    metadata:
      labels:
        app: payments
    spec:
      containers:
        - name: payments
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          command: ["/bin/sh", "-c"]
          args:
            - "echo {{ .Values.appMessage }}; sleep 3600"
```

In this template:

- `{{ .Release.Name }}` gets the release name.
- `{{ .Values.replicaCount }}` gets the replica count from `values.yaml`.
- `{{ .Values.image.repository }}` gets the image repository.
- `{{ .Values.image.tag }}` gets the image tag.
- `{{ .Values.appMessage }}` gets the application message.

For example, if `values.yaml` contains:

```yaml
replicaCount: 2

image:
  repository: busybox
  tag: latest

appMessage: "I am a payment service"
```

Helm uses these values while generating the final Kubernetes YAML resources.

This makes our chart reusable and customizable.

## 13. Creating the Payments Chart

Move into the Payments chart:

```bash
cd payments
```

Open the templates directory:

```bash
cd templates
```

We can remove the default example Deployment template and create our own.

Create or edit:

```bash
vim deployment.yaml
```

Add the Deployment template shown in the previous section.

Now move back to the Payments chart directory:

```bash
cd ..
```

Open the values file:

```bash
vim values.yaml
```

Use:

```yaml
replicaCount: 2

image:
  repository: busybox
  tag: latest
  pullPolicy: IfNotPresent

appMessage: "I am a payment service"
```

Check `Chart.yaml` and update the application version:

```yaml
appVersion: "1.0.0"
```

Now the Payments chart has its template and configuration values.

## 14. Creating the Shipping Chart

Move back to the main project directory:

```bash
cd ..
```

Open the Shipping chart:

```bash
cd shipping
```

Open its templates directory:

```bash
cd templates
```

Create or update:

```bash
vim deployment.yaml
```

Use the same Deployment template structure, but use the Shipping label and message.

For example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deployment
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: shipping
  template:
    metadata:
      labels:
        app: shipping
    spec:
      containers:
        - name: shipping
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          command: ["/bin/sh", "-c"]
          args:
            - "echo {{ .Values.appMessage }}; sleep 3600"
```

Move back to the Shipping chart directory:

```bash
cd ..
```

Update `values.yaml`:

```yaml
replicaCount: 2

image:
  repository: busybox
  tag: latest
  pullPolicy: IfNotPresent

appMessage: "I am a shipping service"
```

Update the application version in `Chart.yaml`:

```yaml
appVersion: "1.0.0"
```

Now both microservices have their own Helm charts.

## 15. Packaging Helm Charts

After creating our charts, we need to package them.

A Helm chart can be packaged into a `.tgz` archive.

First, return to the main project directory:

```bash
cd ..
```

Package the Shipping chart:

```bash
helm package shipping
```

Package the Payments chart:

```bash
helm package payments
```

Check the files:

```bash
ls -l
```

We should see files similar to:

```text
payments-0.1.0.tgz
shipping-0.1.0.tgz
```

These are the packaged Helm charts.

## 16. Creating Our Own Helm Repository

Now we have two packaged charts:

- Payments
- Shipping

We want to make them available through a common Helm repository.

The teacher used the `helm repo index` command.

Run:

```bash
helm repo index .
```

This generates an `index.yaml` file.

Check the file:

```bash
ls -l
```

The directory should contain:

```text
best-commerce/
├── payments/
├── shipping/
├── payments-0.1.0.tgz
├── shipping-0.1.0.tgz
└── index.yaml
```

The `index.yaml` file contains information about the charts available in the repository.

We can publish the packaged charts and the index file to a location accessible to other teams.

For example, an organization may use:

- Nexus Repository
- JFrog Artifactory
- GitHub Pages
- Another HTTP-accessible chart repository

Other teams can then add that repository and install the charts.

## 17. Customizing Helm Values During Installation

Sometimes we want to change chart configurations while installing an application.

Helm supports the `--set` option for this.

For example:

```bash
helm install payments-release ./payments \
  --set replicaCount=3
```

This installs the local Payments chart with three replicas.

We can also inspect the default values provided by a chart:

```bash
helm show values bitnami/nginx
```

This displays the chart's configurable values.

We can use this information to understand which configurations can be customized during installation.

## 18. Important Helm Commands – Quick Revision

| Command | Purpose |
|---|---|
| `helm version` | Check Helm version |
| `helm repo add` | Add a repository |
| `helm repo list` | List repositories |
| `helm repo update` | Update repository information |
| `helm search repo` | Search for charts |
| `helm install` | Install a chart |
| `helm list` | List installed releases |
| `helm uninstall` | Uninstall a release |
| `helm create` | Create a new chart |
| `helm package` | Package a chart |
| `helm repo index` | Generate repository index |
| `helm show values` | Display chart configuration values |

## 19. What I Learned

In this session, I learned:

1. Helm is a package manager for Kubernetes.
2. Why Helm is useful for managing multiple Kubernetes applications.
3. The difference between a repository, chart and release.
4. How to install Helm on a machine.
5. How Helm uses the current Kubernetes context.
6. How to add and update Helm repositories.
7. How to search for charts in a repository.
8. How to install and uninstall applications using Helm.
9. How to create our own Helm charts using `helm create`.
10. The purpose of `Chart.yaml`, `values.yaml` and the `templates` directory.
11. How to package Helm charts.
12. How to generate an `index.yaml` file for our own Helm repository.
13. How to customize chart configurations using `--set`.

## 20. Interview Questions

**1. What is Helm?**

Helm is a package manager for Kubernetes. It helps us install, upgrade, uninstall and manage applications using Helm charts.

**2. Why do we use Helm?**

We use Helm to simplify Kubernetes application deployment and configuration management. It avoids manually maintaining multiple YAML files for every application.

**3. What is a Helm chart?**

A Helm chart is a package containing templates and configuration files required to deploy an application in Kubernetes.

**4. What is a Helm repository?**

A Helm repository is a location where Helm charts are stored and shared.

**5. What is a Helm release?**

A release is an installed instance of a Helm chart.

**6. What is the difference between Chart.yaml and values.yaml?**

`Chart.yaml` contains metadata about the chart, whereas `values.yaml` contains default configuration values used by the templates.

**7. What is the purpose of the templates directory?**

It contains Kubernetes YAML templates such as Deployment, Service and ConfigMap definitions.

**8. Does Helm 3 require Tiller inside the Kubernetes cluster?**

No. Helm 3 does not require Tiller. It communicates with the Kubernetes API using the configured Kubernetes context.

**9. How do you uninstall an application installed through Helm?**

```bash
helm uninstall <release-name>
```

**10. How do you create a Helm chart?**

```bash
helm create <chart-name>
```

## 21. References

I used the following resources while learning and documenting this topic:

- Teacher: Abhishek Veeramalla – Helm session.
- Official Helm documentation: https://helm.sh/docs/
- Helm installation guide: https://helm.sh/docs/intro/install/
- Helm chart documentation: https://helm.sh/docs/topics/charts/
- Helm command reference: https://helm.sh/docs/helm/

---

## Final Understanding

Before learning Helm, I was creating and managing Kubernetes YAML files manually to deploy applications.

Now I understand that Helm makes this process easier by packaging Kubernetes resources into reusable charts.

I can use existing repositories to install third-party applications such as Prometheus and NGINX. I can also create my own charts for microservices such as Payments and Shipping, package them and share them through a Helm repository.

**My main takeaway:** Helm helps DevOps engineers manage Kubernetes application deployments in a simpler, reusable and organized way.
