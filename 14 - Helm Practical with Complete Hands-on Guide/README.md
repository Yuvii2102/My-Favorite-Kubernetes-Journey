# Helm Practical – Complete Hands-on Guide

## 1. Practical Overview

In this practical, I learned how to use Helm to manage Kubernetes applications and create my own Helm charts.

I performed all the practicals using:

- AWS EC2
- Ubuntu
- Minikube
- kubectl
- Helm
- Bitnami Helm repository
- BusyBox Docker image

### Goal of this practical

The main goal was to understand how Helm works and how we can create reusable Helm charts for microservices.

We used an example e-commerce company called **Best Commerce**, which has two microservices:

1. Payments
2. Shipping

We created separate Helm charts for both services, customized their configurations, packaged the charts and generated a Helm repository index.

---

# Part 1 – Getting Started with Helm

## Step 1: Verify Helm Installation

First, check whether Helm is installed.

```bash
helm version
```

**Purpose:** Displays the installed Helm version.

Example output:

```text
version.BuildInfo{
  Version:"v4.3.2",
  ...
}
```

The exact output may differ depending on your installed version.

**Where this command comes from:** Basic Helm CLI verification.

## Step 2: Check Kubernetes Cluster

Before using Helm, make sure the Kubernetes cluster is running.

```bash
kubectl get nodes
```

Expected output:

```text
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   ...   ...
```

**Explanation:**

- `kubectl` is the command-line tool used to communicate with Kubernetes.
- `get nodes` displays the Kubernetes nodes.
- `Ready` means the node is available to run workloads.

Check Minikube:

```bash
minikube status
```

This displays the status of the Minikube components.

**Where these commands come from:** Kubernetes and Minikube CLI commands used before starting the Helm practical.

---

# Part 2 – Working with Helm Repositories

## Step 3: Add the Bitnami Repository

A Helm repository stores Helm charts.

Our teacher used the Bitnami repository to demonstrate installing applications such as NGINX and Prometheus.

Add the repository:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

**Explanation:**

- `helm repo add` adds a new Helm repository.
- `bitnami` is the repository name.
- The URL points to the repository.

Expected output:

```text
"bitnami" has been added to your repositories
```

## Step 4: Update Repository Information

Run:

```bash
helm repo update
```

**Purpose:** Updates the local information about charts available in the configured repositories.

Expected output:

```text
Hang tight while we grab the latest from your chart repositories...
Update Complete.
```

## Step 5: List Helm Repositories

Run:

```bash
helm repo list
```

Example output:

```text
NAME       URL
bitnami    https://charts.bitnami.com/bitnami
```

This displays the repositories configured in Helm.

## Step 6: Search for NGINX

Run:

```bash
helm search repo nginx
```

To search specifically inside Bitnami:

```bash
helm search repo bitnami/nginx
```

**Explanation:** Helm searches the repository and displays matching charts and their versions.

## Step 7: Search for Prometheus

Run:

```bash
helm search repo prometheus
```

This displays Prometheus-related charts available in the configured repositories.

For example, the results may include:

- Prometheus
- Kube-Prometheus
- Node Exporter

**What I learned:**

We can search for applications in Helm repositories before installing them.

**Where Steps 3–7 come from:** Teacher's Helm repository demonstration.

---

# Part 3 – Installing and Uninstalling Applications

## Step 8: Install NGINX Using Helm

Our teacher demonstrated installing NGINX using a Helm chart.

Run:

```bash
helm install nginx-v1 bitnami/nginx
```

**Explanation:**

| Command | Purpose |
|---|---|
| `helm install` | Installs a Helm chart |
| `nginx-v1` | Release name |
| `bitnami/nginx` | Chart name |

Expected output will confirm that the release has been deployed.

## Step 9: Check Installed Helm Releases

Run:

```bash
helm list
```

Example:

```text
NAME       NAMESPACE   REVISION   STATUS
nginx-v1   default     1          deployed
```

**Explanation:**

A Helm release is an installed instance of a Helm chart.

Here, `nginx-v1` is the release name.

## Step 10: Check Kubernetes Resources

Check the pods:

```bash
kubectl get pods
```

Check the services:

```bash
kubectl get services
```

These commands help us verify the Kubernetes resources created by Helm.

## Step 11: Uninstall NGINX

Run:

```bash
helm uninstall nginx-v1
```

Expected output:

```text
release "nginx-v1" uninstalled
```

Verify:

```bash
helm list
```

The `nginx-v1` release should no longer appear.

**What I learned:**

Helm allows us to install and uninstall Kubernetes applications using simple commands.

**Where Steps 8–11 come from:** Teacher's NGINX installation and uninstallation demonstration.

---

# Part 4 – Understanding Helm Chart Values

## Step 12: View Default Prometheus Values

Our teacher demonstrated how to inspect the default configuration of a Helm chart.

Run:

```bash
helm show values bitnami/prometheus
```

**Purpose:** Displays the default values provided by the Prometheus chart.

The output contains many configuration options.

Some important examples are:

```yaml
server:
  replicaCount: 1

  service:
    type: LoadBalancer
```

**Explanation:**

- `server.replicaCount` controls the number of Prometheus server replicas.
- `server.service.type` controls the Kubernetes Service type.

We can use these values to customize an application during installation.

## Step 13: Understand the --set Option

The `--set` option allows us to override chart values directly from the command line.

For example:

```bash
helm install my-prometheus bitnami/prometheus \
  --set server.replicaCount=2
```

**Explanation:**

- `my-prometheus` is the release name.
- `bitnami/prometheus` is the chart.
- `server.replicaCount=2` overrides the default replica count.

This is an example of customizing a chart during installation.

**Note:** This is an example command for understanding `--set`. It is not necessary to execute it during this custom chart creation practical.

**Where Steps 12–13 come from:** Teacher's explanation of `helm show values` and `--set`.

---

# Part 5 – Creating Our Own Helm Charts

## Step 14: Create the Project Directory

Now we will create our own Helm charts.

Our example is an e-commerce company called Best Commerce.

It has two microservices:

- Payments
- Shipping

Create the main project directory:

```bash
mkdir -p ~/best-commerce
```

Enter the directory:

```bash
cd ~/best-commerce
```

Check the current location:

```bash
pwd
```

Expected output:

```text
/home/ubuntu/best-commerce
```

## Step 15: Create the Payments Helm Chart

Run:

```bash
helm create payments
```

This command automatically creates a Helm chart named `payments`.

## Step 16: Create the Shipping Helm Chart

Run:

```bash
helm create shipping
```

This creates another Helm chart named `shipping`.

Check both directories:

```bash
ls -l
```

Expected structure:

```text
best-commerce/
├── payments/
└── shipping/
```

**Explanation:**

`helm create` generates the default Helm chart structure.

We don't need to create all the folders manually.

## Step 17: Understand the Generated Chart Structure

Enter the Payments directory:

```bash
cd payments
```

Run:

```bash
ls -l
```

The generated chart contains:

```text
payments/
├── Chart.yaml
├── charts/
├── templates/
└── values.yaml
```

Important components:

| File or folder | Purpose |
|---|---|
| `Chart.yaml` | Chart metadata |
| `values.yaml` | Default configuration values |
| `templates/` | Kubernetes YAML templates |
| `charts/` | Chart dependencies |

**What I learned:**

Helm automatically generates the files required to start developing a chart.

**Where Steps 14–17 come from:** Teacher's custom Helm chart creation demonstration.

---

# Part 6 – Configure the Payments Microservice

## Step 18: Update Payments Chart.yaml

Make sure you are inside:

```text
~/best-commerce/payments
```

Open the file:

```bash
vim Chart.yaml
```

Find:

```yaml
appVersion: "1.16.0"
```

Change it to:

```yaml
appVersion: "1.0.0"
```

Save and exit:

1. Press `Esc`.
2. Type `:wq`.
3. Press `Enter`.

Verify:

```bash
cat Chart.yaml
```

Important fields:

```yaml
apiVersion: v2
name: payments
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.0.0"
```

**Explanation:**

- `name` is the chart name.
- `version` is the Helm chart version.
- `appVersion` is the application version.
- `type: application` identifies this as an application chart.

## Step 19: Update Payments Deployment Template

Enter the templates directory:

```bash
cd templates
```

Open the Deployment template:

```bash
vim deployment.yaml
```

Delete the default contents and replace them with:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.name }}
  labels:
    app: {{ .Values.name }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: {{ .Values.name }}
  template:
    metadata:
      labels:
        app: {{ .Values.name }}
    spec:
      containers:
        - name: {{ .Values.name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          command: ["/bin/sh", "-c"]
          args:
            - "while true; do echo '{{ .Values.appMessage }}'; sleep 10; done"
```

Save and exit:

```text
Esc
:wq
Enter
```

Verify the file:

```bash
cat deployment.yaml
```

### Understand the Deployment template

This file defines a Kubernetes Deployment using Helm variables.

| Variable | Purpose |
|---|---|
| `.Values.name` | Application name |
| `.Values.image.repository` | Docker image repository |
| `.Values.image.tag` | Docker image version |
| `.Values.image.pullPolicy` | Image pulling policy |
| `.Values.appMessage` | Message printed by the container |

The Deployment runs a BusyBox container and repeatedly prints the configured message.

## Step 20: Configure Payments values.yaml

Go back to the Payments directory:

```bash
cd ..
```

Open:

```bash
vim values.yaml
```

Delete the default contents and replace them with:

```yaml
name: payments

image:
  repository: busybox
  tag: latest
  pullPolicy: IfNotPresent

appMessage: "I am a payment service"
```

Save and exit.

Verify:

```bash
cat values.yaml
```

### Understand the configuration

| Field | Value | Purpose |
|---|---|---|
| name | payments | Application name |
| repository | busybox | Docker image |
| tag | latest | Image tag |
| pullPolicy | IfNotPresent | Pull image if it is not already available locally |
| appMessage | I am a payment service | Message printed by the container |

**What we achieved:**

We created a Payments chart with its own Deployment template and configuration values.

---

# Part 7 – Configure the Shipping Microservice

## Step 21: Enter the Shipping Directory

Go back to the project directory:

```bash
cd ~/best-commerce
```

Enter Shipping:

```bash
cd shipping
```

## Step 22: Update Shipping Chart.yaml

Open:

```bash
vim Chart.yaml
```

Change:

```yaml
appVersion: "1.16.0"
```

To:

```yaml
appVersion: "1.0.0"
```

Save and exit.

Verify:

```bash
cat Chart.yaml
```

## Step 23: Update Shipping Deployment Template

Enter:

```bash
cd templates
```

Open:

```bash
vim deployment.yaml
```

Delete the default contents and paste the same Deployment template used for Payments:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.name }}
  labels:
    app: {{ .Values.name }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: {{ .Values.name }}
  template:
    metadata:
      labels:
        app: {{ .Values.name }}
    spec:
      containers:
        - name: {{ .Values.name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          command: ["/bin/sh", "-c"]
          args:
            - "while true; do echo '{{ .Values.appMessage }}'; sleep 10; done"
```

Save and exit.

Verify:

```bash
cat deployment.yaml
```

## Step 24: Configure Shipping values.yaml

Go back:

```bash
cd ..
```

Open:

```bash
vim values.yaml
```

Delete the default contents and replace them with:

```yaml
name: shipping

image:
  repository: busybox
  tag: latest
  pullPolicy: IfNotPresent

appMessage: "I am a shipping service"
```

Save and exit.

Verify:

```bash
cat values.yaml
```

### Understand the difference

Both microservices use the same BusyBox image, but their configuration values are different.

| Configuration | Payments | Shipping |
|---|---|---|
| name | payments | shipping |
| image | busybox | busybox |
| tag | latest | latest |
| appMessage | I am a payment service | I am a shipping service |

**What we achieved:**

We created two separate Helm charts with different configurations.

This demonstrates how Helm templates can be reused with different values.

**Where Steps 18–24 come from:** Teacher's Best Commerce example and our hands-on chart customization.

---

# Part 8 – Package the Helm Charts

## Step 25: Go Back to the Main Project Directory

Run:

```bash
cd ~/best-commerce
```

Check the directories:

```bash
ls -l
```

You should see:

```text
payments/
shipping/
```

## Step 26: Package the Payments Chart

Run:

```bash
helm package payments
```

Expected output:

```text
Successfully packaged chart and saved it to:
.../best-commerce/payments-0.1.0.tgz
```

**Explanation:**

Helm packages the entire Payments chart into a compressed `.tgz` archive.

## Step 27: Package the Shipping Chart

Run:

```bash
helm package shipping
```

Expected output:

```text
Successfully packaged chart and saved it to:
.../best-commerce/shipping-0.1.0.tgz
```

## Step 28: Verify the Packages

Run:

```bash
ls -lh
```

Expected structure:

```text
best-commerce/
├── payments/
├── shipping/
├── payments-0.1.0.tgz
└── shipping-0.1.0.tgz
```

**What we achieved:**

Both Helm charts have been converted into portable `.tgz` packages.

These packages can be shared with other teams.

---

# Part 9 – Create the Helm Repository Index

## Step 29: Generate index.yaml

Make sure you are inside:

```text
~/best-commerce
```

Run:

```bash
helm repo index .
```

Expected output:

```text
```

The command creates an `index.yaml` file in the current directory.

## Step 30: Verify the Repository Files

Run:

```bash
ls -lh
```

Expected structure:

```text
best-commerce/
├── payments/
├── shipping/
├── payments-0.1.0.tgz
├── shipping-0.1.0.tgz
└── index.yaml
```

## Step 31: View index.yaml

Run:

```bash
cat index.yaml
```

The file should contain entries for both charts:

```yaml
apiVersion: v1
entries:
  payments:
    - name: payments
      version: 0.1.0
      appVersion: 1.0.0
      urls:
        - payments-0.1.0.tgz

  shipping:
    - name: shipping
      version: 0.1.0
      appVersion: 1.0.0
      urls:
        - shipping-0.1.0.tgz
```

The actual generated file also contains fields such as chart digests and creation timestamps.

### What is index.yaml?

`index.yaml` contains information about the charts available in a Helm repository.

It tells Helm:

- Which charts are available.
- Which versions are available.
- Where the chart packages are located.
- Which application version each chart uses.

**What we achieved:**

We created a local Helm chart repository structure containing two packaged charts and an index file.

**Where Steps 25–31 come from:** Teacher's chart packaging and Helm repository indexing demonstration.

---

# Part 10 – Publishing the Helm Repository

## Step 32: Understand How We Share the Charts

After creating the chart packages and `index.yaml`, we can publish the repository to a centralized location.

Examples:

- GitHub Pages
- Nexus Repository
- JFrog Artifactory

Once hosted, other team members can access the repository and install the charts.

### Add a Hosted Helm Repository

General syntax:

```bash
helm repo add REPOSITORY_NAME REPOSITORY_URL
```

Example:

```bash
helm repo add best-commerce https://example.com/helm-charts
```

This example URL is a placeholder. A real repository must be hosted and contain the index and chart packages.

### Search for Charts

```bash
helm search repo best-commerce
```

### Install a Chart

```bash
helm install payments best-commerce/payments
```

**Important:** In our practical, we created the local chart packages and repository index. We did not actually publish them to GitHub Pages, Nexus or JFrog Artifactory.

---

# Part 11 – Final Project Structure

After completing the practical, our project structure looks like this:

```text
best-commerce/
│
├── payments/
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── charts/
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── serviceaccount.yaml
│       └── ...
│
├── shipping/
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── charts/
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── serviceaccount.yaml
│       └── ...
│
├── payments-0.1.0.tgz
├── shipping-0.1.0.tgz
└── index.yaml
```

---

# Part 12 – Important Commands for Revision

| Command | Purpose |
|---|---|
| `helm version` | Check Helm version |
| `helm repo add` | Add a Helm repository |
| `helm repo update` | Update repository information |
| `helm repo list` | List configured repositories |
| `helm search repo` | Search for charts |
| `helm install` | Install a chart |
| `helm list` | List installed releases |
| `helm uninstall` | Uninstall a release |
| `helm show values` | View default chart values |
| `helm create` | Create a new chart |
| `helm package` | Package a chart |
| `helm repo index .` | Generate repository index |
| `helm upgrade` | Upgrade a release |
| `helm get values` | View values used by an installed release |
| `helm get manifest` | View generated Kubernetes manifests |

---

# Part 13 – What I Learned and Achieved

## What I Learned

1. Understood Helm and why it is used in Kubernetes.
2. Learned the difference between a Helm repository, chart and release.
3. Added and searched Helm repositories.
4. Installed and uninstalled NGINX using Helm.
5. Learned how to view default chart values.
6. Created custom Helm charts using `helm create`.
7. Understood the purpose of `Chart.yaml`, `values.yaml` and `templates/`.
8. Used Helm variables inside Kubernetes Deployment templates.
9. Configured two different microservices using the same Docker image.
10. Packaged Helm charts using `helm package`.
11. Generated a Helm repository index using `helm repo index .`.
12. Learned how Helm charts can be shared through centralized repositories.

## What I Achieved

I created two custom Helm charts for an example e-commerce application:

- Payments
- Shipping

Both charts use BusyBox but have different configurations and application messages.

I packaged both charts into `.tgz` files and generated an `index.yaml` file to create a local Helm repository structure.

## Final Conclusion

Through this practical, I understood how Helm makes Kubernetes application management easier.

Instead of maintaining separate Kubernetes configurations manually, we can use Helm charts, templates and values to create reusable application packages.

This practical helped me understand how DevOps Engineers can manage and share Kubernetes applications using Helm.
