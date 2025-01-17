# The Kubernetes Resume Challenge

## Project Overview
This challenge is all about showcasing my practical Kubernetes skills by deploying a real-world scenario: an e-commerce website. It goes beyond basic deployments, pushing me to demonstrate expertise in containerization, dynamic scaling, high availability, and efficient application management using Kubernetes. 

I'll be containerizing the application and its database using Docker, ensuring consistency across environments. 

To manage the deployment, I'll use a managed Kubernetes service (Amazon EKS), leveraging its features for dynamic scaling, high availability, and self-healing.

I'll also implement configuration management with Kubernetes ConfigMaps and Secrets, and package the whole application using Helm for easier management and versioning.

Finally, to streamline the development process, I'll set up a CI/CD pipeline using GitHub Actions, automating the build and deployment workflow. 

This project will be a practical showcase of my ability to design, deploy, and manage scalable applications in a Kubernetes environment.

## Interaction Flow

User Request: A user accesses the e-commerce website by entering the URL into their browser.

DNS Resolution: The user's browser queries DNS servers to resolve the website's domain name to an IP address.

Load Balancer: The resolved IP address points to a cloud-based Load Balancer (e.g., AWS Application Load Balancer, Azure Load Balancer, or GCP HTTP Load Balancing) managed by Kubernetes as a Service of type LoadBalancer.

Ingress Controller (Optional): If an Ingress controller is used, it would be placed in front of the LoadBalancer service to allow for advanced traffic routing based on hostname and path. Since we are only deploying a single website, an Ingress controller is not necessary.

Kubernetes Service: The Load Balancer distributes incoming traffic across a set of healthy pods running the e-commerce web application. These pods are managed by a Kubernetes Deployment. The Load Balancer is exposed to the public internet by a Kubernetes service of type LoadBalancer.

Web Application Pods: Each pod contains a Docker container running the e-commerce website code (PHP application server). The code is pulled from a Docker image hosted on Docker Hub.

Readiness and Liveness Probes: Kubernetes continuously monitors the health of these pods using readiness and liveness probes. If a pod fails a probe, it is removed from the Load Balancer's pool and/or restarted automatically.

Horizontal Pod Autoscaler (HPA): An HPA monitors the CPU utilization of the web application pods. Based on predefined thresholds, it automatically scales the number of pods up or down to handle fluctuations in traffic.

Database Pod: The e-commerce application connects to a separate pod running a MariaDB database. This pod uses the official MariaDB Docker image.

Persistent Storage: The MariaDB pod utilizes a PersistentVolumeClaim (PVC) to store its data on a persistent volume. This ensures that the database data is preserved even if the database pod is rescheduled or restarted.

ConfigMaps and Secrets: Application configurations (like feature toggles) are managed using ConfigMaps, while sensitive information (like database credentials) is stored securely in Secrets. These are injected into the relevant pods as environment variables or mounted files.

CI/CD Pipeline (GitHub Actions): When code changes are pushed to the GitHub repository, a GitHub Actions workflow is triggered. This workflow automatically builds a new Docker image, pushes it to Docker Hub, and updates the Kubernetes Deployment to use the new image, resulting in a seamless deployment.


**Picture Suggestion:** Include a personal headshot or a picture of you working on your computer here to personalize the README.

## Prerequisites: Getting Started

Before diving into the challenge, I made sure I had all the necessary tools and accounts set up:

*   **Docker and Kubernetes CLI Tools:** Essential for interacting with Docker and Kubernetes.
*   **Cloud Provider Account:** I used [AWS/Azure/GCP] for creating my Kubernetes cluster.
*   **GitHub Account:** For version control and later setting up a CI/CD pipeline.
*   **Kubernetes Crash Course:** A free course by KodeKloud to brush up on K8s basics.
*   **E-commerce Application Source Code and DB Scripts:** Provided by KodeKloud, this gave me a starting point for the application and database setup.

**Picture Suggestion:** A screenshot of your terminal showing `kubectl` and `docker` commands, or a diagram illustrating the interaction between these tools and your cloud provider.

## Step-by-Step Implementation

### Step 1: Certification

I started by completing the Certified Kubernetes Application Developer (CKAD) course by KodeKloud. This not only prepared me for the challenge but also gave me a solid foundation in Kubernetes.

**Picture Suggestion:** Your CKAD certification badge or a screenshot of your progress in the KodeKloud CKAD course.

### Step 2: Containerizing the E-Commerce Website and Database

**A. Web Application Containerization**

I created a `Dockerfile` for the e-commerce application, using `php:7.4-apache` as the base image and ensuring the `mysqli` extension was installed. The application code was copied to `/var/www/html/`, and database connection strings were updated to point to a Kubernetes service named `mysql-service`. Port 80 was exposed for web traffic.

After creating the `Dockerfile`, I built the image with `docker build -t [yourdockerhubusername]/ecom-web:v1 .` and pushed it to Docker Hub using `docker push [yourdockerhubusername]/ecom-web:v1`.

**Picture Suggestion:** A snippet of your `Dockerfile` or a screenshot of your Docker Hub repository showing the `ecom-web` image.

**B. Database Containerization**

For the database, I used the official MariaDB image and prepared the `db-load-script.sql` for initialization through Kubernetes ConfigMaps.

### Step 3: Setting Up Kubernetes on a Public Cloud Provider

I chose [AWS/Azure/GCP] and created a Kubernetes cluster using their managed Kubernetes service (EKS, AKS, or GKE). I configured `kubectl` to interact with my new cluster.

**Picture Suggestion:** A screenshot of your cloud provider's dashboard showing your Kubernetes cluster.

### Step 4: Deploying the Website to Kubernetes

I created a `website-deployment.yaml` file to define the deployment of my web application, referencing the Docker image I pushed earlier. This deployment managed the pods running my e-commerce application.

**Picture Suggestion:** A snippet of your `website-deployment.yaml` file or a screenshot of your terminal showing `kubectl get pods`.

### Step 5: Exposing the Website

To make my website accessible, I defined a `website-service.yaml` file to create a LoadBalancer service. This exposed my deployment to the internet.

**Picture Suggestion:** A diagram illustrating how the LoadBalancer service exposes your application to the internet.

### Step 6: Implementing Configuration Management

I added a feature toggle for a "dark mode" to the application and managed it using a ConfigMap named `feature-toggle-config`. This demonstrated how to manage application features dynamically.

**Picture Suggestion:** A screenshot of your application in dark mode and a snippet of your ConfigMap.

### Step 7: Scaling the Application

Anticipating increased traffic, I scaled the application using `kubectl scale deployment/ecom-web --replicas=6`. This showcased Kubernetes' ability to handle varying loads.

**Picture Suggestion:** A screenshot showing the increased number of pods after scaling.

### Step 8: Performing a Rolling Update

I updated the application to include a new promotional banner and built a new Docker image (`[yourdockerhubusername]/ecom-web:v2`). I then performed a rolling update by modifying the `website-deployment.yaml` file.

**Picture Suggestion:** A screenshot of the website with the new promotional banner and your terminal showing the `kubectl rollout status` command.

### Step 9: Rolling Back a Deployment

To simulate a rollback, I used `kubectl rollout undo deployment/ecom-web` after identifying an issue with the new banner. This demonstrated how to revert to a previous deployment state quickly.

**Picture Suggestion:** A screenshot of your terminal showing the `kubectl rollout undo` command.

### Step 10: Autoscaling the Application

I implemented a Horizontal Pod Autoscaler (HPA) to automatically scale the application based on CPU usage, ensuring performance under unpredictable traffic.

**Picture Suggestion:** A graph showing the CPU usage and the corresponding increase/decrease in the number of pods.

### Step 11: Implementing Liveness and Readiness Probes

I added liveness and readiness probes to the `website-deployment.yaml` file to ensure the application's health and readiness to receive traffic.

**Picture Suggestion:** A snippet of your `website-deployment.yaml` file showing the liveness and readiness probe definitions.

### Step 12: Utilizing ConfigMaps and Secrets

I used ConfigMaps for non-sensitive data like feature toggles and Secrets for sensitive data like database credentials, demonstrating best practices for configuration and secret management.

**Picture Suggestion:** Snippets of your ConfigMap and Secret definitions.

### Step 13: Documenting the Process

I documented each step of the process, decisions made, and challenges overcome. This README serves as a comprehensive guide to my project.

**Picture Suggestion:** A picture of a notebook or a digital document where you planned and documented your process.

## Extra Credit

### Packaging Everything in Helm

I packaged my application using Helm, creating a chart that defined all necessary Kubernetes resources. This simplified deployment and management. I also used the KodeKloud Helm Course to help me understand the process.

**Picture Suggestion:** A screenshot of your Helm chart directory structure or your terminal showing `helm` commands.

### Implementing Persistent Storage

I implemented persistent storage for the MariaDB database using a PersistentVolumeClaim (PVC), ensuring data durability across pod restarts.

**Picture Suggestion:** A snippet of your PVC definition or a diagram showing how the database data is stored persistently.

### Implementing a Basic CI/CD Pipeline

I set up a basic CI/CD pipeline using GitHub Actions to automate the build and deployment process, showcasing an efficient development workflow.

**Picture Suggestion:** A screenshot of your GitHub Actions workflow or a diagram illustrating your CI/CD pipeline.

## Conclusion

This Kubernetes Resume Challenge was a comprehensive learning experience that significantly enhanced my understanding of Kubernetes and containerization. It not only tested my technical skills but also pushed me to think critically about deployment strategies, scalability, and resilience. I'm excited to apply these skills in future projects and continue my journey in the world of cloud-native technologies.

**Picture Suggestion:** A final picture of you smiling, perhaps with a Kubernetes logo or your completed project dashboard in the background, to signify the successful completion of the challenge.

Thank you for reading about my journey! I hope this inspires you to take on the challenge yourself and deepen your understanding of Kubernetes.