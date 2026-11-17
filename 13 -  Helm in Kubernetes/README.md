# Helm in Kubernetes – Theory Notes

## 1. What is Helm?

Helm is a **package manager for Kubernetes**.

To understand Helm, we can compare it with the `apt` package manager in Ubuntu.

In Ubuntu, if we want to install Python, Java or any other package, we use the `apt` package manager.

Similarly, in Kubernetes, if we want to install applications or controllers such as Prometheus, Grafana, Argo CD or NGINX Ingress Controller, we can use Helm.

Without Helm, we may have to manually create and manage multiple Kubernetes YAML files for installing an application.

Helm makes this process easier by packaging all the required Kubernetes resources into a package called a **Helm chart**.

Using Helm, we can:

- Install Kubernetes applications and controllers.
- Upgrade applications to newer versions.
- Uninstall applications.
- Manage different versions of applications.
- Package our organization's applications as Helm charts.
- Share our Helm charts with other teams.

In simple words, Helm makes Kubernetes application deployment and management easier.

## 2. Why Do We Need Helm?

We can deploy applications in Kubernetes using `kubectl apply` or `kubectl create`. So why do we need Helm?

Let's understand this with a real-world example.

Imagine I am working as a DevOps engineer in an organization. My responsibility is to install and maintain different applications and tools on Linux servers.

Suppose I need to install 10 different packages, such as:

- Apache
- Java
- Node.js
- Python
- NGINX

Without a package manager, I may have to write separate shell scripts for installing each package.

These scripts may include:

- Downloading the package.
- Extracting the files.
- Configuring the application.
- Setting permissions.
- Adding the application to the system path.
- Managing dependencies.

The problem does not end with installation. I also need to maintain these scripts whenever a new version is released or the installation process changes.

If I have to manage 100 different packages, maintaining all these scripts becomes difficult.

This is where the `apt` package manager helps. Instead of manually handling all these steps, I can use the package manager to install and manage the required packages.

The same problem exists in Kubernetes.

Imagine I need to install Prometheus. It may require multiple Kubernetes resources such as:

- Namespace
- ServiceAccount
- ConfigMap
- StatefulSet or Deployment
- Services

Similarly, Argo CD and other controllers may also require multiple resources and configurations.

If I manually maintain separate YAML files and scripts for every application, it creates a lot of maintenance work for the DevOps team.

**Helm solves this problem.**

It packages all the required Kubernetes resources into Helm charts. We can then use Helm to install, upgrade and uninstall applications without manually managing every resource separately.

### Main benefits of Helm

1. **Simplifies installation:** We can install applications using Helm charts instead of managing many YAML files manually.
2. **Version management:** We can manage different versions of applications.
3. **Easy upgrades:** We can upgrade applications when newer versions are available.
4. **Easy removal:** We can uninstall applications using their release names.
5. **Reusability:** We can create charts once and reuse them in different environments.
6. **Customization:** We can customize application configurations using values.
7. **Sharing:** We can package and share our organization's applications as Helm charts.

## 3. What Can We Do Using Helm?

Helm is not only used for installing applications.

It supports different application management operations.

### 3.1 Install

We can install applications and Kubernetes controllers using Helm.

For example, Prometheus, Grafana, Argo CD and NGINX.

### 3.2 Upgrade

We can upgrade an installed application to a newer version.

For example, if an organization is using Argo CD version 2.12, it may upgrade to version 2.13.

Helm helps manage this upgrade process.

### 3.3 Uninstall

We can remove applications that are no longer required.

### 3.4 Create Our Own Helm Charts

We can also package applications developed by our organization as Helm charts.

For example, if an organization has Payments, Shipping and Transactions microservices, we can create separate Helm charts for them.

Other teams can then use those charts to deploy the applications.

## 4. Helm 3 Architecture

Helm 3 does not require any additional server-side component to be installed inside the Kubernetes cluster.

In earlier Helm versions, particularly Helm 2, a component called **Tiller** was used inside the cluster.

However, Helm 3 removed this requirement.

We only need Helm installed on the machine from which we manage our Kubernetes cluster.

### How does Helm know which Kubernetes cluster to connect to?

This works similarly to `kubectl`.

We may have multiple Kubernetes clusters configured on our machine.

For example:

- Development cluster
- Testing cluster
- Production cluster

The Kubernetes configuration file, called `kubeconfig`, contains information about these clusters and their contexts.

Helm uses the current Kubernetes context to determine which cluster it should communicate with.

So, if our current context points to the development cluster, Helm communicates with that cluster.

If we change the current context to production, Helm communicates with the production cluster.

We do not need to install Helm separately inside every cluster.

## 5. Important Components of Helm

There are three important concepts that we need to understand before working with Helm:

1. Helm Repository
2. Helm Chart
3. Helm Release

### 5.1 Helm Repository

A Helm repository is a centralized location that stores Helm charts.

We need to add a repository before searching for and installing the charts available in it.

This concept is similar to package repositories in Linux.

For example, when we install a package using `apt`, the package manager looks for that package in its configured repositories.

Similarly, Helm looks for charts in the repositories that we have added.

One popular repository mentioned by my teacher is **Bitnami**.

Bitnami provides charts for many commonly used applications and tools.

For example, its repository has included charts for applications such as Redis, Spark and other commonly used software.

However, not every application is available in the same repository.

Some organizations maintain their own Helm repositories for their applications and controllers.

### 5.2 Helm Chart

A Helm chart is a package or bundle containing the Kubernetes resources and configuration templates required to deploy an application.

For example:

- Prometheus chart
- Grafana chart
- NGINX chart
- Argo CD chart

The chart contains the required Kubernetes configuration files and templates for deploying the application.

Instead of manually managing every YAML file, we can use the chart to deploy the application.

We can also create our own charts for applications developed inside our organization.

### 5.3 Helm Release

A Helm release is a deployed instance of a Helm chart.

For example, imagine I install the NGINX chart and give it the release name `nginx-v1`.

Here:

- `nginx-v1` is the release name.
- NGINX is the chart.
- Bitnami is the repository.

The release name helps us identify a particular installation.

We can install the same chart multiple times using different release names.

We also use the release name when managing or uninstalling that particular installation.

### Understanding the relationship

| Component | Meaning | Example |
|---|---|---|
| Repository | Stores Helm charts | Bitnami |
| Chart | Package of an application | NGINX |
| Release | Installed instance of a chart | nginx-v1 |

## 6. Installing Applications from Different Repositories

Not every Kubernetes application or controller is available in the Bitnami repository.

For example, my teacher discussed the AWS Load Balancer Controller.

If a chart is not available in the repository we are currently using, we need to find the repository maintained by the organization responsible for that application.

For example, AWS provides its own Helm repository for the AWS Load Balancer Controller.

The general process is:

1. Find the official documentation for the application.
2. Identify the Helm repository provided by its maintainers.
3. Add that repository.
4. Find the required chart.
5. Install the chart with the required configuration.

Some charts also have additional prerequisites and configuration requirements.

Therefore, we should always refer to the official documentation before installing third-party controllers.

## 7. Creating Our Own Helm Charts

Apart from installing third-party applications, Helm also allows us to package our organization's applications as Helm charts.

This is especially useful when working with microservices.

My teacher explained this using an example of an e-commerce organization called Best Commerce.

Imagine the organization has two microservices:

1. Payments
2. Shipping

As a DevOps engineer, I am responsible for deploying these microservices in Kubernetes.

Instead of maintaining separate deployment instructions for every team, I can create a Helm chart for each microservice.

For example:

- Payments Helm chart
- Shipping Helm chart

I can then package both charts and publish them to a common Helm repository.

Other teams can add that repository and install the charts whenever they need them.

This makes application deployment easier and more consistent across teams.

## 8. Structure of a Helm Chart

When we create a Helm chart, Helm provides a predefined folder structure.

The important files and directories are:

```text
payments/
├── Chart.yaml
├── values.yaml
├── charts/
└── templates/
```

Let's understand each one.

### 8.1 Chart.yaml

`Chart.yaml` contains metadata about the Helm chart.

It provides information such as:

- Name of the chart.
- Chart version.
- Application version.
- Description.
- Chart type.

For example, if we create a Payments chart, its metadata identifies the chart as Payments and specifies its version.

It helps Helm understand information about the chart.

### 8.2 values.yaml

`values.yaml` contains the configuration values used by the templates.

Instead of hardcoding every configuration inside Kubernetes YAML files, we can define values separately.

For example, we may define:

- Number of replicas.
- Container image repository.
- Container image tag.
- Image pull policy.
- Resource configurations.
- Application-specific settings.

The main purpose of `values.yaml` is to make the chart customizable.

### 8.3 templates Directory

The `templates` directory contains the Kubernetes YAML templates required to deploy the application.

For example:

- Deployment
- Service
- ServiceAccount
- ConfigMap

We can place the required Kubernetes resource templates inside this directory.

Helm processes these templates and replaces the variables with the values provided in `values.yaml`.

### 8.4 charts Directory

The `charts` directory is used for managing chart dependencies.

For example, an application may depend on another application or component.

We can use chart dependencies to manage such requirements.

For the basic Payments and Shipping demonstration, my teacher explained that we could ignore this directory.

## 9. Understanding Templates and Values

This is one of the most important concepts in Helm.

**Templates define how Kubernetes resources should be created. Values provide the configuration used by those templates.**

Let's understand with an example.

Imagine we have a Payments microservice.

In the Deployment template, we may need to define:

- Deployment name.
- Number of replicas.
- Container image.
- Container image tag.

Instead of hardcoding all these values, we can use variables.

For example, the template can refer to the replica count using a value from `values.yaml`.

If the values file contains two replicas, Helm uses that value while generating the Deployment.

If we change the value to three replicas, the same template can be reused with three replicas.

This is useful when deploying applications in different environments.

For example:

| Configuration | UAT | Production |
|---|---:|---:|
| Replicas | 2 | 3 |
| Application | Payments | Payments |

The same chart can be customized for both environments.

We do not need to maintain completely separate Deployment templates for every environment.

### Why are templates and values important?

- Templates make Kubernetes resource definitions reusable.
- Values allow us to customize the configuration.
- The same chart can be used in multiple environments.
- We can avoid hardcoding configuration values in every template.

## 10. Packaging Helm Charts

After creating our own Helm charts, we can package them into chart archive files.

For example, if we have Payments and Shipping charts, we can package them separately.

The packaged files can then be shared with other teams.

The package contains the chart files and is commonly generated as a `.tgz` archive.

This makes it easier to distribute and use our charts.

## 11. Creating Our Own Helm Repository

After packaging our charts, we can make them available through a common Helm repository.

For example, Best Commerce may have the following charts:

- Payments
- Shipping

We can publish both charts to a centralized location.

The repository also contains an `index.yaml` file that provides information about the charts available in it.

Other teams can add this repository to their Helm configuration and search for the available charts.

My teacher mentioned that organizations can use centralized storage solutions such as:

- Nexus Repository
- JFrog Artifactory
- GitHub Pages

This allows teams inside or outside the organization to access and install the charts.

## 12. Customizing Helm Charts

Helm charts often provide configurable values.

When installing a chart, we may want to change some of these values according to our requirements.

For example, we may want to change:

- Number of replicas.
- Container image.
- Application configuration.
- Other deployment settings.

Helm supports customization through values provided by the chart.

We can inspect the available default values of a chart to understand which settings can be customized.

This is especially useful when installing the same application in multiple environments.

For example, the UAT environment may require two replicas, while the production environment may require three replicas.

We can use the same chart with different configuration values.

## 13. Final Understanding

Before learning Helm, I understood that Kubernetes applications could be deployed using YAML files and `kubectl`.

However, when applications require multiple resources and different configurations, manually maintaining all those files becomes difficult.

Helm makes this process easier by packaging Kubernetes resources into reusable charts.

I learned that:

- A repository stores Helm charts.
- A chart is a package containing application deployment templates and configuration.
- A release is an installed instance of a chart.
- `Chart.yaml` contains chart metadata.
- `values.yaml` contains configuration values.
- The `templates` directory contains Kubernetes resource templates.
- We can package our own charts and share them through a Helm repository.

**My main takeaway:** Helm helps DevOps engineers simplify Kubernetes application deployment, manage application versions and reuse deployment configurations across different environments.
