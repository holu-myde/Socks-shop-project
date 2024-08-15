# **The Socks Shop Microservices-based Application Deployment on Kubernetes using IaC.**

## **Project Overview:**

The Socks Shop application is a popular microservices-based e-commerce platform that is used as a reference application for demonstrating modern cloud-native technologies. The application is composed of multiple microservices, each of which is responsible for a specific function, such as product catalog, shopping cart, and user authentication. The application is designed to be highly scalable, resilient, and fault-tolerant, making it an ideal candidate for deployment on Kubernetes.

This project will involve deploying the Socks Shop application on a Kubernetes cluster using an Infrastructure as Code (IaC) approach. This will include provisioning the necessary infrastructure resources on Azure, setting up a deployment pipeline, monitoring the performance and health of the application, and securing the infrastructure.

The project will be implemented using Terraform for infrastructure provisioning, GitHub Actions for the deployment pipeline, Kubernetes for container orchestration, Helm for package management, Prometheus for monitoring, ELK Stack for logging, and Ansible for security.

## **Project Objectives:**

- Deploy the Socks Shop application on a Kubernetes cluster using an Infrastructure as Code (IaC) approach
- Provision the necessary infrastructure resources using a preferred cloud provider (I will be using Azure in this case)
- Set up a deployment pipeline using GitHub Actions to build and deploy the Socks Shop application to the Kubernetes cluster
- Monitor the performance and health of the Socks Shop application using Prometheus
- Collect and analyze logs from the Socks Shop application using a centralized logging solution, such as the ELK stack.

## **This project documentation has the following sections:**

- [Infrastructure Provisioning](#infrastructure-provisioning)
- [Deployment Pipeline](#deployment-pipeline)
- [Monitoring](#monitoring)
- [Logging](#logging)
- [Security](#security)
- [conclusion](#conclusion)
- [References](#references)

## **Required Tools and Technologies and what they are used for:**

The following tools and technologies will be used in this project:

- Terraform: an open-source infrastructure as code (IaC) tool that provides a consistent CLI workflow to manage hundreds of cloud services. It codifies APIs into declarative configuration files, creating infrastructure as code using a high-level configuration language called HCL (HashiCorp Configuration Language).

- An Azure Account: required to provision the necessary infrastructure resources, such as the AKS cluster.

- GitHub Actions: will be used to set up a deployment pipeline to build and deploy the Socks Shop application to the Kubernetes cluster.

- Kubernetes: an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications.

- Helm: a package manager for Kubernetes that provides an easy way to find, share, and use software built for Kubernetes.

- Prometheus: an open-source monitoring and alerting toolkit designed for reliability and scalability. It collects metrics from configured targets at given intervals, evaluates rule expressions, displays the results, and can trigger alerts if some condition is observed to be true.

- ELK Stack: The ELK Stack is a collection of three open-source products — Elasticsearch, Logstash, and Kibana — all developed, managed, and maintained by Elastic. The ELK Stack is used to collect, search, analyze, and visualize log data in real time.

- Let's Encrypt: a free, automated and open certificate authority made available by the nonprofit Internet Security Research Group (ISRG).

- Docker: a set of platform as a service (PaaS) products that use OS-level virtualization to deliver software in packages called containers.

- The Socks Shop Application Repository: The Socks Shop application is a popular microservices-based e-commerce platform that is used as a reference application for demonstrating modern cloud-native technologies.

## **Infrastructure Provisioning:**

Using Terraform, we will provision the necessary infrastructure resources on Azure, to ensure a clear and reproducible infrastructure setup.

1.  Download and install Terraform and Azure CLI on your local machine. If not, you can download it from the official website.

    [Azure CLI Installation guide](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)

    [Terraform Download](https://www.terraform.io/downloads.html)

2.  Create a new directory for the Terraform configuration files and navigate to it.

        mkdir Socks-Shop_Terraform
        cd Socks-Shop_Terraform

3.  Git clone this repository and navigate to the terraform folder to have the Terraform configuration files and initiate the Terraform project.

        git clone https://github.com/holu-myde/Socks-shop-project/tree/main/Sock-shop/Terraform

4.  Run the following command to initialize the Terraform project:

        terraform init

5.  Run the following command to create an execution plan:

        terraform plan

6.  Run the following command to apply the changes:

        terraform apply --auto-approve

        #the flag --auto-approve can be added to avoid the prompt for confirmation.

Below is a screenshot of my AKS cluster being provisioned by terraform👇🏽:

  <img src="Images/K8s deployment.JPG">

The command below allows us to configure the kubectl to connect to the AKS cluster.

    az aks update-kubeconfig --name=sock-shop

7.  After the infrastructure has been provisioned, you will see the output of the Terraform apply command, including the AKS cluster endpoint and the kubeconfig file.

- We apply our deployment manifests to our cluster using the following command:

        kubectl create -f Kubernetes/deployment.yaml

    <img src="Images/App deployment to K8s cluster.JPG">

8.  You can use the following commands to verify that the Socks Shop application is running on the Kubernetes cluster:

        kubectl get pods

    <img src="Images/Running pods.JPG">

        kubectl get svc
    <img src="Images/Services running.JPG">

10. After you confirm that your pods are running, you can test the application by copying the external IP of the frontend service (type- Load Balancer) into your browser.

    <img src="Images/Frontend via the loadbalancer external IP.JPG">

11. To serve the frontend via a custom domain, you need to obtain a custom domain from a provider then create A records to point your custom domain name to the external IP of the front end service (see example of namecheap DNS configuration and how application was served via custom domain below).

    <img src="Images/A-records created and pointing to the LoadBalancer external IP.JPG">

    <img src="Images/App hosted with my domain name - unsecured.JPG">

## **Deployment Pipeline:**

The deployment pipeline will be configured using a GitHub Actions workflow file, which will define the steps required to build and deploy the Socks Shop application. The workflow file will be triggered by a push to the main branch of the repository, and will include the following steps:

Our workflow file must be in our root directory for our GitHub Actions to detect the file automatically.

- Checkout the source code from the repository
- Build the Docker images for the Socks Shop application
- Deploy the Socks Shop application to the Kubernetes cluster

The deployment pipeline will be configured to run automatically whenever changes are pushed to the main branch of the repository, ensuring that the Socks Shop application is always up to date and running the latest version.

## **Monitoring**

Prometheus will be used to monitor the performance and health of the Socks Shop application. This will include metrics such as request latency, error rate, and request volume. The Prometheus server will be configured to scrape metrics from the Socks Shop application and store them in a time-series database. Grafana will be used to visualize the metrics and create dashboards to monitor the performance and health of the application.

- **Prometheus**

First install Prometheus using helm by running each of the commands below.

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

    helm repo update

    helm install prometheus prometheus-community/prometheus

To deploy simply apply all the prometheus manifests (01-10) in any order:

    kubectl apply $(ls *-prometheus-*.yaml | awk ' { print " -f " $1 } ')

The image below shows successful installation and deployment

<img src="Images/Prometheus installation and deployment.JPG">

The prometheus server will be exposed through a Load Balancer using the following command:

    kubectl expose service prometheus-server --type=LoadBalancer --target-port=9090 --name=prometheus-server-ext

<img src="Images/Prometheus 2.JPG">

The external IP of the Prometheus external server will then be used to configure Grafana

- **Grafana**

First install Grafana using helm by running each of the commands below

    helm repo add grafana https://grafana.github.io/helm-charts

    helm repo update

    helm install grafana grafana/grafana

Then apply the grafana manifests:

    kubectl apply $(ls *-grafana-*.yaml | awk ' { print " -f " $1 }'  | grep -v grafana-import)

The image below shows successful installation and deployment

<img src="Images/Grafana installed and deployed.JPG">

Grafana will be exposed through a Load Balancer using the following command:

    kubectl expose service grafana — type=LoadBalancer — target-port=3000 — name=grafana-ext

<img src="Images/Grafana details.JPG">

The external IP of the Grafana external service will be used to access the dashboard. See screenshots below. 

<img src="Images/Grafana dahboard 1.JPG">
<img src="Images/Grafana 2.JPG">
<img src="Images/Grafana 3.JPG">
<img src="Images/Grafana 4.JPG">
<img src="Images/Grafana 5.JPG">
<img src="Images/Grafana 6.JPG">


