
# Kubernetes – Startup Probes & Readiness Probes

## 1. What are Probes in Kubernetes?

Probes are health checks performed by Kubernetes to understand the condition of containers running inside Pods.

In simple words, probes help Kubernetes determine whether an application has started properly, is ready to receive traffic or is still healthy.

Kubernetes provides three main types of probes:

1. **Startup Probe** – Checks whether the application has started successfully.
2. **Readiness Probe** – Checks whether the application is ready to receive traffic.
3. **Liveness Probe** – Checks whether the application is still healthy and should continue running.

---

# PART 1: STARTUP PROBE

## 2. What is a Startup Probe?

A Startup Probe is a Kubernetes health check that determines whether an application inside a container has successfully started.

**In simple words: Startup Probe checks whether your application has finished starting.**

### Why Do We Need Startup Probes?

Imagine you have a Java Spring Boot application running inside a Kubernetes Pod.

Java applications sometimes take 2–3 minutes to start because they need to:

- Load application configurations.
- Establish database connections.
- Initialize dependencies.
- Load application components.

During this time, the application might not be ready.

Without a Startup Probe, Kubernetes could perform liveness checks before the application finishes starting and restart the container unnecessarily.

Startup Probes help prevent this.

### Real-World Example

Imagine you switch on your laptop.

Your laptop needs some time to:
- Boot the operating system.
- Load background services.
- Initialize applications.

You cannot expect everything to work immediately after pressing the power button.

Similarly, a containerized application may need time to start.

Startup Probe gives the application time to complete its startup process.

## 3. How Does Startup Probe Work?

1. Kubernetes starts the container.
2. Startup Probe begins checking the application.
3. If the application is still starting, Kubernetes continues checking.
4. Once the probe succeeds, startup checking stops.
5. Kubernetes then allows the configured readiness and liveness probes to take over.

**Important:** While a Startup Probe has not succeeded, Kubernetes does not run the configured readiness and liveness probes.

If the Startup Probe fails the configured number of consecutive times, Kubernetes restarts the container.

---

# PART 2: READINESS PROBE

## 4. What is a Readiness Probe?

A Readiness Probe is a Kubernetes health check that determines whether an application is ready to receive incoming traffic.

**In simple words: Readiness Probe checks whether your application is ready to serve users.**

### Why Do We Need Readiness Probes?

Imagine you deployed an application inside a Kubernetes Pod.

The container is running, but the application is still:

- Connecting to a database.
- Loading configurations.
- Initializing services.
- Preparing resources.

Although the container is running, the application may not be ready to handle user requests.

If Kubernetes sends traffic to that Pod immediately, users may experience errors.

Readiness Probe prevents this.

### Real-World Example

Imagine you visit a restaurant.

The restaurant is open, but the kitchen is still preparing food.

You cannot expect your order to be served immediately.

Once the kitchen is ready, the restaurant starts accepting orders.

Similarly, Readiness Probe ensures that an application receives traffic only when it is ready.

## 5. How Does Readiness Probe Work?

1. Kubernetes starts the container.
2. Readiness Probe checks the application.
3. If the application is not ready, the probe fails.
4. Kubernetes marks the Pod as NotReady.
5. The Pod is removed from the ready endpoints of its Service.
6. Once the application becomes ready, the probe succeeds.
7. The Pod becomes Ready and can receive traffic.

**Important:** A Readiness Probe failure does not restart the container.

It only prevents the Pod from receiving traffic through its Kubernetes Service.

---

# PART 3: DIFFERENCE BETWEEN STARTUP AND READINESS PROBES

## 6. Startup Probe vs Readiness Probe

| Startup Probe | Readiness Probe |
|---|---|
| Checks whether the application has started. | Checks whether the application is ready to receive traffic. |
| Used during application startup. | Used throughout the container's lifecycle after startup succeeds. |
| Protects slow-starting applications. | Prevents traffic from reaching unready applications. |
| Runs until it succeeds or reaches its failure threshold. | Continues checking periodically. |
| Failure beyond the configured threshold causes a container restart. | Failure removes the Pod from ready Service endpoints. |

### Easy Way to Remember

- **Startup Probe:** Has the application started?
- **Readiness Probe:** Can the application handle traffic?

---

# PART 4: PRACTICAL KUBERNETES YAML CONFIGURATION

## 7. Startup Probe Example

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: my-app

spec:
  replicas: 1

  selector:
    matchLabels:
      app: my-app

  template:
    metadata:
      labels:
        app: my-app

    spec:
      containers:
        - name: my-container
          image: nginx

          ports:
            - containerPort: 80

          startupProbe:
            httpGet:
              path: /
              port: 80

            initialDelaySeconds: 10
            periodSeconds: 5
            failureThreshold: 30
```

### Explanation

| Parameter | Meaning |
|---|---|
| startupProbe | Configures the Startup Probe. |
| httpGet | Performs an HTTP health check. |
| path: / | Checks the application's root endpoint. |
| port: 80 | Sends the health check to port 80. |
| initialDelaySeconds: 10 | Waits 10 seconds before the first check. |
| periodSeconds: 5 | Performs a check every 5 seconds. |
| failureThreshold: 30 | Allows 30 consecutive failures before restarting the container. |

In this example, the Startup Probe can allow approximately 150 seconds of failed checks after the initial delay before Kubernetes restarts the container.

## 8. Readiness Probe Example

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: my-app

spec:
  replicas: 1

  selector:
    matchLabels:
      app: my-app

  template:
    metadata:
      labels:
        app: my-app

    spec:
      containers:
        - name: my-container
          image: nginx

          ports:
            - containerPort: 80

          readinessProbe:
            httpGet:
              path: /
              port: 80

            initialDelaySeconds: 5
            periodSeconds: 5
            failureThreshold: 3
```

### Explanation

| Parameter | Meaning |
|---|---|
| readinessProbe | Configures the Readiness Probe. |
| httpGet | Performs an HTTP health check. |
| path: / | Checks the root endpoint. |
| port: 80 | Checks port 80. |
| initialDelaySeconds: 5 | Waits 5 seconds before the first check. |
| periodSeconds: 5 | Checks every 5 seconds. |
| failureThreshold: 3 | Marks the container unready after 3 consecutive failures. |

When the Readiness Probe fails, Kubernetes removes the Pod from the ready Service endpoints.

It does not restart the container.

**Note:** These examples use Nginx, which serves HTTP on port 80. For a real application, use an appropriate health-check endpoint provided by that application.

---

# PART 5: IMPORTANT PROBE CONFIGURATION PARAMETERS

## 9. Probe Configuration Parameters

| Parameter | Explanation |
|---|---|
| initialDelaySeconds | Time to wait before the first probe. |
| periodSeconds | How frequently the probe runs. |
| timeoutSeconds | Maximum time allowed for a probe to respond. |
| successThreshold | Number of consecutive successes required to consider the probe successful. |
| failureThreshold | Number of consecutive failures required before the probe is considered failed. |

**Important:** For Startup and Liveness Probes, successThreshold must be 1.

---

# PART 6: TYPES OF PROBE CHECKS

## 10. HTTP GET

Checks whether an application responds to an HTTP request.

Example:

```yaml
httpGet:
  path: /health
  port: 8080
```

Used when applications expose HTTP health endpoints.

## 11. TCP Socket

Checks whether a particular port is accepting TCP connections.

Example:

```yaml
tcpSocket:
  port: 8080
```

Used to check whether a service is listening on a specific port.

## 12. Exec Command

Executes a command inside the container to check application health.

Example:

```yaml
exec:
  command:
    - cat
    - /tmp/healthy
```

If the command exits with status code 0, the probe succeeds.

---

# PART 7: INTERVIEW QUESTIONS AND ANSWERS

## Q1. What are Probes in Kubernetes?

**Answer:**

Probes are health checks performed by Kubernetes to monitor containers and determine whether applications have started, are ready to receive traffic or are still healthy.

## Q2. What is a Startup Probe?

**Answer:**

A Startup Probe checks whether an application inside a container has successfully started. It is especially useful for slow-starting applications such as Java and Spring Boot applications.

## Q3. What is a Readiness Probe?

**Answer:**

A Readiness Probe checks whether an application is ready to receive incoming traffic. If the probe fails, Kubernetes removes the Pod from the ready Service endpoints until the application becomes ready again.

## Q4. Why do we need a Startup Probe?

**Answer:**

Some applications take more time to start. A Startup Probe gives them sufficient time to initialize without being restarted by liveness checks before startup is complete.

## Q5. Why do we need a Readiness Probe?

**Answer:**

A Readiness Probe prevents traffic from reaching applications that are running but not ready to handle requests. It helps avoid failed requests during application initialization or temporary unavailability.

## Q6. What happens when a Startup Probe fails?

**Answer:**

If the Startup Probe fails consecutively for the configured failureThreshold, Kubernetes restarts the container.

## Q7. What happens when a Readiness Probe fails?

**Answer:**

Kubernetes marks the Pod as NotReady and removes it from the ready Service endpoints. The container continues running and is not restarted just because the Readiness Probe failed.

## Q8. What is the difference between Startup Probe and Readiness Probe?

**Answer:**

A Startup Probe checks whether the application has completed its startup process, whereas a Readiness Probe checks whether the application is ready to receive traffic.

## Q9. Can we use Startup and Readiness Probes together?

**Answer:**

Yes. We can configure both probes in the same container. The Startup Probe checks application startup, and after it succeeds, the Readiness Probe checks whether the application is ready to receive traffic.

## Q10. Which applications benefit from Startup Probes?

**Answer:**

Applications that take a long time to initialize, such as Java, Spring Boot and other complex applications, can benefit from Startup Probes.

## Q11. Does a Readiness Probe restart a container?

**Answer:**

No. A Readiness Probe failure does not restart the container. It only marks the Pod as NotReady and removes it from ready Service endpoints.

## Q12. What are the different types of Kubernetes probe checks?

**Answer:**

Kubernetes supports three main probe mechanisms:

1. HTTP GET – Checks an HTTP endpoint.
2. TCP Socket – Checks whether a TCP port is accepting connections.
3. Exec Command – Executes a command inside the container.

---

# PART 8: SCENARIO-BASED INTERVIEW QUESTIONS AND ANSWERS

## Q1. Your Java application takes three minutes to start. Kubernetes keeps restarting the container before the application starts. How would you solve this?

**Answer:**

I would configure a Startup Probe with an appropriate periodSeconds and failureThreshold.

This gives the application enough time to complete initialization before Kubernetes starts performing the configured liveness and readiness checks.

## Q2. Your Kubernetes Pod is running, but users are receiving errors when accessing the application. What could be the reason?

**Answer:**

The container might be running, but the application may not be ready to handle incoming traffic.

I would configure a Readiness Probe to check application health and ensure that traffic reaches the Pod only when it is ready.

## Q3. Your application becomes temporarily unavailable because it is connecting to a database. What would you use?

**Answer:**

I would use a Readiness Probe to temporarily mark the Pod as NotReady while the application cannot serve requests.

Once the database connection is restored and the health check succeeds, Kubernetes can add the Pod back to the ready Service endpoints.

## Q4. Your Startup Probe fails continuously. What happens?

**Answer:**

If the Startup Probe reaches its configured failureThreshold, Kubernetes restarts the container. I would check application logs and startup configurations to identify the reason for failure.

## Q5. You have three replicas of an application. One Pod becomes unready. Will the other two Pods continue receiving traffic?

**Answer:**

Yes. If the other two Pods remain ready and are correctly configured behind the same Kubernetes Service, they can continue receiving traffic while the unready Pod is excluded from ready Service endpoints.

---

# PART 9: QUICK REVISION

| Concept | Remember |
|---|---|
| Probe | Health check for a container. |
| Startup Probe | Checks whether the application has started. |
| Readiness Probe | Checks whether the application can receive traffic. |
| Liveness Probe | Checks whether the application should continue running. |
| HTTP GET | Checks an HTTP endpoint. |
| TCP Socket | Checks a TCP port. |
| Exec | Executes a command inside the container. |
| failureThreshold | Consecutive failures allowed before the probe is considered failed. |
| periodSeconds | Frequency of health checks. |
| timeoutSeconds | Maximum time allowed for a probe response. |

---

# FINAL TAKEAWAY

Remember these three questions:

1. **Startup Probe:** Has my application started?
2. **Readiness Probe:** Is my application ready to receive traffic?
3. **Liveness Probe:** Is my application still healthy, or should Kubernetes restart it?

### Easy Flow to Remember

Container Starts → Startup Probe Succeeds → Readiness Probe Checks Application → Pod Becomes Ready → Receives Traffic.

### Important Interview Tip

Never confuse Readiness and Liveness Probes.

- Readiness Probe failure stops traffic from reaching the Pod.
- Liveness Probe failure causes Kubernetes to restart the container after the configured failure threshold.

## Official Documentation

Kubernetes – Configure Liveness, Readiness and Startup Probes:

https://kubernetes.io/docs/concepts/configuration/liveness-readiness-startup-probes/
