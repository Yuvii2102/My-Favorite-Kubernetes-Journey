# Kubernetes Practical: Deploying an Application Using Deployments

## 1. Practical Overview

In this practical, I learned how to deploy an NGINX application using Kubernetes Deployments.

I used AWS EC2 Ubuntu as my terminal and Minikube to create a local Kubernetes cluster.

The main objective was to understand how Kubernetes manages application Pods through Deployments and ReplicaSets.

### What I worked on

- Installing Docker, kubectl and Minikube.
- Starting a Kubernetes cluster using Minikube.
- Creating a Deployment YAML manifest.
- Deploying an NGINX application.
- Inspecting Deployments, ReplicaSets and Pods.
- Testing automatic Pod replacement.
- Scaling the application from one replica to three replicas.
- Inspecting Deployment details and events.

## 2. Tools and Technologies Used

| Tool | Purpose |
|---|---|
| AWS EC2 Ubuntu | Environment used to run commands |
| Docker | Container runtime used by Minikube |
| kubectl | Command-line tool to communicate with Kubernetes |
| Minikube | Creates a local Kubernetes cluster |
| NGINX | Application deployed inside Kubernetes |
| YAML | Defines the Kubernetes Deployment configuration |

## 3. Prerequisites

Before starting, make sure you have:

- An AWS EC2 Ubuntu instance.
- SSH access to the instance.
- Internet connectivity.
- Sufficient memory and disk space to run Docker and Minikube.

I used an EC2 instance with Ubuntu 26.04 and the AMD64 architecture.

Connect to your EC2 instance using SSH or MobaXterm.

Update the package information:

```bash
sudo apt update
```

## 4. Install Docker

Docker is required because we are using Minikube with the Docker driver.

First, check whether Docker is already installed:

```bash
docker --version
```

If Docker is installed, you will see its version.

If the command is not found, install Docker using Ubuntu's package:

```bash
sudo apt update
sudo apt install -y docker.io
```

Start the Docker service:

```bash
sudo systemctl start docker
```

Enable Docker to start automatically when the system boots:

```bash
sudo systemctl enable docker
```

Check Docker's status:

```bash
sudo systemctl status docker
```

Press `q` to exit the status screen.

Check the Docker version:

```bash
docker --version
```

### Configure Docker permissions

By default, Docker commands may require root privileges.

To allow the current user to run Docker commands:

```bash
sudo usermod -aG docker $USER
```

Apply the group change in the current terminal:

```bash
newgrp docker
```

Verify Docker access:

```bash
docker ps
```

If the command works without `sudo`, Docker permissions are configured.

**Important:** Membership in the Docker group grants root-level privileges. Only add trusted users to this group.

Official reference:

https://docs.docker.com/engine/install/linux-postinstall/

For the official Docker Engine installation method, refer to:

https://docs.docker.com/engine/install/ubuntu/

## 5. Install kubectl

`kubectl` is the command-line tool used to communicate with the Kubernetes cluster.

First, check whether kubectl is already installed:

```bash
kubectl version --client
```

If the command is not found, install it.

### Installation commands

Download the latest stable kubectl binary for Linux AMD64:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Install it:

```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Verify the installation:

```bash
kubectl version --client
```

You should now see the kubectl client version.

Official reference:

https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

## 6. Install Minikube

Minikube helps us create and run a Kubernetes cluster on a local machine or virtual machine.

First, check whether Minikube is installed:

```bash
minikube version
```

If it is not installed, download the latest Linux AMD64 binary:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```

Install Minikube:

```bash
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Verify the installation:

```bash
minikube version
```

Official reference:

https://minikube.sigs.k8s.io/docs/start/

## 7. Start the Kubernetes Cluster

Before starting Minikube, make sure Docker is running:

```bash
sudo systemctl status docker
```

If Docker is stopped:

```bash
sudo systemctl start docker
```

Start Minikube using the Docker driver:

```bash
minikube start --driver=docker
```

This command starts a Kubernetes cluster using Docker as the driver.

Check the cluster status:

```bash
minikube status
```

Check the Kubernetes nodes:

```bash
kubectl get nodes
```

To get additional information about the node:

```bash
kubectl get nodes -o wide
```

### Expected result

The node should show:

```text
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   ...   ...
```

The `Ready` status means that the node is ready to run workloads.

### If Minikube is not running

Check the status:

```bash
minikube status
```

Start the cluster again:

```bash
minikube start --driver=docker
```

If Docker is stopped:

```bash
sudo systemctl start docker
```

If Docker permission is denied:

```bash
newgrp docker
```

If the Kubernetes context is incorrect:

```bash
kubectl config use-context minikube
```

Check Minikube logs when startup fails:

```bash
minikube logs
```

Verify everything again:

```bash
minikube status
kubectl get nodes
```

**Note:** If the EC2 instance is restarted or Minikube is stopped, check the status and start the cluster again when necessary.

## 8. Create a Project Directory

Create a folder to keep the Kubernetes YAML files:

```bash
mkdir kubernetes-practice
```

Move into the folder:

```bash
cd kubernetes-practice
```

Check the current directory:

```bash
pwd
```

## 9. Create the Deployment YAML File

Now we will create a Deployment for our NGINX application.

Create a file:

```bash
vim deployment.yaml
```

Press `i` to enter insert mode.

Paste the following configuration:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

spec:
  replicas: 1

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

Save the file:

1. Press `Esc`.
2. Type `:wq`.
3. Press Enter.

Verify the file:

```bash
cat deployment.yaml
```

### Understanding the YAML configuration

| Field | Purpose |
|---|---|
| `apiVersion: apps/v1` | API version used for Deployments |
| `kind: Deployment` | Defines the Kubernetes resource type |
| `metadata.name` | Name of the Deployment |
| `replicas: 1` | Specifies that we want one Pod |
| `selector.matchLabels` | Identifies the Pods managed by the Deployment |
| `template` | Defines the Pod configuration |
| `labels` | Identifies the Pods using the label `app: nginx` |
| `containers` | Defines the containers inside the Pod |
| `image` | Specifies the NGINX container image |
| `containerPort: 80` | Documents the port used by the container |

**Important:** The labels under `template.metadata.labels` must match the labels under `selector.matchLabels`.

The `containerPort` field does not expose the application publicly. A Kubernetes Service is needed for network access through a Service.

## 10. Create the Deployment

Apply the YAML configuration:

```bash
kubectl apply -f deployment.yaml
```

Expected output:

```text
deployment.apps/nginx-deployment created
```

### Understanding the command

- `kubectl` communicates with the Kubernetes cluster.
- `apply` creates or updates resources using a configuration file.
- `-f` specifies the file.
- `deployment.yaml` is our Deployment manifest.

The Deployment is now created.

## 11. Verify the Deployment

Check the Deployment:

```bash
kubectl get deployments
```

Expected output:

```text
NAME               READY   UP-TO-DATE   AVAILABLE
nginx-deployment   1/1     1            1
```

### Understanding the output

- `READY`: Number of ready replicas compared with the desired number.
- `UP-TO-DATE`: Number of replicas updated to match the latest Pod template.
- `AVAILABLE`: Number of replicas available to serve the application.

Our Deployment has one ready replica.

## 12. Verify the ReplicaSet

Run:

```bash
kubectl get rs
```

Expected output will look similar to:

```text
NAME                          DESIRED   CURRENT   READY
nginx-deployment-xxxxx        1         1         1
```

Here:

- `DESIRED`: Number of Pods we want.
- `CURRENT`: Number of Pods currently created.
- `READY`: Number of Pods that are ready.

Notice that Kubernetes automatically created a ReplicaSet.

We did not create the ReplicaSet manually.

## 13. Verify the Pods

Run:

```bash
kubectl get pods
```

Expected output:

```text
NAME                                READY   STATUS    RESTARTS
nginx-deployment-xxxxx              1/1     Running   0
```

This confirms that our NGINX Pod is running.

To get additional details:

```bash
kubectl get pods -o wide
```

This displays information such as the Pod IP address and the node where the Pod is running.

### What happened behind the scenes?

When we applied the Deployment YAML:

1. Kubernetes created the Deployment.
2. The Deployment created a ReplicaSet.
3. The ReplicaSet created one Pod.
4. The Pod started the NGINX container.

The resource relationship is:

**Deployment → ReplicaSet → Pod → Container**

## 14. Test Automatic Pod Replacement (Self-Healing)

Now we will test what happens when a Pod managed by a Deployment gets deleted.

First, list the Pods:

```bash
kubectl get pods
```

Copy the actual Pod name.

Delete the Pod:

```bash
kubectl delete pod <pod-name>
```

Replace `<pod-name>` with the actual Pod name.

For example:

```bash
kubectl delete pod nginx-deployment-xxxxx
```

Expected output:

```text
pod "nginx-deployment-xxxxx" deleted
```

Now check the Pods again:

```bash
kubectl get pods
```

You should see a new Pod being created.

To watch the changes live:

```bash
kubectl get pods -w
```

Press `Ctrl + C` to stop watching.

### What happened?

1. We deleted the original Pod.
2. The ReplicaSet detected that the number of Pods was below the desired count.
3. It created a replacement Pod.
4. The new Pod reached the Running state.

The new Pod has a different name because Kubernetes created a new Pod instead of recovering the deleted one.

This behavior is called **Self-Healing**.

## 15. Scale the Deployment from One Replica to Three Replicas

Now we will increase the number of NGINX Pods from one to three.

Open the Deployment file:

```bash
vim deployment.yaml
```

Find:

```yaml
spec:
  replicas: 1
```

Change it to:

```yaml
spec:
  replicas: 3
```

Save the file:

1. Press `Esc`.
2. Type `:wq`.
3. Press Enter.

Apply the updated configuration:

```bash
kubectl apply -f deployment.yaml
```

Expected output:

```text
deployment.apps/nginx-deployment configured
```

### Verify the Deployment

```bash
kubectl get deployments
```

Expected result:

```text
NAME               READY   UP-TO-DATE   AVAILABLE
nginx-deployment   3/3     3            3
```

### Verify the ReplicaSet

```bash
kubectl get rs
```

Expected result:

```text
NAME                          DESIRED   CURRENT   READY
nginx-deployment-xxxxx        3         3         3
```

### Verify the Pods

```bash
kubectl get pods
```

Expected result:

```text
NAME                                READY   STATUS
nginx-deployment-xxxxx              1/1     Running
nginx-deployment-xxxxx              1/1     Running
nginx-deployment-xxxxx              1/1     Running
```

The actual Pod names will be different.

### What happened?

1. We changed the desired replica count from one to three.
2. Kubernetes detected the configuration change.
3. The ReplicaSet created two additional Pods.
4. All three Pods reached the Running state.

This is called **Scaling**.

In this practical, we performed manual scaling by changing the replica count in the YAML file.

## 16. Inspect All Resources

Run:

```bash
kubectl get all
```

This displays several common resources in the current namespace.

Our output included:

- NGINX Pods
- Kubernetes Service
- NGINX Deployment
- NGINX ReplicaSet

**Important:** `kubectl get all` does not list every resource type in Kubernetes.

To inspect resources across all namespaces:

```bash
kubectl get all -A
```

## 17. Describe the Deployment

Run:

```bash
kubectl describe deployment nginx-deployment
```

This command displays detailed information about our Deployment.

### Important fields to understand

**Replicas:**

```text
3 desired | 3 updated | 3 total | 3 available | 0 unavailable
```

This confirms that all three requested replicas are available.

**StrategyType:**

```text
StrategyType: RollingUpdate
```

This shows that the Deployment uses the RollingUpdate strategy.

**RollingUpdateStrategy:**

```text
25% max unavailable, 25% max surge
```

- `maxUnavailable` controls how many Pods can be unavailable during an update.
- `maxSurge` controls how many additional Pods can temporarily be created during an update.

**Conditions:**

```text
Progressing    True    NewReplicaSetAvailable
Available      True    MinimumReplicasAvailable
```

These conditions indicate that the Deployment has progressed successfully and has the minimum available replicas.

**Events:**

Our output included:

```text
Normal  ScalingReplicaSet  deployment-controller
Scaled up replica set nginx-deployment-xxxxx from 0 to 1

Normal  ScalingReplicaSet  deployment-controller
Scaled up replica set nginx-deployment-xxxxx from 1 to 3
```

These events show how Kubernetes initially created one replica and then increased the count to three.

## 18. Troubleshooting

Here are some common issues and commands to help troubleshoot them.

### Issue 1: kubectl command not found

Check:

```bash
kubectl version --client
```

If kubectl is not installed, follow the installation instructions in Section 5.

Official installation guide:

https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

### Issue 2: Minikube command not found

Check:

```bash
minikube version
```

If it is not installed, follow Section 6.

Official installation guide:

https://minikube.sigs.k8s.io/docs/start/

### Issue 3: Docker permission denied

Run:

```bash
sudo usermod -aG docker $USER
newgrp docker
docker ps
```

### Issue 4: Docker service is not running

Run:

```bash
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
```

### Issue 5: Minikube is not running

Check:

```bash
minikube status
```

Start it:

```bash
minikube start --driver=docker
```

### Issue 6: Pod is stuck in Pending or ImagePullBackOff

Inspect the Pod:

```bash
kubectl get pods
```

Get detailed information:

```bash
kubectl describe pod <pod-name>
```

Check the events and messages to identify whether the issue is related to scheduling, resources, image availability or network access.

### Issue 7: Deployment is not creating Pods

Check the Deployment:

```bash
kubectl describe deployment nginx-deployment
```

Check the ReplicaSet:

```bash
kubectl get rs
```

Check the Pods:

```bash
kubectl get pods
```

The describe output and events can help identify the problem.

## 19. Complete Command Reference

| Command | Purpose |
|---|---|
| `docker --version` | Check Docker installation |
| `kubectl version --client` | Check kubectl installation |
| `minikube version` | Check Minikube installation |
| `minikube start --driver=docker` | Start Kubernetes cluster |
| `minikube status` | Check cluster status |
| `kubectl get nodes` | List Kubernetes nodes |
| `vim deployment.yaml` | Create or edit Deployment YAML |
| `cat deployment.yaml` | Display YAML configuration |
| `kubectl apply -f deployment.yaml` | Create or update Deployment |
| `kubectl get deployments` | List Deployments |
| `kubectl get rs` | List ReplicaSets |
| `kubectl get pods` | List Pods |
| `kubectl get pods -o wide` | Display additional Pod information |
| `kubectl get pods -w` | Watch Pod changes |
| `kubectl delete pod <pod-name>` | Delete a Pod |
| `kubectl get all` | List several common resources |
| `kubectl get all -A` | List several common resources across namespaces |
| `kubectl describe deployment nginx-deployment` | Inspect Deployment details |
| `minikube logs` | Check Minikube logs |

## 20. Cleanup (Optional)

After completing the practical, we can remove the Deployment and its managed resources.

Run:

```bash
kubectl delete -f deployment.yaml
```

Expected output:

```text
deployment.apps "nginx-deployment" deleted
```

Verify the cleanup:

```bash
kubectl get deployments
kubectl get pods
kubectl get rs
```

The NGINX Deployment, its ReplicaSet and its managed Pods should no longer be present.

The default Kubernetes Service named `kubernetes` will remain.

**Note:** This cleanup is optional. If you want to continue practising with the three NGINX Pods, do not run the delete command.

## 21. Official Documentation and References

These are the official references used for the installation and Kubernetes commands in this practical.

### 1. Docker Engine Installation

Link: https://docs.docker.com/engine/install/ubuntu/

Practical use:
- Installing Docker Engine on Ubuntu.
- Setting up Docker to run containers.
- Preparing Docker for the Minikube Docker driver.

### 2. Docker Post-Installation Steps

Link: https://docs.docker.com/engine/install/linux-postinstall/

Practical use:
- Configuring Docker to run without `sudo`.
- Adding the current user to the Docker group.
- Understanding Docker group permissions.

### 3. kubectl Installation on Linux

Link: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

Practical use:
- Downloading the Linux AMD64 kubectl binary.
- Installing kubectl in `/usr/local/bin`.
- Verifying the client installation.

### 4. Minikube Installation

Link: https://minikube.sigs.k8s.io/docs/start/

Practical use:
- Installing Minikube.
- Starting a local Kubernetes cluster.
- Checking cluster status.

### 5. Minikube Docker Driver

Link: https://minikube.sigs.k8s.io/docs/drivers/docker/

Practical use:
- Understanding how Minikube uses Docker as its driver.
- Checking the requirements for running Minikube with Docker.

### 6. Kubernetes Deployments

Link: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

Practical use:
- Understanding Deployment YAML configuration.
- Learning about replicas, selectors and Pod templates.
- Understanding how Deployments manage ReplicaSets and Pods.

### 7. kubectl Quick Reference

Link: https://kubernetes.io/docs/reference/kubectl/quick-reference/

Practical use:
- Learning common kubectl commands.
- Applying YAML configurations.
- Listing and inspecting Kubernetes resources.

### 8. Update a Deployment Without Downtime

Link: https://kubernetes.io/docs/tasks/run-application/update-deployment-rolling/

Practical use:
- Understanding Rolling Updates.
- Monitoring rollout progress.
- Learning how Kubernetes gradually updates application Pods.

### 9. Horizontal Manual Scaling for a Deployment

Link: https://kubernetes.io/docs/tasks/run-application/scale-deployment/

Practical use:
- Understanding how to increase or decrease replicas.
- Learning different ways to scale a Deployment.

## 22. What I Achieved

In this practical, I successfully deployed an NGINX application using Kubernetes Deployments on an AWS EC2 Ubuntu instance.

I installed and configured the required tools, created a Kubernetes Deployment YAML file and deployed the application using kubectl.

I also verified how Kubernetes automatically creates a ReplicaSet and Pods.

After that, I deleted a running Pod and observed Kubernetes automatically creating a replacement Pod.

Finally, I changed the replica count from one to three and verified that all three Pods were running successfully.

### Final Results

| Task | Result |
|---|---|
| Docker installation and configuration | Completed |
| kubectl installation | Completed |
| Minikube installation | Completed |
| Kubernetes cluster setup | Completed |
| Deployment YAML creation | Completed |
| NGINX Deployment | Successful |
| ReplicaSet verification | Successful |
| Automatic Pod replacement | Successful |
| Scaling from 1 to 3 replicas | Successful |
| Deployment inspection | Successful |

## 23. Final Learning

Through this practical, I understood how Kubernetes Deployments help manage applications.

I learned that we do not need to create and manage ReplicaSets and Pods manually. We can define the desired application configuration in a Deployment YAML file and let Kubernetes manage the resources.

I also experienced how ReplicaSets maintain the desired number of Pods and how Kubernetes replaces deleted Pods automatically.

The most important thing I learned is how Kubernetes maintains the desired state of an application through Deployments, ReplicaSets and Pods.

**Deployment → ReplicaSet → Pods → Containers**

This practical helped me understand how applications are deployed, managed, recovered and scaled in Kubernetes.
