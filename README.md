# Netflix Clone Deployment on AWS using Jenkins - DevSecOps Project

This DevSecOps project demonstrates deploying a Netflix clone application as a Docker container on a Kubernetes cluster using a secure CI/CD pipeline managed by Jenkins, a leading CI/CD tool. The process incorporates several in-demand DevOps tools, including Docker for containerization, SonarQube for code quality analysis, Trivy for vulnerability scanning, Prometheus and Grafana for monitoring, and ArgoCD with Helm for Kubernetes deployment. The project highlights end-to-end DevSecOps practices, covering pipeline setup, security integration, deployment, monitoring, and notifications, offering a comprehensive hands-on experience with modern DevOps workflows.

![Screenshot 2024-12-14 125517](https://github.com/user-attachments/assets/2814730c-b219-4438-8e11-c67c33be3a78)

# Phase 1: Initial Setup and Deployment
This phase focuses on setting up the foundational environment needed for deploying the application. It includes provisioning a cloud instance, installing necessary tools, and initializing the application to ensure a seamless deployment experience.
### Step 1: Launch EC2 (Ubuntu 22.04):
  * Provision an EC2 instance on AWS with Ubuntu 22.04.
  * Connect to the instance using SSH.

### Step 2: Clone the Code:

  * Update all the packages and clone the code:

          sudo apt-get update
          sudo apt-get upgrade -y
          git clone https://github.com/N4si/DevSecOps-Project.git

### Step 3: Install Docker and Run the App Using a Container:

  * Install Docker:

          sudo apt-get install docker.io -y
          sudo usermod -aG docker $USER
          newgrp docker
          sudo chmod 777 /var/run/docker.sock

  * Build and run the application:

          docker build -t netflix .
          docker run -d --name netflix -p 8081:80 netflix:latest

### Step 4: Get the API Key:

  * Obtain a TMDB API key and rebuild the Docker image

          docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix .

# Phase 2: Security

### Install SonarQube and Trivy:

  * Install SonarQube:

          docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

  * Access SonarQube at http://<publicIP>:9000 (default username/password: admin/admin).

  *  Install Trivy:

          sudo apt-get install wget apt-transport-https gnupg lsb-release
          wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
          echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
          sudo apt-get update
          sudo apt-get install trivy

  * Scan Docker images with Trivy:

        trivy image <imageid>

### Integrate SonarQube:
  * Configure SonarQube to analyze code for quality and security issues
# Phase 3: CI/CD Setup

### Install Jenkins:

  * Install Java and Jenkins:

        sudo apt update
        sudo apt install fontconfig openjdk-17-jre
        java -version
        sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
        echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
        sudo apt-get update
        sudo apt-get install jenkins
        sudo systemctl start jenkins
        sudo systemctl enable jenkins

  * Access Jenkins at http://<publicIP>:8080.

  * Install Necessary Jenkins Plugins:
    * Plugins to install
      * Eclipse Temurin Installer
      * SonarQube Scanner
      * NodeJs Plugin
      * Email Extension Plugin

  * Configure Jenkins:
    * Add SonarQube credentials and DockerHub credentials.
    * Install and configure tools like Sonar Scanner and Dependency Check.

  * Add the pipeline code and build now to see the output.

      ![Screenshot 2024-12-14 130505](https://github.com/user-attachments/assets/a24387a7-ab2f-4ca7-932b-85f81a95f1af)
# Phase 4: Monitoring

### Install Prometheus and Grafana:

  * Install Prometheus:

        wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
        tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
        sudo mv prometheus /usr/local/bin/
        sudo mkdir -p /etc/prometheus /data
        sudo mv prometheus.yml /etc/prometheus/
        sudo chown -R prometheus:prometheus /etc/prometheus/ /data

  * Configure and start Prometheus.
  * Install Node Exporter:

        wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
        tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
        sudo mv node_exporter /usr/local/bin/

  * Install Grafana:

        wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
        echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
        sudo apt-get update
        sudo apt-get install grafana -y
        sudo systemctl start grafana-server
        sudo systemctl enable grafana-server

  * Access Grafana at http://<server-ip>:3000.

      ![Screenshot 2024-12-14 130652](https://github.com/user-attachments/assets/c45fb1a5-a1f1-4747-9b38-bfe13c1f0f25)
    
# Phase 5: Notification

  * Configure email or other notification systems in Jenkins.
    
# Phase 6: Kubernetes

### Create Kubernetes Cluster with Nodegroups:

  * Set up a Kubernetes cluster using a cloud provider.
  * Install Prometheus Node Exporter using Helm:

        helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
        kubectl create namespace prometheus-node-exporter
        helm install prometheus-node-exporter prometheus-community/prometheus-node-exporter --namespace prometheus-node-exporter

### Deploy Application with ArgoCD:
  
  * Install and configure ArgoCD to manage application deployments.









