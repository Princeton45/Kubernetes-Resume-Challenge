# The Kubernetes Resume Challenge

## Project Overview
This challenge is all about showcasing my practical Kubernetes skills by deploying a real-world scenario: an e-commerce website. It goes beyond basic deployments, pushing me to demonstrate expertise in containerization, dynamic scaling, high availability, and efficient application management using Kubernetes. 

## Key Features

*   **Containerization:**
    *   The web application is containerized using Docker, ensuring consistency across environments.
    *   The official MariaDB image is utilized for the database.
*   **Kubernetes Deployment:**
    *   Deployed on a managed Kubernetes cluster (EKS).
    *   Utilizes Kubernetes Deployments for managing the web application pods.
*   **Scaling:**
    *   **Manual Scaling:** Implemented to handle anticipated traffic increases during marketing campaigns.
    *   **Horizontal Pod Autoscaler (HPA):** Configured for automatic scaling based on CPU utilization (target 50%, min 2 pods, max 10 pods). This ensures the application remains performant under varying and unpredictable traffic loads.
*   **High Availability:**
    *   **Liveness and Readiness Probes:** Integrated to ensure application health and proper traffic routing. Liveness probes automatically restart unresponsive pods, while readiness probes ensure traffic is only directed to ready pods.
    *   **Rolling Updates:** Enables zero-downtime deployments when updating the application to new versions.
    *   **Rollbacks:** Provides a mechanism to quickly revert to a previous, stable deployment in case of issues with a new release.
*   **Configuration Management:**
    *   **ConfigMaps:** Used to manage non-sensitive configuration data, such as feature toggles (e.g., enabling a "dark mode" for the website).
    *   **Secrets:** Securely store sensitive information like database credentials, preventing them from being hardcoded in the application or configuration files.
*   **Service Exposure:**
    *   A Kubernetes Service of type `LoadBalancer` exposes the application externally, providing a stable endpoint for users to access the e-commerce website.
*   **Automation:**
    *   A CI/CD pipeline is implemented using GitHub Actions. This automates the process of building the Docker image, pushing it to Docker Hub, and updating the Kubernetes deployment whenever changes are pushed to the main branch.
*   **Helm Packaging:**
    *   The entire application, including all Kubernetes resources, is packaged as a Helm chart. This simplifies deployment, enables easy versioning and rollback capabilities, and promotes reusability across different environments.
*   **Persistent Storage:**
    *   PersistentVolumeClaims (PVCs) are used to ensure data durability for the MariaDB database. This ensures that data persists across pod restarts and redeployments.

For more information on the original challenge see: https://cloudresumechallenge.dev/docs/extensions/kubernetes-challenge/


## Interaction Flow with Helm

![Kubernetes Diagram](https://github.com/Princeton45/Kubernetes-Resume-Challenge/blob/main/images/Kubernetes_diagram.png)

![Helm Diagram](https://github.com/Princeton45/Kubernetes-Resume-Challenge/blob/main/images/Helm.jpg)

### Infrastructure Setup
1. **Helm Chart Structure**
   - Create main chart for e-commerce application
   - Define subcharts for web and database components
   - Configure values.yaml for environment-specific settings

### Request Flow
1. **User Entry**
   - User accesses website URL
   - DNS resolves to cloud load balancer

2. **Traffic Handling**
   - Load Balancer (defined in Helm values) routes traffic
   - Kubernetes Service distributes to web application pods

3. **Application Processing**
   - Web pods handle requests (scaled via HPA)
   - Readiness/Liveness probes monitor health
   - ConfigMaps/Secrets provide configuration (managed via Helm)

4. **Data Layer**
   - MariaDB pod handles database operations
   - PVC ensures data persistence
   - Database credentials managed via Helm-templated secrets

### Deployment Process
1. **Development**
   - Code changes pushed to repository
   - GitHub Actions triggered

2. **CI/CD Pipeline**
   - Builds Docker image
   - Updates Helm chart version
   - Deploys using Helm upgrade

3. **Scaling & Updates**
   - HPA manages pod scaling
   - Rolling updates handled via Helm
   - Rollbacks possible using Helm rollback

## Step-by-Step Implementation

### Step 1: Certification

In January 2025, I passed the CKAD Certification.

![Cert](https://github.com/Princeton45/Kubernetes-Resume-Challenge/blob/main/images/Cert.png)

### Step 2: Containerizing the E-Commerce Website and Database

**A. Web Application Containerization**

I created a `Dockerfile` for the e-commerce application, using `php:7.4-apache` as the base image and ensuring the `mysqli` extension was installed. The application codeis copied to `/var/www/html/`. 

Later, we will  setup a `configmap` for the database connection string and mount it to the web application deployment once the EKS Cluster is setup.

![web](https://github.com/Princeton45/Kubernetes-Resume-Challenge/blob/main/images/step2_aa.png)

After creating the `Dockerfile`, I built the image with `docker build -t prince450/ecom-web:v2 .` and pushed it to Docker Hub using `docker push prince450/ecom-web:v2`.

![docker_push](https://github.com/Princeton45/Kubernetes-Resume-Challenge/blob/main/images/docker_push.png)

**A. Database Containerization**

We will use the official MariaDB image in the database Dockerfile.

![docker_db](https://github.com/Princeton45/Kubernetes-Resume-Challenge/blob/main/images/db_dockerfile.png)

![db_push](https://github.com/Princeton45/Kubernetes-Resume-Challenge/blob/main/images/db_push.png)


The official MariaDB (and MySQL) Docker images have a special mechanism where any .sql or .sh files placed in the `/docker-entrypoint-initdb.d/` directory will be automatically executed when the container starts up for the first time.

### Step 3: Setting Up Kubernetes on a Public Cloud Provider

I chose AWS and created a Kubernetes cluster using their managed Kubernetes service (EKS). 

I first needed to install and configure the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).

Then I needed to install [eksctl](https://eksctl.io/installation/) so that we can run commands from the CLI to EKS

Next, I configured [kubectl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html) to interact with the new cluster. 

Lastly, I installed the  [AWS CSI Storage Add-on](https://docs.aws.amazon.com/eks/latest/userguide/managing-ebs-csi.html) because once we get to the "Implement Persistent Storage" part of this challenge, we will need the EBS CSI driver so that we can create Persistent EBS volumes for the MariaDB Database.

I created the EKS cluster using the command below.

```bash
eksctl create cluster --name eks-cluster --region us-east-1 --zones=us-east-1a,us-east-1b --nodegroup-name node-group --node-type t2.small --nodes 2 --nodes-min 2 --nodes-max 5 --managed
```
![eks](https://github.com/Princeton45/Kubernetes-Resume-Challenge/blob/main/images/eks_cluster.png)

### Step 4: Deploying the Website to Kubernetes

First, I am creating the database pod using the `ecom-db:v1` image we created earlier.
`mysql-pod.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: ecom-web
    tier: backend
  name: ecom-db
spec:
  containers:
  - image: briangaber/ecom-db:v1
    name: ecom-db
    ports:
    - containerPort: 3306
    env:
    - name: MYSQL_USER
      value: "ecomuser"
    - name: MYSQL_PASSWORD
      value: "ecompassword"
```

I then created a `website-deployment.yaml` file to define the deployment of my web application, referencing the Docker image I pushed earlier. This deployment managed the pods running my e-commerce application.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: website-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        app: ecom-web
        tier: frontend
    spec:
      containers:
      - image: prince450/ecom-web:v2
        name: ecom-web
        ports:
        - containerPort: 80
        env:
        - name: DB_HOST
          value: "mariadb-service"
        - name: DB_USER
          value: "ecomuser"
        - name: DB_PASSWORD
          value: "ecompassword"
        - name: DB_NAME
          value: "ecomdb"
```

### Step 5: Exposing the Website

To make my website accessible, I defined a `website-service.yaml` file to create a LoadBalancer service. This exposed my deployment to the internet.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: website-service
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    tier: frontend
  type: LoadBalancer
```

Now I am able to access the web application using the auto-generated AWS Elastic Load Balancer DNS name for the application.

![app](https://github.com/Princeton45/Kubernetes-Resume-Challenge/blob/main/images/application.png)

### Step 6: Scaling the Application

Anticipating increased traffic, I scaled the application using `kubectl scale deployment/website-deployment --replicas=6`. This showcased Kubernetes' ability to handle varying loads.

![manual_scale](https://github.com/Princeton45/Kubernetes-Resume-Challenge/blob/main/images/manual_scale.png)

### Step 7: Performing a Rolling Update

I updated the application to include a new promotional banner and built a new Docker image (`prince450/ecom-web:v3`). I then performed a rolling update by modifying the `website-deployment.yaml` file.

![rollout_status](https://github.com/Princeton45/Kubernetes-Resume-Challenge/blob/main/images/rollout_status.png)

![new_banner](https://github.com/Princeton45/Kubernetes-Resume-Challenge/blob/main/images/new_banner.png)

### Step 8: Rolling Back a Deployment

To simulate a rollback, I used `kubectl rollout undo deployment/website-deployment` after identifying an issue with the new banner. This demonstrated how to revert to a previous deployment state quickly.

![rollback](https://github.com/Princeton45/Kubernetes-Resume-Challenge/blob/main/images/rollback.png)
### Step 9: Autoscaling the Application

I implemented a Horizontal Pod Autoscaler (HPA) to automatically scale the application based on CPU usage, ensuring performance under unpredictable traffic.

I created a Horizontal Pod Autoscaler targeting 50% CPU utilization, with a minimum of 2 and a maximum of 10 pods.
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  namespace: default
  name: ecom-hpa
  labels:
    app: ecom-web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: website-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

After applying that defintion to the cluster, I was now getting a `cpu: <unknown>/50%` for the target.

![hpa_error](https://github.com/Princeton45/Kubernetes-Resume-Challenge/blob/main/images/hpa_error.png)

Upon further research, I needed to edit the `website-deployment` and add a resource request for the CPU to fix the issue.

```yaml
 spec:
      containers:
      - image: prince450/ecom-web:v3
        name: ecom-web
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "300m"
```

Now we can see that it is fixed.

![hpa_fixed](https://github.com/Princeton45/Kubernetes-Resume-Challenge/blob/main/images/fixed_hpa.png)

### Load Testing

We will now simulate the website-deployment pods being scaled by using a tool called  [kubectl load generator](https://docs.aws.amazon.com/eks/latest/userguide/horizontal-pod-autoscaler.html) that runs simulated requests on kubernetes endpoints with a single command
```bash
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://website-service; done"
```
*   `-i --tty`: Allocates an interactive terminal for the command to run.
*   `load-generator`: Name of the pod.
*   `--rm`: Removes the pod after it terminates.
*   `--image=busybox`: Specifies the Docker image to use for the pod.
*   `--restart=Never`: Indicates that the pod should not be restarted automatically if it fails.
*   `/bin/sh -c "while sleep 0.01; do wget -q -O- http://website-service; done"`: The command to run inside the pod, which continuously sends HTTP requests to the specified endpoint (http://website-service) with a delay of 0.01 seconds between requests.

As you can see now in the image below, the cpu % is slowly rising

![cpu_rising](https://github.com/Princeton45/Kubernetes-Resume-Challenge/blob/main/images/cpu_rising.png)

### Step 10: Implementing Liveness and Readiness Probes

I added liveness and readiness probes to the `website-deployment.yaml` and `mariadb_deployment.yaml` file to ensure the application's health and readiness to receive traffic.

`website-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: website-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        app: ecom-web
        tier: frontend
    spec:
      containers:
      - image: prince450/ecom-web:v3
        name: ecom-web
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /index.php
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /index.php
            port: 80
        resources:
          requests:
            cpu: "300m"
        env:
        - name: DB_HOST
          value: "mariadb-service"
        - name: DB_USER
          value: "ecomuser"
        - name: DB_PASSWORD
          value: "ecompassword"
        - name: DB_NAME
          value: "ecomdb"
```

`mariadb_deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: mysql
    labels:
      app: ecom-web
      tier: backend
spec:
  replicas: 1
  selector:
    matchLabels:
     app: ecom-web
     tier: backend
  template:
    metadata:
      labels:
        app: ecom-web
        tier: backend
    spec:
      containers:
      - image: prince450/ecom-db:v1
        name: ecom-db
        ports:
        - containerPort: 3306
        readinessProbe:
          tcpSocket:
            port: 3306
          initialDelaySeconds: 5
          periodSeconds: 5
        livenessProbe:
          tcpSocket:
            port: 3306
          initialDelaySeconds: 10
          periodSeconds: 5
        env:
        - name: MARIADB_ROOT_PASSWORD
          value: "rootpassword"
        - name: MARIADB_DATABASE
          value: "ecomdb"
        - name: MARIADB_USER
          value: "ecomuser"
        - name: MARIADB_PASSWORD
          value: "ecompassword"
        volumeMounts:
        - name: mariadb-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mariadb-persistent-storage
        persistentVolumeClaim:
          claimName: mariadb-pvc
```

### Step 11: Utilizing ConfigMaps and Secrets

I used ConfigMaps for non-sensitive data like database name, host, user and Secrets for sensitive data like database password, demonstrating best practices for configuration and secret management.

`configmap.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: website-configmap
data:
  DB_NAME: "ecomdb"
  DB_HOST: "mariadb-service"
  DB_USER: "ecomuser"
```

`configsecret.yaml`
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: configsecret
  creationTimestamp: null
data:
  DB_PASSWORD: ZWNvbXBhc3N3b3Jk
```

The final `website-deployment.yaml` file referencing the config map and secret

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: website-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        app: ecom-web
        tier: frontend
    spec:
      containers:
      - image: prince450/ecom-web:v3
        name: ecom-web
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /index.php
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /index.php
            port: 80
        resources:
          requests:
            cpu: "300m"
        envFrom:
        - configMapRef:
            name: website-configmap
        - secretRef:
            name: configsecret
```

## Extra Credit

### Packaging Everything in Helm

I packaged my application using Helm, creating a chart that defined all necessary Kubernetes resources. This simplified deployment and management. 

Below is the definition for the `chart.yaml`, `website-deployment.yaml` & the `values.yaml `

`chart.yaml`
```yaml
apiVersion: v2
name: ecommerce
description: An e-commerce application Helm chart for kubernetes
type: application
version: 0.1.0
appVersion: "1.16.0"
```

`values.yaml`
```yaml
replicaCount: 2

image:
  repository: prince450/ecom-web
  tag: "v3"
  pullPolicy: IfNotPresent

service:
  type: LoadBalancer
  port: 80

configMap:
  name: website-configmap

secret:
  name: configsecret

resources:
  requests:
    cpu: 300m
```

`website-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: website-deployment
  labels:
    app: ecom-web
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        app: ecom-web
        tier: frontend
    spec:
      containers:
      - image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        name: ecom-web
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /index.php
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /index.php
            port: 80
        resources:
          requests:
            cpu: {{ .Values.resources.requests.cpu }}
        envFrom:
        - configMapRef:
            name: {{ .Values.configMap.name }}
        - secretRef:
            name: {{ .Values.secret.name }}
```

I then packaged the chart into a `.tgz` file which will be stored in AWS S3 for the CI/CD portion of this project.

**Picture Suggestion:** A screenshot of your Helm chart directory structure or your terminal showing `helm` commands.

### Implementing Persistent Storage

I implemented persistent storage for the MariaDB database using a storage class with the provisioner set to `ebs.csi.aws.com` so that it provisions an `EBS` volume instead of a local volume on the node. 

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer
parameters:
  type: gp3
  encrypted: "true"
```
Then I created a PersistentVolumeClaim (PVC) that calls that storage class to create an EBS volume whenever a database pod requests the PVC.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mariadb-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ebs-sc
  resources:
    requests:
      storage: 5Gi
```

Below is a picture of an EBS volume created from the database pod requesting the PVC.

![ebs](https://github.com/Princeton45/Kubernetes-Resume-Challenge/blob/main/images/ebs.png)

### Implementing a Basic CI/CD Pipeline

I set up a basic CI/CD pipeline using GitHub Actions to automate the build and deployment process, showcasing an efficient development workflow.

**Picture Suggestion:** A screenshot of your GitHub Actions workflow or a diagram illustrating your CI/CD pipeline.

## Conclusion

This Kubernetes Resume Challenge was a comprehensive learning experience that significantly enhanced my understanding of Kubernetes and containerization. It not only tested my technical skills but also pushed me to think critically about deployment strategies, scalability, and resilience. I'm excited to apply these skills in future projects and continue my journey in the world of cloud-native technologies.

**Picture Suggestion:** A final picture of you smiling, perhaps with a Kubernetes logo or your completed project dashboard in the background, to signify the successful completion of the challenge.

Thank you for reading about my journey! I hope this inspires you to take on the challenge yourself and deepen your understanding of Kubernetes.