# Deploy Netflix Clone on AWS Cloud using Jenkins

In this project, we will deploy a Netflix clone application as a Docker container on a AWS Kubernetes cluster through a secure CI/CD pipeline using Jenkins, a popular CI/CD tool. Along with Jenkins, we will incorporate a comprehensive security pipeline using tools such as SonarQube, Trivy, and OWASP Dependency-Check. Additionally, we will integrate monitoring with Prometheus and Grafana and implement GitOps with ArgoCD for Kubernetes deployment.

![image](https://github.com/user-attachments/assets/9194817f-dded-47c2-bf7e-29ed6fc83499)

---

# Phase 1: Initial Setup and Deployment
Within this phase, our objective is to deploy the Netflix application on Docker, and in subsequent phases, we will proceed to establish a CI/CD pipeline for it.

### Step 1: Launch EC2 (Ubuntu 22.04)
  Select Ubuntu server. Instance type as t2.large. You can create a new key pair or use an existing one. Enable HTTP and HTTPS settings in the Security Group. EBS Volumes as 30GB.

### Step 2: Install Docker and run the App using container

```bash
# clone the repo
sudo apt-get update
sudo apt-get upgrade -y
git clone https://github.com/kaviara-14/DevSecOps-Netflix-Clone-with-Jenkins-CI-CD.git

# Install Docker
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER
newgrp docker
sudo chmod 777 /var/run/docker.sock

# Build and run the application
docker build -t netflix .
docker run -d --name netflix -p 8081:80 netflix:latest

```

### Step 3: Get the API Key from TMDB

```bash
# Obtain a TMDB API key and rebuild the Docker image
docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix .
```
---

# Phase 2: Security
SonarQube is primarily used for static code analysis and code quality improvement across various programming languages, while Trivy specializes in scanning container images for security vulnerabilities, making it a crucial tool in containerized environments.

### Install SonarQube and Trivy
Install SonarQube and Trivy on the EC2 instance to scan for vulnerabilities.

```bash
# Install SonarQube
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

# To access SonarQube, publicIP:9000 (by default username & password is admin)

# Install Trivy
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy

# Scan Docker images with Trivy:
trivy image <imageid>
```
---

# Phase 3: CI/CD Setup
Our Jenkins pipeline is a powerhouse, integrating SonarQube for code quality, Trivy for container security, and OWASP for web app safety. It pulls code from GitHub, ensuring automated, high-quality, and secure software development.

### Step 1 : Install Jenkins
```bash
# Install Java on EC2 Instance
sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version

# Install Jenkins on EC2 Instance
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Retrieve the initial admin password
cat /var/lib/jenkins/secrets/initialAdminPassword

# Access Jenkins at http://<publicIP>:8080

```

### Step 2 : Install and configure Plugins like JDK, Sonarqube Scanner, NodeJs, OWASP Dependency Check in Jenkins 
Goto Manage Jenkins →Plugins → Available Plugins, Install Necessary Jenkins Plugins like **Eclipse Temurin Installer, SonarQube Scanne, NodeJs Plugin, OWASP Dependency-Check, Email Extension Plugin, Docker, Docker Commons, Docker Pipeline, Docker API, docker-build-step.**
 * **Configure Java and Nodejs in Global Tool Configuration :** Goto Manage Jenkins → Tools → Install JDK(17) and NodeJs(16)→ Click on Apply and Save.
 * **Configure Sonar Server in Manage Jenkins :** Goto your Sonarqube Server, < Your Public IP >:9000, generate a token. In Jenkins, Goto Jenkins Dashboard → Manage Jenkins → Credentials → Add Secret Text. (Put copied token).
 * Add DockerHub Username and Password under Global Credentials.

## Step 3 : Configure CI/CD Pipeline in Jenkins

In this github repo we have a Jenkinsfile, In that file you will see the pipeline code.Build the pipeline in Jenkins to see the result.

---

# Phase 4 : Prometheus Setup and Node Exporter 
### Step 1 : Install Prometheus
Set up Prometheus and Grafana to monitor your application. Prometheus is a monitoring and alerting tool that collects metrics and helps track the health of systems and applications.Launch a new instance (Ubuntu). Instance type as t2.medium. You can create a new key pair or use an existing one. Enable HTTP and HTTPS settings in the Security Group. EBS Volumes as 20GB.

```bash
# Installing Prometheus
sudo useradd --system --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz

# Extract Prometheus files, move them, and create directories
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
sudo mv prometheus /usr/local/bin/
sudo mkdir -p /etc/prometheus /data
sudo mv prometheus.yml /etc/prometheus/
sudo chown -R prometheus:prometheus /etc/prometheus/ /data

# Create a systemd unit configuration file for Prometheus
sudo nano /etc/systemd/system/prometheus.service

# Enable and start Prometheus
sudo systemctl enable prometheus
sudo systemctl start prometheus

# You can access Prometheus in a web browser using your server's IP and port 9090

```

### Step 2 : Install Node Exporter
Next, we're going to set up and configure Node Exporter to collect Linux system metrics like CPU load and disk I/O. Node Exporter will expose these as Prometheus-style metrics.
```bash
# Create a system user for Node Exporter and download Node Exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter /usr/local/bin/

# Create a systemd unit configuration file for Node Exporter
sudo nano /etc/systemd/system/node_exporter.service

# Enable and start Node Exporter
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

### Step 3 : Configure Prometheus Plugin Integration:
Integrate Jenkins with Prometheus to monitor the CI/CD pipeline.To configure Prometheus to scrape metrics from Node Exporter and Jenkins, you need to modify the prometheus.yml file. Here is an example prometheus.yml configuration for your setup:
```bash
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['<your-jenkins-ip>:<your-jenkins-port>']

# Check the validity of the configuration file
promtool check config /etc/prometheus/prometheus.yml

# You can access Prometheus targets at, http://<your-prometheus-ip>:9090/targets
```
---

# Phase 5 - Grafana Dashboard
Grafana complements Prometheus by providing a visualization platform to create interactive dashboards and analyze collected metrics, making it easier to monitor and troubleshoot systems.
```bash
# Add the GPG key for Grafana 
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

# Add Grafana Repository
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list

Update and Install Grafana
sudo apt-get update
sudo apt-get install grafana -y

# Enable and Start Grafana Service
sudo systemctl start grafana-server
sudo systemctl enable grafana-server

# Access Grafana at http://<server-ip>:3000 add Prometheus Data Source and import the dashboard to see the output.

```

--- 
# Phase 5: Notification

  * Configure email notification systems in Jenkins.
    
--- 
# Phase 6: Kubernetes
### Step 1 : Install AWS CLI, Kubectl, eksctl and helm chart

```bash
# Install AWS CLI v2 and configure
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure

# Install kubectl
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client

# Install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

# Install Helm Chart - Use the following script to install the helm chart 
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

### Step 2 : Creating an AWS EKS cluster using eksctl
Now in this step, we are going to create Amazon EKS cluster using eksctl

```bash
# You need the following to run the eksctl command
eksctl create cluster --name eks-netflix-1 --version 1.24 --region us-east-1 --nodegroup-name worker-nodes --node-type t2.medium --nodes 1 --nodes-min 1 --nodes-max 1
aws eks update-kubeconfig --region us-east-1 --name eks-netflix-1
kubectl get nodes

# Create IAM OIDC provider
eksctl utils associate-iam-oidc-provider \
    --region ${AWS_REGION} \
    --cluster ${EKS_CLUSTER_NAME} \
    --approve

# Download IAM policy for the AWS Load Balancer Controller
curl -fsSL -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.0/docs/install/iam_policy.json

# Create a IAM role and ServiceAccount for the AWS Load Balancer controller using eksctl tool
eksctl create iamserviceaccount \
    --cluster=${EKS_CLUSTER_NAME} \
    --namespace=kube-system \
    --name=aws-load-balancer-controller \
    --attach-policy-arn=arn:aws:iam::${AWS_ACCOUNT_ID}:policy/AWSLoadBalancerControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --approve \
    --region ${AWS_REGION}

# Install the helm chart by specifying the chart values serviceAccount.create=false and serviceAccount.name=aws-load-balancer-controller
helm repo add eks https://aws.github.io/eks-charts
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
    -n kube-system \
    --set clusterName=${EKS_CLUSTER_NAME} \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller

# Install ArgoCD
kubectl create namespace argocd 
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.7/manifests/install.yaml

```
### Step 3 : Configure Argocd
* Take the LoadBalancer link and open it in your browser.After installing ArgoCD, you need to set up your GitHub repository as a source for your application deployment. This typically involves configuring the connection to your repository and defining the source for your ArgoCD application.
* Once you configured you have now successfully deployed an application using Argo CD.Argo CD is a Kubernetes controller, responsible for continuously monitoring all running applications and comparing their live state to the desired state specified in the Git repository.

### Step 4 : Install Node Exporter using Helm
To begin monitoring your Kubernetes cluster, you'll install the Prometheus Node Exporter. This component allows you to collect system-level metrics from your cluster nodes. Here are the steps to install the Node Exporter using Helm:

```bash
# Add the Prometheus Community Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Create a Kubernetes namespace for the Node Exporter
kubectl create namespace prometheus-node-exporter

# Install the Node Exporter using Helm
helm install prometheus-node-exporter prometheus-community/prometheus-node-export
```

Open inbound traffic on port 30007 and 9100 for EKS cluster node IP.To Access the app make sure port 30007 is open in your security group and then open a new tab paste your NodeIP:30007, your app should be running.
