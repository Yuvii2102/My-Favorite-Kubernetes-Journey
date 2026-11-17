# Helm – Complete Theory Notes

## 1. What is Helm?

Helm is a **package manager for Kubernetes**.

To understand Helm, let's compare it with the Ubuntu package manager.

In Ubuntu, we use `apt` to install, update and remove software packages.

For example:

```bash
sudo apt update
sudo apt install nginx
```

Similarly, Helm helps us install, upgrade and uninstall applications and Kubernetes components in a Kubernetes cluster.

For example, using Helm, we can install:

- Prometheus
- Grafana
- Argo CD
- NGINX
- Kubernetes applications developed by our organization

Without Helm, we may need to manage multiple Kubernetes YAML files manually. Helm makes this process easier by packaging the required Kubernetes resources together.

**In simple words:** Helm helps us manage Kubernetes applications easily.

---

## 2. Why Do We Need Helm?

Imagine you are working as a DevOps Engineer in an organization.

Your organization uses multiple Kubernetes applications such as:

- Prometheus for monitoring
- Grafana for dashboards
- Argo CD for GitOps
- NGINX for handling incoming traffic

To install these applications manually, you may need multiple YAML files.

For example, an application might require:

- Deployment
- Service
- ConfigMap
- ServiceAccount
- PersistentVolumeClaim
- Other Kubernetes resources

Managing all these YAML files manually can become difficult.

There are also situations where different environments need different configurations.

For example:

| Environment | Replicas |
|---|---:|
| Development | 1 |
| UAT | 2 |
| Production | 3 |

If we manage everything manually, maintaining different configurations becomes difficult.

Helm solves this problem.

### Benefits of Helm

1. **Easy installation:** Install applications using simple Helm commands.
2. **Easy upgrades:** Upgrade applications and their configurations.
3. **Easy uninstallation:** Remove installed applications using Helm.
4. **Reusable configurations:** Use values to customize applications.
5. **Version management:** Manage different chart versions.
6. **Application packaging:** Package your organization's Kubernetes applications into Helm charts.

---

## 3. Helm Architecture and Important Components

There are three important concepts to understand:

1. Helm Repository
2. Helm Chart
3. Helm Release

### 3.1 Helm Repository

A Helm repository is a centralized location where Helm charts are stored.

Think of it like a software repository in Ubuntu.

For example, Ubuntu uses software repositories to provide packages that can be installed using `apt`.

Similarly, Helm repositories provide charts that can be installed using Helm.

Examples:

- Bitnami
- An organization's internal Helm repository
- A repository hosted using GitHub Pages

### 3.2 Helm Chart

A Helm chart is a package containing the files and templates required to deploy an application on Kubernetes.

For example, Prometheus has a Helm chart.

That chart contains the Kubernetes resources and configuration required to deploy Prometheus.

Other examples include:

- NGINX chart
- Grafana chart
- Argo CD chart
- Payments chart
- Shipping chart

A chart can contain Kubernetes templates, default configuration values and metadata.

### 3.3 Helm Release

A Helm release is an installed instance of a Helm chart.

For example:

```bash
helm install nginx-v1 bitnami/nginx
```

Here:

- `nginx-v1` is the release name.
- `bitnami/nginx` is the chart.

If we install the same chart multiple times with different release names, Helm manages them as separate releases.

For example:

```bash
helm install nginx-v1 bitnami/nginx
helm install nginx-v2 bitnami/nginx
```

Both releases use the same chart but have different release names.

**Remember:**

| Component | Meaning |
|---|---|
| Repository | Stores Helm charts |
| Chart | Package of a Kubernetes application |
| Release | Installed instance of a chart |

---

## 4. Helm 3 Architecture

In Helm 2, a component called Tiller was required inside the Kubernetes cluster.

Tiller was responsible for managing Helm releases.

From Helm 3 onwards, Tiller was removed.

Helm 3 communicates directly with the Kubernetes API server using the current Kubernetes configuration.

This means we do not need to install Tiller inside our Kubernetes cluster.

---

## 5. Helm Repositories

Before installing a chart, we need a repository that contains that chart.

One popular repository used in the teacher's demonstration is Bitnami.

We can add a Helm repository using:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Update repository information:

```bash
helm repo update
```

List the configured repositories:

```bash
helm repo list
```

Search for available charts:

```bash
helm search repo bitnami
```

Search for a particular chart:

```bash
helm search repo bitnami/nginx
```

Search for Prometheus:

```bash
helm search repo bitnami/prometheus
```

**Important:** A repository contains charts. We search for a chart inside a repository before installing it.

---

## 6. Installing Applications Using Helm

Helm allows us to install Kubernetes applications using a simple command.

General syntax:

```bash
helm install RELEASE_NAME CHART_NAME
```

Example:

```bash
helm install nginx-v1 bitnami/nginx
```

Here:

- `helm install` installs a chart.
- `nginx-v1` is the release name.
- `bitnami/nginx` is the chart name.

After installation, we can check the release:

```bash
helm list
```

Check the Kubernetes pods:

```bash
kubectl get pods
```

Check the services:

```bash
kubectl get services
```

---

## 7. Upgrading Applications Using Helm

Helm also supports upgrading an existing release.

General syntax:

```bash
helm upgrade RELEASE_NAME CHART_NAME
```

Example:

```bash
helm upgrade nginx-v1 bitnami/nginx
```

We can also change configuration values during an upgrade.

Helm helps us manage application updates without manually recreating all Kubernetes resources.

---

## 8. Uninstalling Applications Using Helm

We can uninstall an application using its release name.

General syntax:

```bash
helm uninstall RELEASE_NAME
```

Example:

```bash
helm uninstall nginx-v1
```

To check the remaining releases:

```bash
helm list
```

**Important:** We provide the release name, not the chart name, when uninstalling a release.

---

# Creating Our Own Helm Charts

Installing third-party applications is only one use of Helm.

As DevOps Engineers, we may also need to package our organization's applications into Helm charts.

Let's understand this with an example.

## 9. Real-World Example – E-commerce Application

Imagine you are working as a DevOps Engineer in an e-commerce company called Best Commerce.

The company has two microservices:

1. Payments
2. Shipping

Your responsibility is to deploy both microservices on Kubernetes.

Instead of maintaining separate deployment files and configurations manually, you can create Helm charts for both services.

Our structure will look like this:

```text
best-commerce/
├── payments/
└── shipping/
```

Each microservice will have its own Helm chart.

This makes it easier to manage, package and share the applications.

---

## 10. Creating a Helm Chart

Helm provides a command called `helm create`.

It automatically generates the required directory structure and default files for a new chart.

Example:

```bash
helm create payments
```

Create another chart:

```bash
helm create shipping
```

Helm generates a structure similar to this:

```text
payments/
├── Chart.yaml
├── values.yaml
├── charts/
└── templates/
```

The important components are explained below.

### 10.1 Chart.yaml

`Chart.yaml` contains metadata about the Helm chart.

It includes information such as:

- Chart name
- Chart version
- Application version
- Description
- Chart type

Example:

```yaml
apiVersion: v2
name: payments
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.0.0"
```

**Important fields:**

| Field | Meaning |
|---|---|
| apiVersion | Helm chart format version |
| name | Name of the chart |
| description | Description of the chart |
| type | Application or library |
| version | Version of the Helm chart |
| appVersion | Version of the application |

**Remember the difference:**

- `version` represents the Helm chart version.
- `appVersion` represents the application version.

### 10.2 Templates Folder

The `templates` folder contains Kubernetes YAML templates.

For example:

```text
templates/
├── deployment.yaml
├── service.yaml
├── serviceaccount.yaml
└── ...
```

These files define the Kubernetes resources that Helm will generate and deploy.

We can place resources such as:

- Deployment
- Service
- ConfigMap
- ServiceAccount
- Ingress

Helm processes these templates and replaces their variables with actual values.

### 10.3 values.yaml

`values.yaml` contains the default configuration values used by the templates.

For example:

```yaml
replicaCount: 1

image:
  repository: busybox
  tag: latest
  pullPolicy: IfNotPresent
```

Instead of hardcoding every value inside the Deployment YAML, we can refer to values from `values.yaml`.

For example:

```yaml
replicas: {{ .Values.replicaCount }}
```

Here, `.Values.replicaCount` gets its value from `values.yaml`.

If `replicaCount` is 1, Helm renders the Deployment with one replica.

### 10.4 charts Folder

The `charts` folder is used to store dependent charts.

For example, an application may depend on another service such as Redis.

Those dependencies can be managed through Helm charts.

For our basic Payments and Shipping demonstration, we do not need to add any dependencies.

---

## 11. How Templates and values.yaml Work Together

This is one of the most important concepts in Helm.

Imagine your Deployment template contains:

```yaml
image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

And your `values.yaml` contains:

```yaml
image:
  repository: busybox
  tag: latest
```

Helm reads the values and renders the template.

The resulting Kubernetes YAML contains:

```yaml
image: "busybox:latest"
```

So:

- Templates define the Kubernetes resources.
- `values.yaml` provides the configuration.
- Helm combines them to generate the final Kubernetes manifests.

This makes the same chart reusable with different configurations.

---

## 12. Customizing Values Using --set

Sometimes we don't want to edit the `values.yaml` file.

Helm provides the `--set` option to override values directly through the command line.

Example:

```bash
helm install my-prometheus bitnami/prometheus \
  --set server.replicaCount=2
```

Here, we are overriding the default replica count and setting it to 2.

The general syntax is:

```bash
--set key=value
```

We can also override multiple values:

```bash
helm install my-prometheus bitnami/prometheus \
  --set server.replicaCount=2 \
  --set server.service.type=NodePort
```

This allows us to customize applications during installation.

---

## 13. Viewing Default Values Using helm show values

Before installing a chart, we may want to understand which configurations are available.

Helm provides:

```bash
helm show values CHART_NAME
```

Example:

```bash
helm show values bitnami/prometheus
```

This displays the default `values.yaml` configuration provided by the chart.

We can inspect these values and decide which ones we need to customize.

---

## 14. Packaging Helm Charts

After creating and configuring our Helm chart, we can package it.

A Helm package bundles the chart files into a versioned `.tgz` archive.

General syntax:

```bash
helm package CHART_DIRECTORY
```

Examples:

```bash
helm package payments
```

```bash
helm package shipping
```

This generates files similar to:

```text
payments-0.1.0.tgz
shipping-0.1.0.tgz
```

These packaged charts can be shared with other teams or published to a Helm repository.

---

## 15. Creating a Helm Repository Index

After packaging the charts, we need an index file for the repository.

Helm provides:

```bash
helm repo index .
```

This command generates an `index.yaml` file in the current directory.

The index file contains information about the available charts, including:

- Chart name
- Chart version
- Application version
- Package location
- Package digest

Example directory structure:

```text
best-commerce/
├── payments/
├── shipping/
├── payments-0.1.0.tgz
├── shipping-0.1.0.tgz
└── index.yaml
```

The `index.yaml` file helps Helm discover the charts available in the repository.

---

## 16. Publishing a Helm Repository

After creating the chart packages and `index.yaml`, we can publish them to a centralized location.

Some examples are:

- GitHub Pages
- Nexus Repository
- JFrog Artifactory

Once the repository is hosted and accessible, other team members can add it to their Helm configuration.

General command:

```bash
helm repo add REPOSITORY_NAME REPOSITORY_URL
```

Example:

```bash
helm repo add best-commerce https://example.com/helm-charts
```

Then search for charts:

```bash
helm search repo best-commerce
```

And install a chart:

```bash
helm install payments best-commerce/payments
```

**Note:** The example URL above is only a placeholder. A real repository must be hosted with its `index.yaml` and chart packages available at the correct locations.

---

# 17. Important Helm Commands – Quick Revision

| Command | Purpose |
|---|---|
| `helm version` | Check Helm version |
| `helm repo add` | Add a Helm repository |
| `helm repo update` | Update repository information |
| `helm repo list` | List configured repositories |
| `helm search repo` | Search for charts |
| `helm install` | Install a chart |
| `helm upgrade` | Upgrade a release |
| `helm uninstall` | Uninstall a release |
| `helm list` | List installed releases |
| `helm create` | Create a new Helm chart |
| `helm package` | Package a chart |
| `helm repo index .` | Generate repository index |
| `helm show values` | Display chart default values |
| `helm get values` | Display values used by an installed release |
| `helm get manifest` | Display Kubernetes manifests generated for a release |

---

# 18. Final Summary

Helm is a package manager for Kubernetes that makes application deployment and management easier.

The three important concepts are:

- Repository – Stores Helm charts.
- Chart – Package of a Kubernetes application.
- Release – Installed instance of a chart.

The three important files and folders inside a basic Helm chart are:

- `Chart.yaml` – Stores chart metadata.
- `values.yaml` – Stores default configuration values.
- `templates/` – Contains Kubernetes YAML templates.

We can create our own charts, customize them using values, package them and publish them through a Helm repository.

**The main goal of Helm is to make Kubernetes application deployment reusable, organized and easier to manage.**
